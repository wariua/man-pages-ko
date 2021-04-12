## NAME

mmap, munmap - 파일이나 장치를 메모리로 맵 하거나 해제하기

## SYNOPSIS

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
int munmap(void *addr, size_t length);
```

기능 확인 매크로 요건에 대한 정보는 NOTES를 보라.

## DESCRIPTION

`mmap()`은 호출 프로세스의 가상 주소 공간 안에 새 매핑을 만든다. `addr`에는 새 매핑의 시작 주소를 지정한다. `length` 인자는 매핑의 길이(0보다 커야 함)를 나타낸다.

`addr`이 NULL이면 매핑을 생성할 (페이지에 정렬된) 주소를 커널에서 선택한다. 이게 새 매핑을 만드는 가장 이식성 좋은 방식이다. `addr`이 NULL이 아니면 커널에서 이를 매핑을 둘 위치에 대한 힌트로 받아들인다. 리눅스 커널에선 가까운 페이지 경계로 (하지만 항상 `/proc/sys/vm/mmap_min_addr`에 지정한 값 이상으로) 정한다. 새 매핑의 주소를 호출 결과로 반환한다.

파일 매핑의 내용은 (익명 매핑과 달리, 아래 `MAP_ANONYMOUS` 참고) 파일 디스크립터 `fd`가 가리키는 파일의 (또는 다른 객체의) 오프셋 `offset`의 `length` 개 바이트로 채워진다. `offset`은 `sysconf(_SC_PAGE_SIZE)`가 반환하는 페이지 크기의 배수여야 한다.

`mmap()` 호출 반환 후에 즉시 `fd`를 닫을 수 있다. 그렇게 해도 매핑이 무효화되지 않는다.

`prot` 인자는 매핑에 원하는 메모리 보호 방식을 기술한다. (파일의 열기 모드와 충돌하지 않아야 한다.) `PROT_NONE`이거나 다음 플래그들 중 하나 이상을 비트 OR 한 것이다.

`PROT_EXEC`
:   페이지를 실행할 수 있다.

`PROT_READ`
:   페이지를 읽을 수 있다.

`PROT_WRITE`
:   페이지에 쓸 수 있다.

`PROT_NONE`
:   페이지에 접근할 수 없다.

`flags` 인자는 매핑에 대한 갱신 내용이 같은 영역을 맵 하고 있는 다른 프로세스에게 보이는지 여부를, 그리고 그 갱신 내용이 기반 파일까지 가게 되는지 여부를 결정한다. 다음 중 정확히 한 개 값을 `flags`에 집어넣어서 동작 방식을 정한다.

`MAP_SHARED`
:   매핑을 공유한다. 매핑에 대한 갱신 내용이 같은 영역을 맵 하고 있는 다른 프로세스에게 보이며 (파일 기반 매핑인 경우) 그 갱신 내용이 기반 파일까지 간다. (갱신 내용이 기반 파일로 가는 시점을 정확히 제어하려면 <tt>[[msync(2)]]</tt>를 사용해야 한다.)

`MAP_SHARED_VALIDATE` (리눅스 4.15부터)
:   이 플래그의 동작 방식은 `MAP_SHARED`와 같되, `MAP_SHARED` 매핑에서는 `flags`에 모르는 플래그가 있으면 무시한다는 점이 다르다. 반면 `MAP_SHARED_VALIDATE`로 매핑을 만들 때는 전달된 플래그들이 모두 알려진 것인지 커널에서 검사해서 모르는 플래그가 있으면 `EOPNOTSUPP` 오류로 매핑이 실패한다. 또 일부 매핑 플래그(가령 `MAP_SYNC`)를 사용하려면 이 매핑 유형이 필요하다.

`MAP_PRIVATE`
:   비공유 copy-on-write 매핑을 만든다. 매핑에 대한 갱신 내용이 같은 파일을 맵 하고 있는 다른 프로세스에게 보이지 않으며 기반 파일까지 가지 않는다. `mmap()` 호출 후에 파일에서 일어난 변경 내용이 맵 된 영역에 보이는지 여부는 명세되어 있지 않다.

`MAP_SHARED`와 `MAP_PRIVATE` 모두 POSIX.1-2001 및 POSIX.1-2008에 기술돼 있다. `MAP_SHARED_VALIDATE`는 리눅스 확장이다.

더불어 `flags`에 다음 값들을 0개 이상 OR 할 수 있다.

`MAP_32BIT` (리눅스 2.4.20, 2.6부터)
:   매핑을 프로세스 주소 공간의 처음 2기가바이트 안에 넣는다. x86-64 상에서 64비트 프로그램에만 이 플래그를 지원한다. 스레드 스택을 메모리의 처음 2GB 내의 어딘가에 할당해서 일부 초기 64비트 프로세서들에서 문맥 전환 성능을 개선할 수 있도록 추가된 것이다. 요즘의 x86-64 프로세서에는 이런 성능 문제가 없기 때문에 이 플래그를 쓸 필요가 없다. `MAP_FIXED`가 설정돼 있는 경우 `MAP_32BIT` 플래그를 무시한다.

`MAP_ANON`
:   `MAP_ANONYMOUS`의 동의어. 제거 예정.

`MAP_ANONYMOUS`
:   매핑이 파일을 기반으로 하지 않으며 내용물이 0으로 채워진다. `fd` 인자를 무시한다. 하지만 일부 구현에서는 `MAP_ANONYMOUS` (또는 `MAP_ANON`) 지정 시 `fd`가 -1이기를 요구하므로 이식 가능한 응용에서는 그렇게 하는 게 좋다. `offset` 인자는 0이어야 할 것이다. `MAP_ANONYMOUS`를 `MAP_SHARED`와 결합해 쓰는 것은 리눅스 커널 2.4부터 지원한다.

`MAP_DENYWRITE`
:   이 플래그는 무시한다. (예전에는 (리눅스 2.0 및 그 전에서는) 기반 파일에 대한 쓰기가 `ETXTBUSY`로 실패해야 한다는 뜻이었다. 하지만 서비스 거부 공격의 원인이 됐다.)

`MAP_EXECUTABLE`
:   이 플래그는 무시한다.

`MAP_FILE`
:   호환용 플래그. 무시한다.

`MAP_FIXED`
:   `addr`을 힌트로 해석하지 않는다. 즉 정확히 그 주소에 매핑을 위치시킨다. `addr`이 적절히 정렬돼 있어야 한다. 대부분의 아키텍처에서는 페이지 크기의 배수이면 충분하지만 어떤 아키텍처에서는 추가 제약이 있을 수 있다. `addr`과 `len`으로 지정한 메모리 영역이 기존 매핑(들)의 페이지와 겹치는 경우에는 기존 매핑의 겹치는 부분을 버리게 된다. 지정한 주소를 사용할 수 없으면 `mmap()`이 실패하게 된다.

    이식성이 있기를 바라는 소프트웨어에서는 `MAP_FIXED` 플래그를 조심해서 사용해야 한다. 프로세스 메모리 매핑들의 정확한 배치가 커널 버전, C 라이브러리 버전, 운영 체제 릴리스에 따라 크게 다를 수 있다는 점에 유념해야 한다. *NOTES에 있는 이 플래그에 대한 내용을 주의해서 읽어 보라!*

`MAP_FIXED_NOREPLACE` (리눅스 4.17부터)
:   이 플래그의 동작 방식은 `addr` 강제라는 점에서 `MAP_FIXED`와 비슷한데, `MAP_FIXED_NOREPLACE`는 절대 기존 매핑 범위를 손상시키지 않는다는 점이 다르다. 요청 범위가 기존 매핑과 충돌하려는 경우에 이 호출은 `EEXIST` 오류로 실패한다. 따라서 이 플래그를 사용하면 (다른 스레드들에 대해) 원자적으로 주소 범위 매핑을 시도할 수 있다. 한 스레드만 성공하고 나머지는 실패를 보고하게 된다.

    참고로 `MAP_FIXED_NOREPLACE` 플래그를 인식하지 못하는 구식 커널들은 (기존 매핑과의 충돌을 탐지했을 때) 보통 "비-`MAP_FIXED`" 동작 방식으로 후퇴하게 된다. 즉 요청한 주소와 다른 주소를 반환하게 된다. 따라서 하위 호환성 있는 소프트웨어에서는 반환 주소가 요청 주소와 같은지 확인할 필요가 있다.

`MAP_GROWSDOWN`
:   스택에 쓰는 플래그다. 매핑이 메모리에서 아래 방향으로 늘어나야 한다고 커널 가상 메모리 시스템에게 알린다. 반환 주소는 프로세스 가상 주소 공간에 실제 생성한 메모리 영역보다 한 페이지 아래의 주소이다. 매핑 아래의 그 "방호 구역" 페이지 내의 주소를 건드리면 매핑이 한 페이지만큼 성장하게 된다. 그 성장이 반복될 수 있으며, 매핑이 아래쪽 다음 매핑의 최상단 페이지로 성장해야 하는 시점에 "방호 구역" 페이지를 건드리면 `SIGSEGV` 시그널이 발생하게 된다.

`MAP_HUGETLB` (리눅스 2.6.32부터)
:   "거대 페이지"로 매핑을 할당한다. 자세한 내용은 리눅스 커널 소스 파일 `Documentation/admin-guide/mm/hugetlbpage.rst`를 보라. 아래 NOTES도 참고.

`MAP_HUGE_2MB`, `MAP_HUGE_1GB` (리눅스 4.8부터)
:   여러 가지 hugetlb 페이지 크기를 지원하는 시스템에서 `MAP_HUGETLB`와 조합해 사용해서 다른 hugetlb 페이지 크기(각각 2MB와 1GB)를 선택한다.

    더 일반적으로는 오프셋 `MAP_HUGE_SHIFT`의 여섯 비트에 원하는 페이지 크기의 밑수 2 로그를 인코딩 해서 원하는 거대 페이지 크기를 설정할 수 있다. (이 비트 필드에서 0 값은 기본 거대 페이지 크기이다. 기본 거대 페이지 크기는 `/proc/meminfo`에 나오는 `Hugepagesize` 필드를 통해 알 수 있다.) 따라서 위 두 상수는 다음과 같이 정의돼 있다.

        #define MAP_HUGE_2MB    (21 << MAP_HUGE_SHIFT)
        #define MAP_HUGE_1GB    (30 << MAP_HUGE_SHIFT)

    `/sys/kernel/mm/hugepages`의 하위 디렉터리를 확인하면 시스템에서 지원하는 거대 페이지 크기들을 알 수 있다.

`MAP_LOCKED` (리눅스 2.5.37부터)
:   맵 한 영역을 <tt>[[mlock(2)]]</tt>과 같은 방식으로 고정하게 표시한다. 여기선 전체 범위를 채우려고 (미리 폴트) 시도는 하지만 실패한 경우에 `mmap()` 호출이 실패하지 않는다. 따라서 이후에 메이저 폴트가 발생할 수도 있다. 즉 <tt>[[mlock(2)]]</tt>만큼 의미론이 강력하지 않다. 매핑 초기화 후의 메이저 폴트를 허용할 수 없다면 `mmap()`에 <tt>[[mlock(2)]]</tt>을 더해서 쓰는 게 좋다. 구식 커널에서는 `MAP_LOCKED` 플래그를 무시한다.

`MAP_NONBLOCK` (리눅스 2.5.46부터)
:   이 플래그는 `MAP_POPULATE`와 함께 쓸 때만 의미가 있다. 미리 읽기를 수행하지 않는다. 즉 이미 램 내에 있는 페이지에만 페이지 테이블 항목을 만든다. 리눅스 2.6.23부터는 이 플래그를 쓰면 `MAP_POPULATE`가 아무것도 하지 않게 된다. 언젠가 `MAP_POPULATE`와 `MAP_NONBLOCK`의 조합이 재구현될 수도 있다.

`MAP_NORESERVE`
:   이 매핑을 위한 스왑 공간을 예비해 두지 않는다. 스왑 공간이 예비돼 있을 때는 매핑을 변경하는 게 가능하다고 보장된다. 스왑 공간이 예비돼 있지 않을 때는 사용 가능한 물리적 메모리가 없으면 쓰기 시 `SIGSEGV`를 받을 수도 있다. <tt>[[proc(5)]]</tt>의 `/proc/sys/vm/overcommit_memory` 파일 논의도 참고하라. 2.6 전의 커널에서는 비공유 쓰기 가능 매핑에만 이 플래그가 효과가 있었다.

`MAP_POPULATE` (리눅스 2.5.46부터)
:   매핑에 대한 페이지 테이블을 채운다 (미리 폴트). 파일 매핑인 경우 파일에서 미리 읽기를 하게 한다. 이후 페이지 폴트에서 막히는 걸 줄이는 데 도움이 될 것이다. 비공유 매핑에는 리눅스 2.6.23부터 `MAP_POPULATE`를 지원한다.

`MAP_STACK` (리눅스 2.6.27부터)
:   프로세스나 스레드 스택에 적합한 주소에 매핑을 할당한다. 이 플래그는 현재 no-op지만 glibc 스레딩 구현에서 사용하는데, 일부 아키텍처에서 스택 할당에 특별한 처리를 요하는 경우 나중에 투명하게 지원을 구현할 수 있기 위해서다.

`MAP_SYNC` (리눅스 4.15부터)
:   이 플래그는 `MAP_SHARED_VALIDATE` 매핑 유형에만 사용 가능하다. `MAP_SHARED` 유형 매핑에서는 이 플래그를 조용히 무시한다. DAX(영속 메모리 직접 매핑) 지원 파일에서만 이 플래그를 지원한다. 그렇지 않은 파일에서 이 플래그로 매핑을 만들려고 하면 `EOPNOTSUPP` 오류가 발생한다.

    이 플래그를 쓴 공유 파일 매핑에서는 프로세스 주소 공간에 어떤 메모리가 쓰기 가능하게 맵 되어 있는 동안에 시스템이 죽거나 재부팅 된 후에도 동일 파일의 동일 오프셋에서 그 내용을 볼 수 있다고 보장된다. 적절한 CPU 인스트럭션과 함께 사용하면 더 효율적인 방식으로 데이터 변경을 영속시키는 매핑이 가능해진다.

`MAP_UNINITIALIZED` (리눅스 2.6.33부터)
:   익명 페이지 내용을 비우지 않는다. 이 플래그는 임베디드 장치에서 성능을 개선하기 위한 것이다. 커널이 `CONFIG_MMAP_ALLOW_UNINITIALIZED` 옵션으로 구성된 경우에만 이 플래그를 따른다. 보안적 함의 때문에 보통 임베디드 장치에서만 (즉 사용자 메모리 내용물에 완전한 통제가 이뤄지는 장치에서만) 이 옵션을 켠다.

위 플래그들 중 `MAP_FIXED`만 POSIX.1-2001 및 POSIX.1-2008에 명세되어 있다. 하지만 많은 시스템에서 `MAP_ANONYMOUS`도 (또는 동의어인 `MAP_ANON`을) 지원한다.

`mmap()`으로 맵 한 메모리는 <tt>[[fork(2)]]</tt>를 거치며 같은 속성으로 유지된다.

파일은 페이지 크기의 배수로 맵 된다. 파일 크기가 페이지 크기의 배수가 아닌 경우 맵 할 때 나머지 메모리를 0으로 채우며 그 영역에 쓴 것이 파일에 기록되지 않는다. 매핑 기반 파일의 크기가 바뀔 때 파일에 추가되거나 제거되는 영역에 대응하는 페이지들에 어떤 영향을 주는지는 명세되어 있지 않다.

### `munmap()`

`munmap()` 시스템 호출은 지정한 주소 범위에 대한 매핑을 삭제하고 그 범위 내 주소에 대한 이후 참조가 비유효 메모리 참조를 일으키도록 한다. 프로세스가 종료할 때도 자동으로 해제한다. 반면 파일 디스크립터를 닫는 것으로는 맵을 해제하지 않는다.

주소 `addr`은 페이지 크기의 배수여야 한다. (`length`는 그럴 필요가 없다.) 지정한 범위의 일부라도 담은 모든 페이지들이 해제되며 이후 그 페이지들에 대한 참조가 `SIGSEGV`를 일으키게 된다. 지정한 범위 안에 맵 된 페이지가 없더라도 오류가 아니다.

## RETURN VALUE

성공 시 `mmap()`은 맵 한 영역에 대한 포인터를 반환한다. 오류 시 `MAP_FAILED` 값(즉 `(void *) -1`)을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다.

성공 시 `munmap()`은 0을 반환한다. 실패 시 -1을 반환하며 오류 원인(아마 `EINVAL`)을 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   파일 디스크립터가 비정규 파일을 가리키고 있다. 또는 파일 매핑을 요청했는데 `fd`가 읽기 가능하게 열려 있지 않다. 또는 `MAP_SHARED`를 요청했고 `PROT_WRITE`가 설정돼 있는데 `fd`가 읽기/쓰기 모드(`O_RDWR`)로 열려 있지 않다. 또는 `PROT_WRITE`가 설정돼 있는데 파일이 덧붙임 전용이다.

`EAGAIN`
:   파일이 잠겨 있거나 너무 많은 메모리가 고정돼 있다. (<tt>[[setrlimit(2)]]</tt> 참고.)

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니다. (그리고 `MAP_ANONYMOUS`를 설정하지 않았다.)

`EEXIST`
:   `flags`에 `MAP_FIXED_NOREPLACE`를 지정했으며 `addr`과 `length`가 나타내는 범위가 기존 매핑과 충돌한다.

`EINVAL`
:   `addr`, `length`, `offset`이 마음에 들지 않는다. (가령 너무 크거나 페이지 경계에 정렬돼 있지 않다.)

`EINVAL`
:   (리눅스 2.6.12부터) `length`가 0이다.

`EINVAL`
:   `flags`에 `MAP_PRIVATE`과 `MAP_SHARED` 어느 쪽도 없거나 둘 다 포함돼 있다.

`ENFILE`
:   열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.

`ENODEV`
:   지정한 파일의 기반 파일 시스템이 메모리 매핑을 지원하지 않는다.

`ENOMEM`
:   사용 가능한 메모리가 없다.

`ENOMEM`
:   프로세스의 매핑 최대 개수를 초과하려 했다. `munmap()`에서도 이 오류가 발생할 수 있는데, 기존 매핑의 중간 부분을 맵 해제하면 그 양쪽으로 작은 매핑 두 개가 생기기 때문이다.

`ENOMEM`
:   (리눅스 4.7부터) 프로세스의 `RLIMIT_DATA` 제한(<tt>[[getrlimit(2)]]</tt>에서 설명함)을 초과하려 했다.

`EOVERFLOW`
:   32비트 아키텍처에 큰 파일 확장(즉 64비트 `off_t`)을 쓰는 경우: `length`를 위한 페이지 개수에 `offset`을 위한 페이지 개수를 더하면 `unsigned long`(32비트)을 넘게 된다.

`EPERM`
:   `prot` 인자에서 `PROT_EXEC`를 요청하는데 맵 영역이 실행 불가능하게 마운트 된 파일 시스템 상의 파일에 속해 있다.

`EPERM`
:   파일 봉인 때문에 동작이 막혔다. <tt>[[fcntl(2)]]</tt> 참고.

`ETXTBSY`
:   `MAP_DENYWRITE`가 설정돼 있지만 `fd`로 지정한 객체가 쓰기 가능하게 열려 있다.

맵 한 영역을 사용할 때 다음 시그널이 발생할 수 있다.

`SIGSEGV`
:   읽기 전용으로 맵 한 영역에 쓰기를 시도했다.

`SIGBUS`
:   파일에 대응하지 않는 버퍼 부분에 접근을 시도했다. (예를 들어 파일 끝 너머에 접근하기. 다른 프로세스가 파일을 잘라내는 경우 포함.)

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mmap()`, `munmap()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.4BSD.

## AVAILABILITY

`mmap()`, <tt>[[msync(2)]]</tt>, `munmap()`이 사용 가능한 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_MAPPED_FILES`가 0보다 큰 값으로 정의되어 있다. (<tt>[[sysconf(3)]]</tt>도 참고.)

## NOTES

일부 하드웨어 아키텍처(가령 i386)에서는 `PROT_WRITE`가 `PROT_READ`를 함의한다. 아키텍처에 따라 `PROT_READ`가 `PROT_EXEC`를 함의하는지 여부가 다르다. 이식 가능한 프로그램에서는 새 매핑 내의 코드를 실행할 예정이면 항상 `PROT_EXEC`를 설정하는 게 좋다.

매핑을 만드는 이식성 있는 방법은 `addr`을 0(NULL)으로 지정하고 `flags`에서 `MAP_FIXED`를 빼는 것이다. 이렇게 하는 경우 매핑에 쓸 주소를 시스템이 선택한다. 기존 매핑과 충돌하지 않으며 0이 아니게 주소를 선택한다. `MAP_FIXED` 플래그를 지정하는 경우 `addr`이 0(NULL)이면 맵 한 주소가 0(NULL)이 된다.

어떤 `flags` 상수들은 적절한 기능 확인 매크로가 (때론 기본으로) 정의돼 있는 경우에만 정의돼 있다. glibc 2.19 및 이후에서는 `_DEFAULT_SOURCE`, glibc 2.19 및 이전에서는 `_BSD_SOURCE`나 `_SVID_SOURCE`이다. (`_GNU_SOURCE` 사용으로도 충분하며, 명확히 이 매크로를 요건으로 하는 게 더 논리적일 수도 있는 것이, 그 플래그들 모두 리눅스 전용이다.) 해당하는 플래그는 `MAP_32BIT`, `MAP_ANONYMOUS` (및 동의어인 `MAP_ANON`), `MAP_DENYWRITE`, `MAP_EXECUTABLE`, `MAP_FILE`, `MAP_GROWSDOWN`, `MAP_HUGETLB`, `MAP_LOCKED`, `MAP_NONBLOCK`, `MAP_NORESERVE`, `MAP_POPULATE`, `MAP_STACK`이다.

응용에서 <tt>[[mincore(2)]]</tt>를 사용하면 매핑의 어떤 페이지가 현재 버퍼/페이지 캐시에 상주 중인지 알아낼 수 있다.

### `MAP_FIXED` 안전하게 사용하기

`MAP_FIXED`를 안전하게 쓸 수 있는 유일한 경우는 `addr`과 `length`로 지정한 주소 범위가 이미 다른 매핑을 통해 잡혀 있을 때이다. 그 외 경우에서는 `MAP_FIXED` 사용이 극히 위험한데, 기존에 있던 매핑을 강제로 없애므로 다중 스레드 프로세스에서 자기 주소 공간을 오염시키기 쉽게 되기 때문이다.

예를 들어 스레드 A가 `/proc/<pid>/maps`를 살펴봐서 `MAP_FIXED`로 맵 할 수 있는 안 쓰는 주소 범위를 찾는 동안 스레드 B가 동시에 같은 주소 범위의 일부 내지 전체를 획득할 수 있다. 그러고서 스레드 A가 `mmap(MAP_FIXED)`를 사용하면 스레드 B가 만든 매핑을 실질적으로 손상시키게 된다. 이 경우에서 스레드 B가 꼭 매핑을 직접 생성해야 하는 건 아니다. 내부적으로 <tt>[[dlopen(3)]]</tt>을 써서 어떤 다른 공유 라이브러리를 적재하는 라이브러리 호출을 하는 것만으로 충분하다. 그 <tt>[[dlopen(3)]]</tt> 호출이 라이브러리를 프로세스의 주소 공간으로 맵 하게 된다. 뿐만 아니라 거의 모든 라이브러리 호출이 이 기법이나 단순 메모리 할당을 통해 주소 공간에 메모리 매핑을 추가하도록 구현되어 있을 수 있다. <tt>[[brk(2)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, PAM 라이브러리(http://www.linux-pam.org) 등이 그 예이다.

리눅스 4.17부터는 다중 스레드 프로그램에서 `MAP_FIXED_NOREPLACE` 플래그를 사용해서 기존 매핑으로 예약돼 있지 않은 고정 주소에 매핑을 만들려 할 때 위에서 설명한 위험을 피할 수 있다.

### 파일 기반 매핑에서의 타임스탬프 변경

파일 기반 매핑에서는 `mmap()`과 대응하는 맵 해제 사이의 어느 시점에도 맵 된 파일의 `st_atime` 필드가 갱신될 수 있다. 맵 된 페이지에 처음 참조할 때 그 필드가 아직 갱신되지 않았으면 갱신하게 된다.

`PROT_WRITE` 및 `MAP_SHARED`로 맵 한 파일의 `st_ctime` 및 `st_mtime` 필드는 맵 영역에 대한 쓰기 후에, 그리고 이후 `MS_SYNC` 내지 `MS_ASYNC` 플래그로 <tt>[[msync(2)]]</tt> 하는 경우 그 전에 갱신된다.

### 거대 페이지 (Huge TLB) 매핑

거대 페이지를 이용하는 매핑에서는 `mmap()`과 `munmap()` 인자에 대한 요건이 시스템 기본 페이지 크기를 쓰는 매핑에 대한 요건과 좀 다르다.

`mmap()`에서는 `offset`이 기반 거대 페이지 크기의 배수여야 한다. 시스템에서 `length`를 기반 거대 페이지 크기의 배수로 자동으로 맞춘다.

`munmap()`에서는 `addr`과 `length`가 모두 기반 거대 페이지 크기의 배수여야 한다.

### C 라이브러리/커널 차이

이 페이지에서는 glibc의 `mmap()` 래퍼 함수가 제공하는 인터페이스를 기술한다. 원래는 그 함수에서 이름이 같은 시스템 호출을 불렀다. 커널 2.4부터 그 시스템 호출이 <tt>[[mmap2(2)]]</tt>로 대체됐고, 그래서 요즘에는 glibc의 `mmap()` 래퍼 함수에서 적절히 조정한 `offset` 값으로 <tt>[[mmap2(2)]]</tt>를 부른다.

## BUGS

리눅스에서는 위의 `MAP_NORESERVE`에서 언급하는 것들을 보장하지 않는다. 기본적으로 시스템에 메모리가 부족할 때는 언제 어느 프로세스든 죽을 수 있다.

커널 2.6.7 전에서는 `prot`를 `PROT_NONE`으로 지정한 경우에만 `MAP_POPULATE` 플래그에 효력이 있다.

SUSv3에서는 `length`가 0이면 `mmap()`이 실패하는 게 좋다고 명세한다. 하지만 커널 2.6.12 전에서는 이 경우에 `mmap()`이 성공했다. 아무 매핑도 만들지 않고 호출이 `addr`을 반환했다. 커널 2.6.12부터는 이 경우에 `mmap()`이 `EINVAL` 오류로 실패한다.

POSIX에서는 객체 끝의 불완전한 페이지를 항상 시스템에서 0으로 채워야 하며 객체 끝 너머에서의 변경 내용을 시스템이 절대 기록하지 않는다고 명세한다. 리눅스에서 그런 객체 끝 다음의 불완전한 페이지에 데이터를 쓰는 경우 파일이 닫히고 맵이 해제된 후에 절대 파일에는 기록되지 않지만 데이터가 페이지 캐시 내에 남아서 이후 매핑에서 그 변경된 내용을 볼 수도 있다. 어떤 경우에는 맵 해제 전에 <tt>[[msync(2)]]</tt>를 호출해서 고칠 수도 있지만 <tt>[[tmpfs(5)]]</tt>에서는 (예를 들어 <tt>[[shm_overview(7)]]</tt>에 기록된 POSIX 공유 메모리 인터페이스를 쓸 때는) 통하지 않는다.

## EXAMPLE

다음 프로그램은 첫 번째 명령행 인자에 지정한 파일의 일부를 표준 출력으로 찍는다. 두 번째와 세 번째 명령행 인자의 오프셋 및 길이 값을 통해 찍을 범위를 지정한다. 프로그램에서 파일의 필요한 페이지들로 메모리 매핑을 만든 다음 `write(2)`를 써서 원하는 바이트들을 출력한다.

### 프로그램 소스

```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define handle_error(msg) \
    do { perror(msg); exit(EXIT_FAILURE); } while (0)

int
main(int argc, char *argv[])
{
    char *addr;
    int fd;
    struct stat sb;
    off_t offset, pa_offset;
    size_t length;
    ssize_t s;

    if (argc < 3 || argc > 4) {
        fprintf(stderr, "%s file offset [length]\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    fd = open(argv[1], O_RDONLY);
    if (Fd == -1)
        handle_error("open");

    if (fstat(fd, &sb) == -1)           /* 파일 크기 얻기 */
        handle_error("fstat");

    offset = atoi(argv[2]);
    pa_offset = offset & ~(sysconf(_SC_PAGE_SIZE) - 1);
        /* mmap() 오프셋이 페이지에 정렬돼 있어야 함 */

    if (offset >= sb.st_size) {
        fprintf(stderr, "offset is past end of file\n");
        exit(EXIT_FAILURE);
    }

    if (argc == 4) {
        length = atoi(argv[3]);
        if (offset + length > sb.st_size)
            length = sb.st_size - offset;
                /* 파일 끝 너머의 바이트는 표시할 수 없음 */

    } else {    /* length 인자 없음 ==> 파일 끝까지 표시 */
        length = sb.st_size - offset;
    }

    addr = mmap(NULL, length + offset - pa_offset, PROT_READ,
                MAP_PRIVATE, fd, pa_offset);
    if (addr == MAP_FAILED)
        handle_error("mmap");

    s = write(STDOUT_FILENO, addr + offset - pa_offset, length);
    if (s != length) {
        if (s == -1)
            handle_error("write");

        fprintf(stderr, "partial write");
        exit(EXIT_FAILURE);
    }

    munmap(addr, length + offset - pa_offset);
    close(fd);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[ftruncate(2)]]</tt>, <tt>[[getpagesize(2)]]</tt>, <tt>[[memfd_create(2)]]</tt>, <tt>[[mincore(2)]]</tt>, <tt>[[mlock(2)]]</tt>, <tt>[[mmap2(2)]]</tt>, <tt>[[mprotect(2)]]</tt>, <tt>[[mremap(2)]]</tt>, <tt>[[msync(2)]]</tt>, <tt>[[remap_file_pages(2)]]</tt>, <tt>[[setrlimit(2)]]</tt>, <tt>[[shmat(2)]]</tt>, <tt>[[userfaultfd(2)]]</tt>, <tt>[[shm_open(3)]]</tt>, <tt>[[shm_overview(7)]]</tt>

<tt>[[proc(5)]]</tt> 내의 `/proc/[pid]/maps`, `/proc/[pid]/map_files`, `/proc/[pid]/smaps` 파일에 대한 설명.

B.O. Gallmeister, POSIX.4, O'Reilly, 128-129쪽 및 389-391쪽.

----

2019-02-27
