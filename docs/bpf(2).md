## NAME

bpf - 확장 BPF 맵과 프로그램에 대한 명령 수행하기

## SYNOPSIS

```c
#include <linux/bpf.h>

int bpf(int cmd, union bpf_attr *attr, unsigned int size);
```

## DESCRIPTION

`bpf()` 시스템 호출은 확장 버클리 패킷 필터(extended Berkeley Packet Filter)와 관련된 다양한 작업을 수행한다. 확장 BPF(eBPF)는 네트워크 패킷 필터링에 쓰는 원래의 ("전통적") BPF(cBPF)와 비슷하다. cBPF 프로그램과 eBPF 프로그램 모두 적재 전에 커널이 정적 분석을 하여 실행 시스템에 해를 끼치지 않도록 한다.

eBPF는 cBPF를 여러 방식으로 확장한 것이다. (eBPF에서 제공하는 `BPF_CALL` 명령 코드 확장을 통해) 정해진 커널 내 함수들을 호출할 수 있으며 eBPF 맵 같은 공유 자료 구조에 접근할 수 있다.

### 확장 BPF 설계/구조

eBPF 맵은 다양한 종류의 데이터 저장을 위한 범용 자료 구조이다. 데이터 타입들을 일반적으로 이진 바이트 열로 다루며, 그래서 사용자가 맵 생성 시점에 키의 크기와 값의 크기를 지정할 뿐이다. 다시 말해 맵이 있을 때 그 키/값은 어떤 구조든 가질 수 있다.

사용자 프로세스에서 (불투명 데이터인 키/값 쌍을 가진) 여러 맵을 만들고 파일 디스크립터를 통해 접근할 수 있다. 여러 eBPF 프로그램들이 같은 맵에 병렬로 접근할 수 있다. 맵 안에 무엇을 저장할지는 사용자 프로세스와 eBPF 프로그램의 결정에 달려 있다.

특수한 종류의 맵이 하나 있는데, 프로그램 배열이라는 것이다. 이 맵은 다른 eBPF 프로그램들을 가리키는 파일 디스크립터들을 저장한다. 그 맵에서 탐색 수행 시 프로그램 흐름이 다른 eBPF 프로그램의 시작점으로 그대로 옮겨지며 호출 프로그램으로 되돌아 오지 않는다. 들어가는 깊이에 32번이라는 고정 제한이 있어서 무한 루프가 생길 수 없게 한다. 런타임에 맵에 저장된 프로그램 파일 디스크립터들을 변경할 수 있으므로 구체적 요구에 따라 프로그램 기능을 바꿀 수 있다. 프로그램 배열 맵에서 참조하는 모든 프로그램은 `bpf()`를 통해 커널로 미리 적재해 두어야 한다. 맵 탐색이 실패하면 현재 프로그램이 실행을 계속한다. 더 자세한 건 아래의 `BPF_MAP_TYPE_PROG_ARRAY` 설명을 보라.

일반적으로 eBPF 프로그램은 사용자 프로그램에 의해 적재되며 그 프로세스가 끝날 때 자동으로 내려간다. <tt>[[tc-bpf(8)]]</tt> 같은 일부 경우에서는 프로그램을 적재한 프로세스가 끝난 후에도 프로그램이 커널 내에 계속 살아 있다. 그 경우에는 사용자 공간 프로그램이 파일 디스크립터를 닫은 후에 tc 서브시스템이 eBPF 프로그램에 대한 참조를 잡고 있는다. 즉, 특정 프로그램이 커널 내에 계속 살아 있는지 여부는 `bpf()`를 통해 적재된 후에 해당 커널 서브시스템에 어떻게 붙는가에 따라 정해진다.

각 eBPF 프로그램은 완료 때까지 안전하게 실행할 수 있는 인스트럭션들의 집합이다. eBPF 프로그램에 끝이 있고 실행하기에 안전한지를 커널 내 검증기가 정적으로 판단한다. 검증하는 동안 커널은 그 eBPF 프로그램에서 사용하는 맵들 각각에 대한 참조 카운터를 올려서 프로그램을 내릴 때까지 관련 맵들이 제거되지 않도록 한다.

eBPF 프로그램을 다양한 이벤트에 연계할 수 있다. 그 이벤트는 네트워크 패킷 도착일 수도 있고, 추적 이벤트, 네트워크 큐 규제의 분류 이벤트 (`tc(8)` 분류자에 연계된 eBPF 프로그램의 경우), 그리고 향후 추가될 다른 종류일 수 있다. 새 이벤트가 eBPF 프로그램 실행을 일으키고, 그러면 프로그램에서 eBPF 맵에 그 이벤트에 대한 정보를 저장할 수도 있다. 데이터 저장 외에 eBPF 프로그램이 한정된 커널 내 헬퍼 함수들을 호출할 수도 있다.

동일한 eBPF 프로그램을 여러 이벤트에 연계할 수 있고 상이한 eBPF 프로그램이 동일 맵에 접근할 수 있다.

```text
추적       추적       추적        eth0의      eth1의      eth2의
이벤트 A   이벤트 B   이벤트 C    패킷        패킷        패킷
  |          |          |           |           |           ^
  |          |          |           |           v           |
  --> 추적 <--        추적        소켓       tc 입구     tc 출구
      prog_1          prog_2      prog_3      분류자       행위
      |  |              |           |         prog_4      prog_5
   |---  -----|  |------|          map_3        |           |
 map_1       map_2                              --| map_4 |--
```

### 인자

`cmd` 인자가 `bpf()` 시스템 호출이 수행할 작업을 결정한다. 각 작업은 `bpf_attr` 타입 공용체에 대한 포인터인 `attr`을 통해 추가 인자를 받는다 (아래 참고). `size` 인자는 `attr`이 가리키는 공용체의 크기이다.

`cmd`로 주는 값은 다음 중 하나이다.

`BPF_MAP_CREATE`
:   맵을 생성하고 그 맵을 가리키는 파일 디스크립터를 반환한다. 새 파일 디스크립터에는 exec에서 닫기 플래그(<tt>[[fcntl(2)]]</tt> 참고)가 자동으로 켜진다.

`BPF_MAP_LOOKUP_ELEM`
:   지정한 맵에서 키로 항목을 찾아서 그 값을 반환한다.

`BPF_MAP_UPDATE_ELEM`
:   지정한 맵에서 항목(키/값 쌍)을 생성하거나 갱신한다.

`BPF_MAP_DELETE_ELEM`
:   지정한 맵에서 키로 항목을 찾아서 삭제한다.

`BPF_MAP_GET_NEXT_KEY`
:   지정한 맵에서 키로 항목을 찾아서 다음 항목의 키를 반환한다.

`BPF_PROG_LOAD`
:   eBPF 프로그램을 검증 및 적재하고 프로그램과 연계된 새 파일 디스크립터를 반환한다. 새 파일 디스크립터에는 exec에서 닫기 플래그(<tt>[[fcntl(2)]]</tt> 참고)가 자동으로 켜진다.

`bpf_attr` 공용체는 여러 `bpf()` 명령에서 쓰는 다양한 익명 구조체들로 이뤄져 있다.

```c
union bpf_attr {
    struct {    /* BPF_MAP_CREATE에 사용 */
        __u32         map_type;
        __u32         key_size;    /* 키 크기, 바이트 단위 */
        __u32         value_size;  /* 값 크기, 바이트 단위 */
        __u32         max_entries; /* 맵 내의 항목 최대 개수 */
    };

    struct {    /* BPF_MAP_*_ELEM 및 BPF_MAP_GET_NEXT_KEY
                   명령에 사용 */
        __u32         map_fd;
        __aligned_u64 key;
        union {
            __aligned_u64 value;
            __aligned_u64 next_key;
        };
        __u64         flags;
    };

    struct {    /* BPF_PROG_LOAD에 사용 */
        __u32         prog_type;
        __u32         insn_cnt;
        __aligned_u64 insns;      /* 'const struct bpf_insn *' */
        __aligned_u64 license;    /* 'const char *' */
        __u32         log_level;  /* 검증기의 상세도 수준 */
        __u32         log_size;   /* 사용자 버퍼 크기 */
        __aligned_u64 log_buf;    /* 사용자가 제공하는 'char *'
                                     버퍼 */
        __u32         kern_version;
                                  /* prog_type=kprobe일 때 검사
                                     (리눅스 4.1부터) */
    };
} __attribute__((aligned(8)));
```

### eBPF 맵

맵은 다양한 데이터 타입을 저장할 수 있는 범용 자료 구조이다. 이를 통해 eBPF 커널 프로그램들 사이에서, 그리고 커널과 사용자 공간 응용 사이에서 데이터 공유가 가능하다.

각 맵에는 다음 속성이 있다.

 * 종류

 * 항목 최대 개수

 * 키의 바이트 단위 크기

 * 값의 바이트 단위 크기

다양한 `bpf()` 명령으로 맵에 접근하는 방법을 아래 래퍼 함수들이 보여 준다. 이 함수들은 `cmd` 인자를 이용해 각기 다른 작업을 호출한다.

#### `BPF_MAP_CREATE`

`BPF_MAP_CREATE` 명령은 새로운 맵을 만들고 그 맵을 가리키는 새 파일 디스크립터를 반환한다.

```c
int
bpf_create_map(enum bpf_map_type map_type,
               unsigned int key_size,
               unsigned int value_size,
               unsigned int max_entries)
{
    union bpf_attr attr = {
        .map_type    = map_type,
        .key_size    = key_size,
        .value_size  = value_size,
        .max_entries = max_entries
    };

    return bpf(BPF_MAP_CREATE, &attr, sizeof(attr));
}
```

새 맵은 `map_type`으로 지정한 종류이고 `key_size`, `value_size`, `max_entries`로 지정한 속성을 가진다. 성공 시 이 동작은 파일 디스크립터를 반환한다. 오류 시 -1을 반환하며 `errno`를 `EINVAL`이나 `EPERM`, `ENOMEM`으로 설정한다.

프로그램 적재 과정에서 프로그램이 올바로 초기화 한 `key`로 `bpf_map_*_elem()` 헬퍼 함수를 호출하며 맵 항목 `value`를 지정한 `value_size` 너머로 접근하지 않음을 검증기에서 확인하는 데 `key_size`와 `value_size` 속성을 사용한다. 예를 들어 `key_size`를 8로 해서 맵을 생성했는데 eBPF 프로그램에서 다음 호출을 하면 프로그램이 거부될 것이다.

```c
bpf_map_lookup_elem(map_fd, fp - 4)
```

커널 내 헬퍼 함수인

```c
bpf_map_lookup_elem(map_fd, void *key)
```

에서는 `key`가 가리키는 위치에서 8바이트를 읽기를 기대하는데 `fp - 4`(`fp`는 스택 상단)라는 시작 주소는 범위를 벗어난 스택 접근을 유발할 것이기 때문이다.

마찬가지로 `value_size`를 1로 해서 맵을 생성했는데 eBPF 프로그램에 다음 내용이 있으면 프로그램이 거부될 것이다.

```c
value = bpf_map_lookup_elem(...);
*(u32 *) value = 1;
```

지정한 1바이트 `value_size` 제한 너머로 `value` 포인터에 접근하기 때문이다.

현재 `map_type`으로 다음 값들을 지원한다.

```c
enum bpf_map_type {
    BPF_MAP_TYPE_UNSPEC,  /* 0은 유효하지 않은 맵 종류로 예약 */
    BPF_MAP_TYPE_HASH,
    BPF_MAP_TYPE_ARRAY,
    BPF_MAP_TYPE_PROG_ARRAY,
    BPF_MAP_TYPE_PERF_EVENT_ARRAY,
    BPF_MAP_TYPE_PERCPU_HASH,
    BPF_MAP_TYPE_PERCPU_ARRAY,
    BPF_MAP_TYPE_STACK_TRACE,
    BPF_MAP_TYPE_CGROUP_ARRAY,
    BPF_MAP_TYPE_LRU_HASH,
    BPF_MAP_TYPE_LRU_PERCPU_HASH,
    BPF_MAP_TYPE_LPM_TRIE,
    BPF_MAP_TYPE_ARRAY_OF_MAPS,
    BPF_MAP_TYPE_HASH_OF_MAPS,
    BPF_MAP_TYPE_DEVMAP,
    BPF_MAP_TYPE_SOCKMAP,
    BPF_MAP_TYPE_CPUMAP,
    BPF_MAP_TYPE_XSKMAP,
    BPF_MAP_TYPE_SOCKHASH,
    BPF_MAP_TYPE_CGROUP_STORAGE,
    BPF_MAP_TYPE_REUSEPORT_SOCKARRAY,
    BPF_MAP_TYPE_PERCPU_CGROUP_STORAGE,
    BPF_MAP_TYPE_QUEUE,
    BPF_MAP_TYPE_STACK,
    /* 전체 목록은 /usr/include/linux/bpf.h를 보라. */
};
```

`map_type`이 커널에서 사용 가능한 맵 구현들 중 하나를 선택한다. eBPF 프로그램은 모든 맵 종류에 동일한 `bpf_map_lookup_elem()` 및 `bpf_map_update_elem()` 헬퍼 함수를 사용해 접근한다. 아래에 여러 맵 종류에 대한 자세한 내용이 있다.

#### `BPF_MAP_LOOKUP_ELEM`

`BPF_MAP_LOOKUP_ELEM` 명령은 파일 디스크립터 `fd`가 가리키는 맵에서 주어진 `key`로 항목을 찾는다.

```c
int
bpf_lookup_elem(int fd, const void *key, void *value)
{
    union bpf_attr attr = {
        .map_fd = fd,
        .key    = ptr_to_u64(key),
        .value  = ptr_to_u64(value),
    };

    return bpf(BPF_MAP_LOOKUP_ELEM, &attr, sizeof(attr));
}
```

항목을 찾으면 동작이 0을 반환하며 항목의 값을 `value`에 저장한다. `value`는 `value_size` 바이트 크기의 버퍼를 가리켜야 한다.

항목을 찾지 못하면 동작이 -1을 반환하며 `errno`를 `ENOENT`로 설정한다.

#### `BPF_MAP_UPDATE_ELEM`

`BPF_MAP_UPDATE_ELEM` 명령은 파일 디스크립터 `fd`가 가리키는 맵에서 주어진 `key`/`value`로 항목을 생성 또는 갱신한다.

```c
int
bpf_update_elem(int fd, const void *key, const void *value,
                uint64_t flags)
{
    union bpf_attr attr = {
        .map_fd = fd,
        .key    = ptr_to_u64(key),
        .value  = ptr_to_u64(value),
        .flags  = flags,
    };

    return bpf(BPF_MAP_UPDATE_ELEM, &attr, sizeof(attr));
}
```

`flags` 인자는 다음 중 하나로 지정해야 한다.

`BPF_ANY`
:   새 항목을 생성하거나 기존 항목을 갱신한다.

`BPF_NOEXIST`
:   존재하지 않을 때 새 항목을 생성하기만 한다.

`BPF_EXIST`
:   기존 항목을 갱신한다.

성공 시 동작이 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 `EINVAL`이나 `EPERM`, `ENOMEM`, `E2BIG`으로 설정한다. `E2BIG`은 맵 내의 항목 수가 맵 생성 시 지정한 `max_entries` 제한에 도달했다는 뜻이다. `flags`가 `BPF_NOEXIST`인데 `key`를 가진 항목이 이미 맵에 존재하면 `EEXIST`를 반환한다. `flags`가 `BPF_EXIST`인데 `key`를 가진 항목이 맵에 존재하지 않으면 `ENOENT`를 반환한다.

#### `BPF_MAP_DELETE_ELEM`

`BPF_MAP_DELETE_ELEM` 명령은 파일 디스크립터 `fd`가 가리키는 맵에서 키가 `key`인 항목을 삭제한다.

```c
int
bpf_delete_elem(int fd, const void *key)
{
    union bpf_attr attr = {
        .map_fd = fd,
        .key    = ptr_to_u64(key),
    };

    return bpf(BPF_MAP_DELETE_ELEM, &attr, sizeof(attr));
}
```

성공 시 0을 반환한다. 항목을 찾지 못하면 -1을 반환하며 `errno`를 `ENOENT`로 설정한다.

#### `BPF_MAP_GET_NEXT_KEY`

`BPF_MAP_GET_NEXT_KEY` 명령은 파일 디스크립터 `fd`가 가리키는 맵에서 `key`로 항목을 찾아서 `next_key`가 가리키는 버퍼를 그 다음 항목의 키로 설정한다.

```c
int
bpf_get_next_key(int fd, const void *key, void *next_key)
{
    union bpf_attr attr = {
        .map_fd   = fd,
        .key      = ptr_to_u64(key),
        .next_key = ptr_to_u64(next_key),
    };

    return bpf(BPF_MAP_GET_NEXT_KEY, &attr, sizeof(attr));
}
```

`key`를 찾으면 동작이 0을 반환하며 `next_key`가 가리키는 버퍼를 다음 항목의 키로 설정한다. `key`를 찾지 못하면 동작이 0을 반환하며 `next_key`가 가리키는 버퍼를 첫 번째 항목의 키로 설정한다. `key`가 마지막 항목이면 -1을 반환하며 `errno`를 `ENOENT`로 설정한다. 가능한 다른 `errno` 값은 `ENOMEM`, `EFAULT`, `EPERM`, `EINVAL`이다. 이 방법을 사용해 맵의 항목 전체를 순회할 수 있다.

#### `close(map_fd)`

파일 디스크립터 `fd`가 가리키는 맵을 삭제한다. 맵을 생성한 사용자 공간 프로그램이 종료할 때 모든 맵들이 자동으로 삭제된다. (하지만 NOTES를 보라.)

### eBPF 맵 종류

다음 종류의 맵을 지원한다.

`BPF_MAP_TYPE_HASH`
:   해시 테이블 맵의 특징은 다음과 같다.

    * 사용자 공간 프로그램이 맵을 생성하고 없앤다. 사용자 공간 프로그램과 eBPF 프로그램 모두 검색, 갱신, 삭제 작업을 수행할 수 있다.

    * 커널이 키/값 쌍의 할당과 해제를 맡는다.

    * `max_entries` 한계에 도달하면 `map_update_elem()` 헬퍼가 삽입에 실패하게 된다. (그래서 eBPF 프로그램이 메모리를 고갈시키지 못한다.)

    * `map_update_elem()`이 기존 항목을 원자적으로 교체한다.

    해시 테이블 맵은 검색 속도에 최적화되어 있다.

`BPF_MAP_TYPE_ARRAY`
:   배열 맵의 특징은 다음과 같다.

    * 가능한 최고 속도의 검색에 최적화되어 있다. 향후에 검증기/JIT 컴파일러가 상수 키를 사용하는 `lookup()` 작업을 인식해서 이를 상수 포인터로 최적화할 수도 있다. 상수 아닌 키를 포인터 직접 계산으로 최적화하는 것도 가능한데, eBPF 프로그램의 수명 동안 포인터와 `value_size`가 고정돼 있기 때문이다. 다시 말해 검증기/JIT 컴파일러가 `array_map_lookup_elem()`을 '인라인'으로 만들면서도 여전히 사용자 공간에서 이 맵에 동시 접근 가능하도록 할 수 있다.

    * 모든 배열 항목들은 사전 할당되며 초기화 시점에 0으로 초기화 된다.

    * 키는 배열 색인이며 정확히 4바이트여야 한다.

    * `map_delete_elem()`이 `EINVAL` 오류로 실패한다. 항목들을 삭제할 수 없기 때문이다.

    * `map_update_elem()`이 **비원자적** 방식으로 항목을 교체한다. 원자적 갱신을 원하면 해시 테이블 맵을 사용해야 한다. 하지만 배열에서도 가능한 특별한 경우가 있는데, 32비트 및 64비트 원자 카운터에 원자적인 내장 `__sync_fetch_and_add()`를 사용할 수 있다. 예를 들어 값이 단일 카운터를 나타낸다면 값 전체에 적용할 수 있으며 여러 카운터를 담은 구조체인 경우에는 개별 카운터에 사용할 수 있을 것이다. 이벤트 합산 및 계수에 종종 유용하다.

    배열 맵을 다음과 같이 사용할 수 있다.

    * eBPF "전역" 변수: 한 항목짜리 배열에 키를 (색인) 0으로 하고 값을 '전역' 변수들의 집합으로 해서 eBPF 프로그램이 이를 이용해 이벤트 간에 상태를 유지할 수 있다.

    * 고정 항목들에 추적 이벤트 합산해 넣기.

    * 패킷 수나 패킷 크기 같은 네트워킹 이벤트 계수.

`BPF_MAP_TYPE_PROG_ARRAY` (리눅스 4.2부터)
:   프로그램 배열 맵은 특별한 종류의 배열 맵인데, 값으로 다른 eBPF 프로그램을 가리키는 파일 디스크립터만 담는다. 따라서 `key_size`와 `value_size` 모두 정확히 4바이트여야 한다. 이 맵은 `bpf_tail_call()` 헬퍼와 결합해서 사용한다.

    이게 뜻하는 바는 프로그램 배열 맵이 있는 eBPF 프로그램이 커널 쪽에서 다음을 호출하여 자기 프로그램 흐름을 해당 프로그램 배열 슬롯에 있는 프로그램의 흐름으로 교체할 수 있다는 것이다.

        void bpf_tail_call(void *context, void *prog_map,
                           unsigned int index);

    이 배열을 다른 eBPF 프로그램으로 가는 일종의 점프 테이블로 볼 수 있다. 그렇게 호출된 프로그램은 같은 스택을 재사용하게 된다. 새 프로그램으로 점프를 수행하고 나면 이전 프로그램으로는 더이상 돌아오지 않는다.

    (맵 슬롯에 유효한 프로그램 파일 디스크립터가 없거나, 지정한 검색 색인/키가 범위 밖이거나, 호출 깊이 제한 32번을 초과해서) 프로그램 배열의 주어진 색인에서 eBPF 프로그램을 찾을 수 없으면 현재 eBPF 프로그램 실행을 계속한다. 이 동작 방식을 이용해 기본 경우로 떨어지는 것을 구현할 수 있다.

    프로그램 배열 맵이 유용한 경우로 추적이나 네트워킹이 있는데, 개별 시스템 호출이나 프로토콜을 별개의 하위 프로그램에서 다루고 식별자를 맵 색인으로 사용할 수 있다. 이 방식으로 인해 성능이 향상될 수도 있으며 단일 eBPF 프로그램의 최대 인스트럭션 수 제한을 넘어설 수 있기도 한다. 가변적인 환경에서 사용자 공간 데몬이 예를 들어 전역 정책이 바뀌었을 때 런타임에 원자적으로 개별 하위 프로그램들을 새 버전으로 교체하여 전체 프로그램 동작을 바꿀 수도 있을 것이다.

### eBPF 프로그램

`BPF_PROG_LOAD` 명령을 사용해 eBPF 프로그램을 커널로 적재한다. 이 명령의 반환 값은 그 eBPF 프로그램에 연계된 새 파일 디스크립터이다.

```c
char bpf_log_buf[LOG_BUF_SIZE];

int
bpf_prog_load(enum bpf_prog_type type,
              const struct bpf_insn *insns, int insn_cnt,
              const char *license)
{
    union bpf_attr attr = {
        .prog_type = type,
        .insns     = ptr_to_u64(insns),
        .insn_cnt  = insn_cnt,
        .license   = ptr_to_u64(license),
        .log_buf   = ptr_to_u64(bpf_log_buf),
        .log_size  = LOG_BUF_SIZE,
        .log_level = 1,
    };

    return bpf(BPF_PROG_LOAD, &attr, sizeof(attr));
}
```

`prog_type`은 사용 가능한 프로그램 종류들 중 하나이다.

```c
enum bpf_prog_type {
    BPF_PROG_TYPE_UNSPEC,        /* 0은 유효하지 않은
                                    프로그램 종류로 예약 */
    BPF_PROG_TYPE_SOCKET_FILTER,
    BPF_PROG_TYPE_KPROBE,
    BPF_PROG_TYPE_SCHED_CLS,
    BPF_PROG_TYPE_SCHED_ACT,
    BPF_PROG_TYPE_TRACEPOINT,
    BPF_PROG_TYPE_XDP,
    BPF_PROG_TYPE_PERF_EVENT,
    BPF_PROG_TYPE_CGROUP_SKB,
    BPF_PROG_TYPE_CGROUP_SOCK,
    BPF_PROG_TYPE_LWT_IN,
    BPF_PROG_TYPE_LWT_OUT,
    BPF_PROG_TYPE_LWT_XMIT,
    BPF_PROG_TYPE_SOCK_OPS,
    BPF_PROG_TYPE_SK_SKB,
    BPF_PROG_TYPE_CGROUP_DEVICE,
    BPF_PROG_TYPE_SK_MSG,
    BPF_PROG_TYPE_RAW_TRACEPOINT,
    BPF_PROG_TYPE_CGROUP_SOCK_ADDR,
    BPF_PROG_TYPE_LWT_SEG6LOCAL,
    BPF_PROG_TYPE_LIRC_MODE2,
    BPF_PROG_TYPE_SK_REUSEPORT,
    BPF_PROG_TYPE_FLOW_DISSECTOR,
    /* 전체 목록은 /usr/include/linux/bpf.h를 보라. */
};
```

eBPF 프로그램 종류에 대한 자세한 내용은 아래를 보라.

`bpf_attr`의 나머지 비트들을 다음과 같이 설정한다.

 * `insns`는 `struct bpf_insn` 인스트럭션의 배열이다.

 * `insn_cnt`는 `insns`가 가리키는 프로그램 내 인스트럭션 개수이다.

 * `license`는 라이선스 문자열이며, `gpl_only`로 표시된 헬퍼 함수들을 호출하려면 GPL 호환이어야 한다. (라이선스 규칙이 커널 모듈과 같으므로 "Dual BSD/GPL" 같은 이중 라이선스를 사용할 수도 있다.)

 * `log_buf`는 호출자가 할당한 버퍼에 대한 포인터이며 커널 내 검증기가 여기에 검증 로그를 저장할 수 있다. 그 로그는 여러 행의 문자열이며 프로그램 작성자가 이를 확인하여 검증기가 어떻게 그 eBPF 프로그램이 안전하지 않다는 결론에 도달했는지 알 수 있다. 검증기가 발전함에 따라 출력 형식이 언제든 바뀔 수 있다.

 * `log_size`는 `log_buf`가 가리키는 버퍼의 크기이다. 버퍼 크기가 검증기 메시지를 모두 담기에 충분하지 않으면 -1을 반환하고 `errno`를 `ENOSPC`로 설정한다.

 * `log_level`은 검증기의 출력 상세 정도이다. 0 값은 검증기가 로그를 제공하지 않는다는 뜻이다. 이 경우 `log_buf`가 NULL 포인터여야 하고 `log_size`가 0이어야 한다.

`BPF_PROG_LOAD`가 반환한 파일 디스크립터에 <tt>[[close(2)]]</tt>를 적용하면 그 eBPF 프로그램을 제거하게 된다. (하지만 NOTES를 보라.)

eBPF 프로그램에서 맵에 접근할 수 있으므로 eBPF 프로그램들 사이에서, 그리고 eBPF 프로그램과 사용자 공간 프로그램 사이에서 데이터를 교환하는 데 맵을 쓴다. 예를 들어 eBPF 프로그램이 (kprobe, 패킷 같은) 다양한 이벤트를 처리하고서 맵에 데이터를 저장할 수 있고, 그러면 사용자 공간 프로그램이 그 맵에서 데이터를 가져올 수 있다. 반대로 사용자 공간 프로그램이 맵을 설정 메커니즘으로 사용할 수 있다. 맵을 어떤 값들로 채우면 eBPF 프로그램이 그걸 확인해서 그 값에 따라 도중에 동작 방식을 변경할 수 있다.

### eBPF 프로그램 종류

eBPF 프로그램 종류(`prog_type`)가 프로그램에서 호출할 수 있는 커널 헬퍼 함수들의 집합을 결정한다. 또한 프로그램 종류가 프로그램 입력(문맥)을, 즉 (eBPF 프로그램에게 첫 번째 인자로 전달되는 데이터 덩어리인) `struct bpf_context`의 형식을 결정한다.

예를 들어 추적 프로그램의 헬퍼 함수 집합은 소켓 필터 프로그램과 같지 않다. (물론 일부 헬퍼들은 공통일 수 있다.) 또 추적 프로그램의 입력(문맥)은 레지스터 값들의 집합인 반면 소켓 필터에서는 네트워크 패킷이다.

어떤 종류의 eBPF 프로그램에서 사용 가능한 함수들의 집합은 향후 커질 수도 있다.

다음 종류의 프로그램을 지원한다.

`BPF_PROG_TYPE_SOCKET_FILTER` (리눅스 3.19부터)
:   현재 `BPF_PROG_TYPE_SOCKET_FILTER`에서 쓸 수 있는 함수들은 다음과 같다.

        bpf_map_lookup_elem(map_fd, void *key)
                            /* map_fd에서 키 검색 */
        bpf_map_update_elem(map_fd, void *key, void *value)
                            /* 키/값 갱신 */
        bpf_map_delete_elem(map_fd, void *key)
                            /* map_fd에서 키 삭제 */

    `bpf_context` 인자는 `struct __sk_buff`에 대한 포인터이다.

`BPF_PROG_TYPE_KPROBE` (리눅스 4.1부터)
:   [작성 예정]

`BPF_PROG_TYPE_SCHED_CLS` (리눅스 4.1부터)
:   [작성 예정]

`BPF_PROG_TYPE_SCHED_ACT` (리눅스 4.1부터)
:   [작성 예정]

### 이벤트

프로그램을 적재하고 나면 이를 이벤트에 연계할 수 있다. 다양한 커널 서브시스템마다 이를 위한 각자의 방법이 있다.

리눅스 3.19부터 다음과 같이 호출하면 앞서 <tt>[[socket(2)]]</tt> 호출로 생성한 소켓 `sockfd`에 프로그램 `prog_fd`를 붙이게 된다.

```c
setsockopt(sockfd, SOL_SOCKET, SO_ATTACH_BPF,
           &prog_fd, sizeof(prog_fd));
```

리눅스 4.1부터 다음과 같이 호출하여 앞서 <tt>[[perf_event_open(2)]]</tt> 호출로 생성한 perf 이벤트 파일 디스크립터 `event_fd`에 파일 디스크립터 `prog_fd`가 가리키는 eBPF 프로그램을 붙일 수 있다.

```c
ioctl(event_fd, PERF_EVENT_IOC_SET_BPF, prog_fd);
```

## RETURN VALUE

성공 호출 시 반환 값은 작업에 따라 다르다.

`BPF_MAP_CREATE`
:   eBPF 맵에 연계된 새 파일 디스크립터.

`BPF_PROG_LOAD`
:   eBPF 프로그램에 연계된 새 파일 디스크립터.

다른 명령들
:   0.

오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`E2BIG`
:   eBPF 프로그램이 너무 크거나 맵이 `max_entries` 제한(최대 항목 수)에 도달했다.

`EACCES`
:   `BPF_PROG_LOAD`에서, 모든 프로그램 인스트럭션이 유효하지만 프로그램이 안전하지 않아 보여서 거부되었다. 허용 안 된 메모리 영역이나 초기화 안 된 스택/레지스터에 접근해서일 수도 있고 함수 제약이 실제 종류와 일치하지 않아서일 수도 있고 정렬 안 된 메모리 접근이 있어서일 수도 있다. 이 경우 `log_level = 1`로 `bpf()`를 다시 호출해서 검증기가 제시한 구체적 이유를 `log_buf`에서 확인해 보는 게 좋다.

`EBADF`
:   `fd`가 열린 파일 디스크립터가 아니다.

`EFAULT`
:   포인터들(`key`, `value`, `log_buf`, `insns`) 중 하나가 접근 가능한 주소 공간 밖이다.

`EINVAL`
:   `cmd`에 지정한 값을 이 커널이 알지 못한다.

`EINVAL`
:   `BPF_MAP_CREATE`에서, `map_type`이나 속성이 유효하지 않다.

`EINVAL`
:   `BPF_MAP_*_ELEM` 명령들에서, `union bpf_attr`의 필드들 중 일부를 이 명령에서 사용하지 않는데 0으로 설정돼 있지 않다.

`EINVAL`
:   `BPF_PROG_LOAD`에서, 유효하지 않은 프로그램 적재 시도를 나타낸다. 인식 불가능한 인스트럭션, 예약된 필드 사용, 범위 밖으로의 점프, 무한 루프, 알 수 없는 함수 호출 때문에 eBPF 프로그램이 유효하지 않다고 볼 수 있다.

`ENOENT`
:   `BPF_MAP_LOOKUP_ELEM`이나 `BPF_MAP_DELETE_ELEM`에서, 해당 `key`를 가진 항목을 찾을 수 없음을 나타낸다.

`ENOMEM`
:   충분한 메모리를 할당할 수 없다.

`EPERM`
:   충분한 특권 없이 (`CAP_SYS_ADMIN` 역능 없이) 호출을 했다.

## VERSIONS

리눅스 3.18에서 `bpf()` 시스템 호출이 처음 등장했다.

## CONFORMING TO

`bpf()` 시스템 호출은 리눅스 전용이다.

## NOTES

리눅스 4.4 전에선 모든 `bpf()` 명령에서 호출자에게 `CAP_SYS_ADMIN` 역능이 필요하다. 리눅스 4.4부터는 비특권 사용자가 `BPF_PROG_TYPE_SOCKET_FILTER` 타입의 제한된 프로그램과 연관 맵을 만들 수 있다. 하지만 그 맵에 커널 포인터를 저장할 수 없으며 현재 다음 헬퍼 함수들만 이용할 수 있다.

* `get_random`
* `get_smp_processor_id`
* `tail_call`
* `ktime_get_ns`

`/proc/sys/kernel/unprivileged_bpf_disabled` 파일에 1 값을 써넣어서 비특권 접근을 막을 수 있다.

eBPF 객체(맵, 프로그램)를 프로세스들 간에 공유할 수 있다. 예를 들면 <tt>[[fork(2)]]</tt> 후에 자식이 같은 eBPF 객체들을 가리키는 파일 디스크립터들을 물려받는다. 더불어 eBPF 객체를 가리키는 파일 디스크립터를 유닉스 도메인 소켓을 통해 전달할 수 있다. 그리고 eBPF 객체를 가리키는 파일 디스크립터를 평상시처럼 <tt>[[dup(2)]]</tt>이나 비슷한 호출을 이용해 복제할 수 있다. eBPF 객체를 가리키는 모든 파일 디스크립터가 닫힌 후에만 그 객체가 할당 해제된다.

eBPF 프로그램을 제약된 C로 작성해서 (`clang` 컴파일러를 이용해) eBPF 바이트코드로 컴파일 할 수 있다. 그 제약된 C에는 루프, 전역 변수, 가변 인자 함수, 부동 소수점, 함수 인자로 구조체 전달하기 같은 여러 기능들이 빠져 있다. 커널 소스 트리의 `samples/bpf/*_kern.c` 파일들에서 예를 볼 수 있다.

커널에는 성능 향상을 위해 eBPF 바이트코드를 네이티브 머신 코드로 변환하는 JIT(just-in-time) 컴파일러가 포함돼 있다. 리눅스 4.15 전의 커널에서는 JIT 컴파일러가 기본적으로 꺼져 있으며 `/proc/sys/net/core/bpf_jit_enable` 파일에 다음 정수 문자열 중 하나를 써넣어서 동작 방식을 제어할 수 있다.

| | |
| --- | --- |
| 0 | JIT 컴파일 끄기. (기본값) |
| 1 | 일반 컴파일. |
| 2 | 디버깅 모드. 생성된 명령 코드를 십육진수로 커널 로그로 찍는다. 그러면 커널 소스 트리에서 제공하는 `tools/net/bpf_jit_disasm.c` 프로그램을 이용해 그 명령 코드를 역어셈블 할 수 있다. |

리눅스 4.15부터 커널 구성에 `CONFIG_BPF_JIT_ALWAYS_ON` 옵션을 쓸 수 있다. 그렇게 하면 JIT 컴파일러가 항상 켜지며 `bpf_jit_enable`은 1로 초기화 되고 변경 불가능하다. (이 커널 구성 옵션은 BPF 인터프리터를 대상으로 하는 어느 스펙터 공격에 대한 완화책으로 나온 것이다.)

현재 다음 아키텍처들에서 eBPF JIT 컴파일러가 사용 가능하다.

* x86-64 (리눅스 3.18부터, cBPF는 리눅스 3.0부터)
* ARM32 (리눅스 3.18부터, cBPF는 리눅스 3.4부터)
* SPARC 32 (리눅스 3.18부터, cBPF는 리눅스 3.5부터)
* ARM-64 (리눅스 3.18부터)
* s390 (리눅스 4.1부터, cBPF는 리눅스 3.7부터)
* PowerPC 64 (리눅스 4.8부터, cBPF는 리눅스 3.1부터)
* SPARC 64 (리눅스 4.12부터)
* x86-32 (리눅스 4.18부터)
* MIPS 64 (리눅스 4.18부터, cBPF는 리눅스 3.16부터)
* riscv (리눅스 5.1부터)

## EXAMPLES

```c
/* bpf+소켓 예시:
 * 1. 256개 항목의 배열 맵 생성
 * 2. 수신 패킷 수를 세는 프로그램 적재
 *    r0 = skb->data[ETH_HLEN + offsetof(struct iphdr, protocol)]
 *    map[r0]++
 * 3. setsockopt()를 통해 raw 소켓에 prog_fd 연계
 * 4. 매초마다 수신한 TCP/UDP 패킷 개수 출력
 */
int
main(int argc, char **argv)
{
    int sock, map_fd, prog_fd, key;
    long long value = 0, tcp_cnt, udp_cnt;

    map_fd = bpf_create_map(BPF_MAP_TYPE_ARRAY, sizeof(key),
                            sizeof(value), 256);
    if (map_fd < 0) {
        printf("failed to create map '%s'\n", strerror(errno));
        /* 아마 루트로 실행하지 않아서 */
        return 1;
    }

    struct bpf_insn prog[] = {
        BPF_MOV64_REG(BPF_REG_6, BPF_REG_1),        /* r6 = r1 */
        BPF_LD_ABS(BPF_B, ETH_HLEN + offsetof(struct iphdr, protocol)),
                                /* r0 = ip->proto */
        BPF_STX_MEM(BPF_W, BPF_REG_10, BPF_REG_0, -4),
                                /* *(u32 *)(fp - 4) = r0 */
        BPF_MOV64_REG(BPF_REG_2, BPF_REG_10),       /* r2 = fp */
        BPF_ALU64_IMM(BPF_ADD, BPF_REG_2, -4),      /* r2 = r2 - 4 */
        BPF_LD_MAP_FD(BPF_REG_1, map_fd),           /* r1 = map_fd */
        BPF_CALL_FUNC(BPF_FUNC_map_lookup_elem),
                                /* r0 = map_lookup(r1, r2) */
        BPF_JMP_IMM(BPF_JEQ, BPF_REG_0, 0, 2),
                                /* if (r0 == 0) goto pc+2 */
        BPF_MOV64_IMM(BPF_REG_1, 1),                /* r1 = 1 */
        BPF_XADD(BPF_DW, BPF_REG_0, BPF_REG_1, 0, 0),
                                /* lock *(u64 *) r0 += r1 */
        BPF_MOV64_IMM(BPF_REG_0, 0),                /* r0 = 0 */
        BPF_EXIT_INSN(),                            /* return r0 */
    };

    prog_fd = bpf_prog_load(BPF_PROG_TYPE_SOCKET_FILTER, prog,
                            sizeof(prog) / sizeof(prog[0]), "GPL");

    sock = open_raw_sock("lo");

    assert(setsockopt(sock, SOL_SOCKET, SO_ATTACH_BPF, &prog_fd,
                      sizeof(prog_fd)) == 0);

    for (;;) {
        key = IPPROTO_TCP;
        assert(bpf_lookup_elem(map_fd, &key, &tcp_cnt) == 0);
        key = IPPROTO_UDP;
        assert(bpf_lookup_elem(map_fd, &key, &udp_cnt) == 0);
        printf("TCP %lld UDP %lld packets\n", tcp_cnt, udp_cnt);
        sleep(1);
    }

    return 0;
}
```

커널 소스 트리의 `samples/bpf` 디렉터리에 완전한 동작 코드들이 좀 있다.

## SEE ALSO

<tt>[[seccomp(2)]]</tt>, <tt>[[bpf-helpers(7)]]</tt>, <tt>[[socket(7)]]</tt>, `tc(8)`, <tt>[[tc-bpf(8)]]</tt>

커널 소스 파일 `Documentation/networking/filter.txt`에서 전통적 BPF와 확장 BPF 모두를 설명한다.

----

2021-03-22
