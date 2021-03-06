## NAME

BPF - 입력/출력 큐 규제를 위한 BPF 프로그램 분류자 및 행위

## SYNOPSIS

eBPF 분류자(필터) 또는 행위:
<pre>
<strong>tc filter</strong> ... <strong>bpf</strong> [ <strong>object-file</strong> OBJ_FILE ] [ <strong>section</strong> CLS_NAME ]
  [ <strong>export</strong> UDS_FILE ] [ <strong>verbose</strong> ] [ <strong>skip_hw</strong> | <strong>skip_sw</strong> ] [ <strong>police</strong> POLICE_SPEC ]
  [ <strong>action</strong> ACTION_SPEC ] [ <strong>classid</strong> CLASSID ]
<strong>tc action</strong> ... <strong>bpf</strong> [ <strong>object-file</strong> OBJ_FILE ] [ <strong>section</strong> CLS_NAME ]
  [ <strong>export</strong> UDS_FILE ] [ <strong>verbose</strong> ]
</pre>

cBPF 분류자(필터) 또는 행위:
<pre>
<strong>tc filter</strong> ... <strong>bpf</strong> [ <strong>bytecode-file</strong> BPF_FILE | <strong>bytecode</strong> BPF_BYTECODE ]
  [ <strong>police</strong> POLICE_SPEC ] [ <strong>action</strong> ACTION_SPEC ] [ <strong>classid</strong> CLASSID ]
<strong>tc action</strong> ... <strong>bpf</strong> [ <strong>bytecode-file</strong> BPF_FILE | <strong>bytecode</strong> BPF_BYTECODE ]
</pre>

## DESCRIPTION

확장 버클리 패킷 필터(**eBPF**)와 전통적 버클리 패킷 필터(원래는 그냥 BPF지만 구별을 위해 여기선 **cBPF**라고 부름) 모두를 완전히 프로그래밍 가능하고 매우 효율적인 분류자(classifier) 및 행위(action)로 사용할 수 있다. 둘 모두는 간단한 프로그램 구현을 위한 작은 인스트럭션 세트를 제공하며, 그 프로그램을 커널로 안전하게 적재하여 커널 공간의 작은 가상 머신에서 실행할 수 있다. 지정한 프로그램이 항상 종료하며 죽거나 커널 데이터를 누출시키지 않음을 커널 내 검증기가 보장한다.

리눅스에서는 일반적으로 eBPF가 cBPF의 뒤를 잇는다고 본다. 커널 내부에서는 cBPF 표현식을 eBPF 표현식으로 변환하여 실행한다. 실행을 인터프리터가 수행할 수도 있고 구성 시점에 저스트인타임(JIT) 컴파일 되어 네이티브 기계어로 돌 수도 있다. 현재 x86_64, ARM64, s390, ppc64, sparc64 아키텍처에서 eBPF JIT를 지원하며 PPC, SPARC, ARM, MIPS에는 cBPF는 지원하지만 (아직) eBPF JIT 지원으로 전환하지 않았다.

eBPF 인스트럭션 세트의 기반 원리는 cBPF 인스트럭션 세트와 비슷하되 더 나은 런타임 성능을 얻기 위해 기반 아키텍처에 좀 더 가깝게 설계해서 네이티브 인스트럭션 세트를 더 잘 흉내낼 수 있도록 하였다. 일대일 매핑으로 JIT 할 수 있게 설계되었는데, 덕분에 컴파일러가 eBPF 백엔드를 통해 네이티브 컴파일 코드에 가깝게 빠르게 동작하는 최적화된 eBPF 코드를 생성할 가능성까지 열린다. LLVM에서 그런 eBPF 백엔드를 제공하므로 eBPF 프로그램을 제한적 C 언어로 쉽게 작성할 수 있다. 이 외에도 eBPF 기반 구조에는 "맵"이라는 요소가 있다. eBPF 맵은 키/값 저장소인데, 이를 여러 eBPF 프로그램들 사이뿐 아니라 eBPF 프로그램과 사용자 공간 응용 사이에서도 공유한다.

트래픽 제어 서브시스템에서 입력 및 출력 qdisc에 붙일 수 있는 분류자와 행위를 eBPF나 cBPF로 작성할 수 있다. 다른 분류자와 행위들에 대한 장점은 eBPF/cBPF는 범용적 프레임워크를 제공하고 사용자가 특화된 용도에 따라 효율적으로 구현할 수 있다는 점이다. 그렇게 작성한 분류자 내지 행위는 과잉 기능으로 고생하지 않을 것이고, 따라서 자기 할 일을 매우 효율적으로 실행할 수 있다. 그리고 비선형 분류가 가능해지고 심지어 행위 부분을 분류로 병합하는 것도 가능하다. 이를 효율적인 eBPF 맵 자료 구조와 결합하면 예를 들어 사용자 공간에서 분류자 재적재 없이 커널로 classid 같은 새 정책을 밀어넣거나, 어느 맵에 저장된 통계를 수집해서 부하를 알아내고 다른 맵을 이용해 동적으로 부하 분산을 할 수 있다.

## PARAMETERS

### `object-file`

실행 및 링크 가능 형식(ELF)이고 eBPF 명령 코드와 eBPF 맵 정의를 담은 오브젝트 파일을 가리킨다. eBPF 분류자에 줄 수 있는 eBPF 오브젝트 생성을 지원하는 프로젝트로는 `clang(1)`을 C 언어 프론트엔드로 하는 LLVM 컴파일러 인프라가 있다. (자세한 내용은 **EXAMPLES** 절 참고.) eBPF 분류자 내지 행위를 적재할 때 이 옵션은 필수이다.

### `section`

오브젝트 파일에서 eBPF 분류자 내지 행위가 위치한 ELF 섹션의 이름이다. 분류자는 기본 섹션 이름이 "classifier"이고 행위는 "action"이다. 한 오브젝트 파일에 여러 분류자와 행위들을 넣을 수 있으므로 기본과 다른 경우 해당 섹션 이름을 명시해야 한다.

### `export`

유닉스 도메인 소켓 파일을 가리킨다. eBPF 오브젝트 파일에 eBPF 맵 명세를 담은 "maps"라는 섹션이 있는 경우 맵 파일 디스크립터를 유닉스 도메인 소켓을 통해 tc 종료 후 파일 디스크립터들을 관리하는 eBPF "에이전트"에게 넘겨줄 수 있다. import를 위한 IPC 동작을 구현하며 그 디스크립터들을 <tt>[[bpf(2)]]</tt> 호출에 사용해서 가령 모니터링이나 새 정책을 내리기 위해 사용자 공간에서 eBPF 맵 데이터를 읽거나 갱신하는 제3자 응용 프로그램이 에이전트가 될 수 있다.

### `verbose`

설정하면 eBPF 프로그램 적재가 성공한 경우에도 eBPF 검증기 출력을 찍는다. 기본적으로는 오류 시에만 사용자에게 검증기 로그를 출력한다.

### `skip_hw | skip_sw`

하드웨어 오프로드 제어 플래그. 기본적으로 TC는 가능한 경우 필터를 하드웨어로 오프로드 하려고 시도한다. `skip_hw`는 오프로드 시도를 명시적으로 비활성화한다. `skip_sw`는 오프로드를 강제하고 커널 내 eBPF 프로그램 실행을 비활성화한다. 이 플래그를 설정했는데 하드웨어 오프로드가 가능하지 않으면 커널이 오류를 알리고 필터가 아예 설치되지 않는다.

### `police`

eBPF/cBPF 분류자를 위한 선택적 매개변수이며, 가령 입력 qdisc 상의 분류자에 붙일 `tc(1)` 내 police를 지정한다.

### `action`

eBPF/cBPF 분류자를 위한 선택적 매개변수이며, 분류자에 붙일 `tc(1)` 내 후속 행위를 지정한다.

### `classid`, `flowid`

이 eBPF/cBPF 분류자를 위한 기본 트래픽 제어 클래스 식별자이다. 이 기본 클래스 식별자를 eBPF/cBPF 프로그램 반환 코드로 대체할 수도 있다. 기본 반환 코드 `-1`은 여기 제공한 기본 클래스 식별자를 사용하라는 뜻이다. 그리고 eBPF/cBPF 프로그램 반환 코드가 0이면 일치가 없었다는 의미다. 이 둘 외의 다른 반환 코드가 기본 classid를 대신하게 된다. 이를 통해 eBPF/cBPF 프로그램 하나만으로 효율적인 비선형 분류가 가능하다. 여러 클래스 식별자를 위해 프로그램을 여러 개 사용해서 프로그램마다 패킷 내용을 다시 파싱 하지 않아도 된다.

### `bytecode`

cBPF 분류자 및 행위 적재에만 사용하고 있다. cBPF 바이트코드를 '`s,c t f k,c t f k,c t f k,...`' 형태의 텍스트 문자열로 직접 전달한다. 여기서 `s`는 이어지는 4튜플들의 개수를 나타낸다. 4튜블 하나는 십진수로 된 `c t f k`로 이뤄지는데, `c`는 cBPF 명령 코드(opcode)를, `t`는 참일 때 점프 대상 오프셋을, `f`는 거짓일 때 점프 대상 오프셋을, `k`는 즉시적 상수/리터럴을 나타낸다. 이런 형식으로 코드를 생성해 주는 여러 도구들이 있는데, 예를 들어 리눅스 커널 소스 트리의 `tools/net/` 아래에 `bpf_asm`이 있다. 따라서 손으로 직접 작성할 필요가 없다. cBPF 분류자나 행위를 적재하려 할 때는 `bytecode`나 `bytecode-file` 옵션이 필수이다.

### `bytecode-file`

마찬가지로 cBPF 분류자 내지 행위를 적재하는 데 쓰인다. 효과는 `bytecode`와 같다. cBPF 바이트코드가 명령행으로 직접 전달되는 게 아니라 텍스트 파일 안에 있을 뿐이다.

## EXAMPLES

### eBPF 도구

eBPF 에이전트 코드를 포함한 제대로 된 예를 iproute2  소스 패키지의 `examples/bpf/`에서 찾을 수 있다.

전제 조건으로 커널에 <tt>[[bpf(2)]]</tt>라는 eBPF 시스템 호출이 활성화되어 있어야 하고 트래픽 제어 서브시스템을 위한 `cls_bpf` 및 `act_bpf` 커널 모듈이 있어야 한다. 아키텍처에서 지원하는 대로 eBPF/cBPF JIT 지원을 켜려면 다음과 같이 한다.

```sh
echo 1 > /proc/sys/net/core/bpf_jit_enable
```

LLVM을 통해 제약된 C 파일을 컴파일 할 수 있다.

```sh
clang -O2 -emit-llvm -c bpf.c -o - | llc -march=bpf -filetype=obj -o bpf.o
```

향후 컴파일러 호출 방법이 더 간단해질 수도 있을 테지만 당분간은 다음과 같이 에일리어스 해 두는 게 간편하다.

```sh
__bcc() {
        clang -O2 -emit-llvm -c $1 -o - | \
        llc -march=bpf -filetype=obj -o "`basename $1 .c` .o"
}

alias bcc=__bcc
```

모든 트래픽이 기본 classid로 (반환 코드 -1) 걸리는 가장 작은 단독 코드는 다음과 같다.

```c
#include <linux/bpf.h>

#ifndef __section
# define __section(x)  __attribute__((section(x), used))
#endif

__section("classifier") int cls_main(struct __sk_buff *skb)
{
        return -1;
}

char __license[] __section("license") = "GPL";
```

아래 **eBPF 프로그래밍** 부절에서 더 많은 예를 볼 수 있다. 여기선 도구에 집중한다.

행위 등을 위한 다양한 섹션들이 더 있을 수 있다. 그래서 eBPF로 된 오브젝트 파일에 여러 진입점이 있을 수 있다. 하지만 tc로 설정할 때는 구체적 진입점을 명시해야 한다. 그리고 제약된 C 코드에 라이선스가 포함되어야 하며 라이선스 문자열 문법은 리눅스 커널 모듈에서와 같다. 커널은 일부 eBPF 함수들을 GPL 호환 라이선스로만 제약할 권한을 가지며, 그래서 그런 라이선스 불일치 발생 시 프로그램을 커널로 적재하는 것을 거절할 수도 있다.

컴파일 해서 나오는 오브젝트 파일은 일반 오브젝트 파일에 사용하는 평범한 도구들로 살펴볼 수 있다. 가령 <tt>[[objdump(1)]]</tt>로 ELF 섹션 헤더들을 살펴볼 수 있다.

```text
objdump -h bpf.o
[...]
3 classifier    000007f8  0000000000000000  0000000000000000  00000040  2**3
                CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
4 action-mark   00000088  0000000000000000  0000000000000000  00000838  2**3
                CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
5 action-rand   00000098  0000000000000000  0000000000000000  000008c0  2**3
                CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
6 maps          00000030  0000000000000000  0000000000000000  00000958  2**2
                CONTENTS, ALLOC, LOAD, DATA
7 license       00000004  0000000000000000  0000000000000000  00000988  2**0
                CONTENTS, ALLOC, LOAD, DATA
[...]
```

기본 ELF 섹션에 분류자가 담긴 오브젝트 파일을 가지고 eBPF 분류자를 추가하는 건 간단하다. (참고로 "object-file" 대신 "obj"처럼 줄여 쓸 수도 있다.)

```text
bcc bpf.c
tc filter add dev em1 parent 1: bpf obj bpf.o flowid 1:1
```

분류자가 "mycls"라는 ELF 섹션에 있는 경우 같은 명령을 다음과 같이 호출해야 한다.

```text
tc filter add dev em1 parent 1: bpf obj bpf.o sec mycls flowid 1:1
```

클래스 설정을 찍으면 식별자의 위치가 나온다. 즉, 오브젝트 파일 "bpf.o"의 "mycls" 섹션에서 왔다고 알려준다.

```text
tc filter show dev em1
filter parent 1: protocol all pref 49152 bpf
filter parent 1: protocol all pref 49152 bpf handle 0x1 flowid 1:1 bpf.o:[mycls]
```

같은 프로그램을 출력이 아니라 입력 qdisc에 설치할 수도 있다.

```text
tc qdisc add dev em1 handle ffff: ingress
tc filter add dev em1 parent ffff: bpf obj bpf.o sec mycls flowid ffff:1
```

마찬가지로 찍어 볼 수 있다.

```text
tc filter show dev em1 parent ffff:
filter protocol all pref 49152 bpf
filter protocol all pref 49152 bpf handle 0x1 flowid ffff:1 bpf.o:[mycls]
```

분류자 및 행위를 입력에 붙일 때는 실제 기반 큐 규제가 없다는 제약이 있다. 입력에서 할 수 있는 건 패킷을 분류하거나, 변조하거나, 재지향하거나, 버리는 것이다. 입력 쪽에서 큐 사용이 필요할 때는 입력에서 패킷을 `ifb` 장비로 재지향해야 한다. 아니면 policing을 쓸 수 있다. 이 외에도 원치 않는 패킷이 네트워크 스택 상위 계층에 가기 전에 일찍 버리는 지점을 두거나, 출력과 공유할 수 있는 eBPF 맵들로 네트워크 사용 통계를 기록하거나, 이르게 조작하거나 다른 네트워크 장치로 재지향할 수 있는 지점을 두는 데 입력을 이용할 수 있다.

여러 eBPF 행위와 분류자들을 한 파일 내의 여러 섹션에 넣을 수 있다. 이 경우 기본과 다른 섹션 이름을 알려 주어야 하는데, 다음 예에서 두 행위 모두가 그렇다.

```text
tc filter add dev em1 parent 1: bpf obj bpf.o flowid 1:1 \
                         action bpf obj bpf.o sec action-mark \
                         action bpf obj bpf.o sec action-rand ok
```

이 방식의 장점은 프로그램에 eBPF 맵이 구현되어 있는 경우 분류자와 두 행위에서 공유할 수 있다는 점이다.

`tc(8)` 설정 후에도 사용자 공간에서 eBPF 맵에 접근할 수 있도록 하기 위해 유닉스 도메인 소켓을 통해 eBPF 에이전트로 소유권을 이전할 수 있다. 이를 구현할 수 있는 방법이 두 가지 있다.

1) 자체 eBPF 에이전트를 만들어서 직접 유닉스 도메인 소켓을 준비하고 `tc(8)`가 요구하는 프로토콜을 구현하기. iproute2 소스 패키지의 `examples/bpf/` 안에서 이를 위한 예시 코드를 볼 수 있다.

2) `tc exec`를 사용해서 유닉스 도메인 소켓으로 eBPF 맵 파일 디스크립터가 이전되면 `sh(1)` 같은 응용 프로세스가 생성되게 하기. 이 방식의 장점은 tc가 파일 디스크립터들을 실행 환경에 집어넣어 주므로 stdin, stdout, stderr 파일 디스크립터처럼 쓸 수 있다는 점이다. 이렇게 하면 그 fd 소유 셸 안에서 사용자 응용을 종료하고 재시작해도 eBPF 맵 파일 디스크립터가 사라지지 않는다. 앞의 분류자와 행위 조합으로 예를 들자면 다음과 같다.

```text
tc exec bpf imp /tmp/bpf
tc filter add dev em1 parent 1: bpf obj bpf.o exp /tmp/bpf flowid 1:1 \
                         action bpf obj bpf.o sec action-mark \
                         action bpf obj bpf.o sec action-rand ok
```

eBPF 맵을 분류자와 행위에서 공유한다고 하면 분류자나 행위 명령에서 한 번만 내보이면 충분하다. 오브젝트 파일을 처음 파싱 하는 시점에 tc가 eBPF 맵 파일 디스크립터들을 모두 준비해 줄 것이다.

셸이 새로 생성될 때 그 환경에 두 가지 eBPF 관련 변수가 있다. `BPF_NUM_MAPS`는 유닉스 도메인 소켓을 통해 이전된 맵들의 총개수를 알려 준다. 그리고 `BPF_MAP<X>`의 값은 eBPF 에이전트 응용에서 접근할 수 있는 파일 디스크립터 번호이다. 즉 이 값을 <tt>[[bpf(2)]]</tt> 시스템 호출에 그대로 파일 디스크립터로 사용해서 eBPF 맵 값들을 조회하거나 변경할 수 있다. `<X>`는 eBPF 맵의 식별자를 나타낸다. tc eBPF 맵 명세에서 `struct bpf_elf_map`의 `id` 멤버에 해당한다.

이 예에서는 환경이 다음과 같이 된다.

```text
sh# env | grep BPF
    BPF_NUM_MAPS=3
    BPF_MAP1=6
    BPF_MAP0=5
    BPF_MAP2=7
sh# ls -la /proc/self/fd
    [...]
    lrwx------. 1 root root 64 Apr 14 16:46 5 -> anon_inode:bpf-map
    lrwx------. 1 root root 64 Apr 14 16:46 6 -> anon_inode:bpf-map
    lrwx------. 1 root root 64 Apr 14 16:46 7 -> anon_inode:bpf-map
sh# my_bpf_agent
```

eBPF 에이전트를 이용하면 사용자 공간에서 eBPF 맵을 미리 채우고 맵을 통해 통계를 관찰하여 그 피드백에 따라서 가령 런타임 중에 eBPF 맵 값의 classid를 바꿔 쓸 수 있다. eBPF 에이전트는 일반적인 응용처럼 구현하므로 동적으로 외부 컨트롤러에게서 트래픽 제어 정책을 받아서 eBPF 맵으로 내려 주어 망 조건에 동적으로 적응할 수도 있다. 또한 eBPF 맵을 다른 종류의 (가령 추적을 위한) eBPF 프로그램들과 공유할 수도 있으므로 아주 강력한 조합을 구현할 수 있다.

### eBPF 프로그래밍

eBPF 분류자와 행위는 제약된 C 문법으로 구현한다. (향후 새로운 언어 프론트엔드가 추가로 지원될 수도 있다.)

헤더 파일 `linux/bpf.h`에 eBPF 프로그램에서 호출할 수 있는 eBPF 헬퍼 함수들이 있다. 이 맨 페이지에서는 최소한의 단독 예시 두 가지만 제공한다. eBPF의 가능성을 더 잘 보여 주는 제대로 된 흐름 분석 프로그램을 보려면 iproute2 소스 패키지의 `examples/bpf`를 살펴보면 된다.

지원하는 C 프로그램 32비트 분류자 반환 코드와 그 의미:
 * 0: 불일치를 나타낸다.
 * -1: 명령행에서 설정한 기본 classid를 나타낸다.
 * 그 외: 기본 classid를 대신하여 비선형 일치 검사가 가능하게 해 준다.

지원하는 C 프로그램 32비트 행위 반환 코드와 그 의미 (`linux/pkt_cls.h`):
 * `TC_ACT_OK` (0): 패킷 처리 파이프라인을 종료하게 되며 패킷이 계속 진행하도록 한다.
 * `TC_ACT_SHOT` (2): 패킷 처리 파이프라인을 종료하게 되며 패킷을 버린다.
 * `TC_ACT_UNSPEC` (-1): tc에서 설정한 기본 행위를 사용한다. (분류자의 -1 반환과 비슷하다.)
 * `TC_ACT_PIPE` (3): 다음 행위가 있으면 거기로 넘어가게 된다.
 * `TC_ACT_RECLASSIFY` (1): 패킷 처리 파이프라인을 종료하게 되며 패킷 분류를 처음부터 시작한다.
 * 그 외: 다른 값은 모두 비명세 반환 코드이다.

분류자 반환 코드와 행위 반환 코드 모두를 eBPF 및 eBPF 프로그램에서 사용할 수 있다.

조그만 분류기 예시를 통해 제약된 C 문법을 살펴보자. 여기선 가령 컨테이너에서 출발한 출력 패킷들에 [0, 255] 범위로 이미 표시가 되어 있다고 가정한다. 이 프로그램은 사용자 공간을 위해 표시 값에 대한 통계를 내며 표시 값 자체를 부 핸들로 해서 루트 qdisc로 classid를 매핑 한다.

```c
#include <stdint.h>
#include <asm/types.h>

#include <linux/bpf.h>
#include <linux/pkt_sched.h>

#include "helpers.h"

struct tuple {
        long packets;
        long bytes;
};

#define BPF_MAP_ID_STATS        1 /* 에이전트의 맵 식별자 */
#define BPF_MAX_MARK            256

struct bpf_elf_map __section("maps") map_stats = {
        .type           =       BPF_MAP_TYPE_ARRAY,
        .id             =       BPF_MAP_ID_STATS,
        .size_key       =       sizeof(uint32_t),
        .size_value     =       sizeof(struct tuple),
        .max_elem       =       BPF_MAX_MARK,
};

static inline void cls_update_stats(const struct __sk_buff *skb,
                                    uint32_t mark)
{
        struct tuple *tu;

        tu = bpf_map_lookup_elem(&map_stats, &mark);
        if (likely(tu)) {
                __sync_fetch_and_add(&tu->packets, 1);
                __sync_fetch_and_add(&tu->bytes, skb->len);
        }
}

__section("cls") int cls_main(struct __sk_buff *skb)
{
        uint32_t mark = skb->mark;

        if (unlikely(mark >= BPF_MAX_MARK))
                return 0;

        cls_update_stats(skb, mark);

        return TC_H_MAKE(TC_H_ROOT, mark);
}

char __license[] __section("license") = "GPL";
```

또 다른 작은 예는 목적 포트 80을 RSS 결과에 따라 [8080, 8087] 구간으로 역다중화하는 포트 리다이렉터인데 입력 qdisc에 붙일 수 있다. 출력 쪽과 IPv6 지원을 추가하는 것은 독자에게 연습으로 남겨둔다.

```c
#include <asm/types.h>
#include <asm/byteorder.h>

#include <linux/bpf.h>
#include <linux/filter.h>
#include <linux/in.h>
#include <linux/if_ether.h>
#include <linux/ip.h>
#include <linux/tcp.h>

#include "helpers.h"

static inline void set_tcp_dport(struct __sk_buff *skb, int nh_off,
                                 __u16 old_port, __u16 new_port)
{
        bpf_l4_csum_replace(skb, nh_off + offsetof(struct tcphdr, check),
                            old_port, new_port, sizeof(new_port));
        bpf_skb_store_bytes(skb, nh_off + offsetof(struct tcphdr, dest),
                            &new_port, sizeof(new_port), 0);
}

static inline int lb_do_ipv4(struct __sk_buff *skb, int nh_off)
{
        __u16 dport, dport_new = 8080, off;
        __u8 ip_proto, ip_vl;

        ip_proto = load_byte(skb, nh_off +
                             offsetof(struct iphdr, protocol));
        if (ip_proto != IPPROTO_TCP)
                return 0;

        ip_vl = load_byte(skb, nh_off);
        if (likely(ip_vl == 0x45))
                nh_off += sizeof(struct iphdr);
        else
                nh_off += (ip_vl & 0xF) << 2;

        dport = load_half(skb, nh_off + offsetof(struct tcphdr, dest));
        if (dport != 80)
                return 0;

        off = skb->queue_mapping & 7;
        set_tcp_dport(skb, nh_off - BPF_LL_OFF, __constant_htons(80),
                      __cpu_to_be16(dport_new + off));
        return -1;
}

__section("lb") int lb_main(struct __sk_buff *skb)
{
        int ret = 0, nh_off = BPF_LL_OFF + ETH_HLEN;

        if (likely(skb->protocol == __constant_htons(ETH_P_IP)))
                ret = lb_do_ipv4(skb, nh_off);

        return ret;
}

char __license[] __section("license") = "GPL";
```

둘 모두에서 사용하는 헬퍼 헤더 파일 `helpers.h`는 다음과 같다.

```c
/* 기타 헬퍼 매크로 */
#define __section(x) __attribute__((section(x), used))
#define offsetof(x, y) __builtin_offsetof(x, y)
#define likely(x) __builtin_expect(!!(x), 1)
#define unlikely(x) __builtin_expect(!!(x), 0)

/* 사용 맵 구조 */
struct bpf_elf_map {
    __u32 type;
    __u32 size_key;
    __u32 size_value;
    __u32 max_elem;
    __u32 id;
};

/* 사용 BPF 함수 몇 가지 */
static int (*bpf_skb_store_bytes)(void *ctx, int off, void *from,
                                  int len, int flags) =
     (void *) BPF_FUNC_skb_store_bytes;
static int (*bpf_l4_csum_replace)(void *ctx, int off, int from,
                                  int to, int flags) =
     (void *) BPF_FUNC_l4_csum_replace;
static void *(*bpf_map_lookup_elem)(void *map, void *key) =
     (void *) BPF_FUNC_map_lookup_elem;

/* 사용 BPF 내부 함수 몇 가지 */
unsigned long long load_byte(void *skb, unsigned long long off)
    asm ("llvm.bpf.load.byte");
unsigned long long load_half(void *skb, unsigned long long off)
    asm ("llvm.bpf.load.half");
```

최상 관행으로 권하는 것은 개별 분류자와 별도의 행위들을 만드는 대신 eBPF 분류자 하나만 tc로 적재해서 거기서 필요한 검사와 조작을 **모두** 수행하는 것이다. 주어진 용도에 맞춰진 분류자 하나만 실행하는 게 가장 효율적일 것이다.

### eBPF 디버깅

`bpf`를 위한 tc의 `filter`와 `action` 명령 모두에 선택적인 `verbose` 매개변수가 있어서 이를 이용해 eBPF 검증기 로그를 조사할 수 있다. 오류 시에는 기본으로 찍힌다.

eBPF/cBPF JIT 컴파일러를 켠 경우에 만들어진 명령 코드 이미지의 디버그 출력을 커널 로그로 찍도록 할 수도 있으며 `dmesg(1)`를 통해 이를 읽을 수 있다.

```sh
echo 2 > /proc/sys/net/core/bpf_jit_enable
```

리눅스 커널 소스 트리의 `tools/net/`에는 `bpf_jit_disasm`이라는 작은 헬퍼 프로그램이 있다. 커널 로그에서 명령 코드 이미지 덤프를 읽어서 역어셈블 결과를 찍어 준다.

```text
bpf_jit_disasm -o
```

그 외에도 리눅스 커널에는 `test_bpf`라는 광범위한 eBPF/cBPF 테스트 스위트 모듈도 포함돼 있다.

```text
modprobe test_bpf
```

이렇게 하면 다양한 테스트 케이스들을 수행해서 결과를 커널 로그로 찍는다. `dmesg(1)`로 이를 조사할 수 있다. JIT 컴파일러가 켜져 있는지 여부에 따라 결과가 다를 수 있다. 실패한 테스트 케이스가 있는 경우 모듈 적재가 실패하게 된다. 그 경우 관련 JIT 작성자들과 리눅스 커널 및 네트워킹 메일링 리스트로 버그를 보고해 주기를 강력히 권한다.

### cBPF

일반적으로 분류자와 행위를 **eBPF**로 구현하는 걸로 전환하기를 권한다. 다만 완전성을 위해 cBPF 프로그램 작성에 대한 여기 몇 마디를 남긴다.

마찬가지로 앞서 언급한 것처럼 `bpf_jit_enable` 스위치를 켤 수 있다. `bpf_jit_disasm` 같은 도구도 eBPF와 cBPF 어느 쪽 코드를 적재하려는지와 무관하다.

eBPF에서처럼 분류자와 행위를 제약된 C로 구현하지 않고 단순한 어셈블리 비슷한 언어로, 또는 다른 도구의 도움으로 구현한다.

tc의 저수준 인터페이스는 명령 코드를 직접 받는다. 예를 들어 모든 패킷에 걸려서 기본 classid 1:1을 반환하는 가장 단순한 분류자는 다음과 같다.

```text
tc filter add dev em1 parent 1: bpf bytecode '1,6 0 0 4294967295,' flowid 1:1
```

바이트코드 열의 첫 번째 십진수 수는 이어지는 4튜플 cBPF 명령 코드들의 개수를 나타낸다. 언급한 것처럼 4튜플은 `c t f k`라는 십진수들로 이뤄지며, `c`는 cBPF 명령 코드를, `t`는 참일 때 점프 대상 오프셋을, `f`는 거짓일 때 점프 대상 오프셋을, `k`는 즉시적 상수/리터럴을 나타낸다. 위 명령 코드는 즉시 값 -1로 프로그램에서 무조건 반환하는 것을 나타낸다.

출력 분류에 있어선, Willem de Bruijn이 `iptables(8)` BPF 확장을 위해 GNU 일반 공중 사용 허가서 버전 2로 구현한 간단한 단독형 헬퍼 프로그램이 있는데 `libpcap` 내부의 전통적 BPF 컴파일러를 이용한다. 여기 코드는 `tc(8)`와의 사용을 위해 그의 코드에서 파생된 것이다.

```c
#include <pcap.h>
#include <stdio.h>

int main(int argc, char **argv)
{
        struct bpf_program prog;
        struct bpf_insn *ins;
        int i, ret, dlt = DLT_RAW;

        if (argc < 2 || argc > 3)
                return 1;
        if (argc == 3) {
                dlt = pcap_datalink_name_to_val(argv[1]);
                if (dlt == -1)
                        return 1;
        }

        ret = pcap_compile_nopcap(-1, dlt, &prog, argv[argc - 1],
                                  1, PCAP_NETMASK_UNKNOWN);
        if (ret)
                return 1;

        printf("%d,", prog.bf_len);
        ins = prog.bf_insns;

        for (i = 0; i < prog.bh_len - 1; ++ins, ++i)
                printf("%u %u %u %u,", ins->code,
                       ins->jt, ins->jf, ins->k);
        printf("%u %u %u %u",
               ins->code, ins->jt, ins->jf, ins->k);

        pcap_freecode(&prog);
        return 0;
}
```

이 작은 헬퍼가 있으면 분류자에 어떤 `tcpdump(8)` 필터 식도 사용할 수 있다. 일치하면 기본 classid를 반환한다.

```text
bpftool EN10MB 'tcp[tcpflags] & tcp-syn != 0' > /var/bpf/tcp-syn
tc filter add dev em1 parent 1: bpf bytecode-file /var/bpf/tcp-syn flowid 1:1
```

기본적으로 이 작은 생성기는 다음과 동등하다.

```text
tcpdump -iem1 -ddd 'tcp[tcpflags] & tcp-syn != 0' | tr '\n' ',' > /var/bpf/tcp-syn
```

`libpcap`의 컴파일러에서는 리눅스 한정 cBPF 확장들을 모두 지원하지 않는다. 그래서 리눅스 커널에는 `tools/net/` 아래에 `bpf_asm`이라는 간단한 BPF 어셈블러가 있어서 완전한 제어가 가능하게 해 준다. 그런 프로그램을 직접 작성하기 위한 자세한 문법과 의미론은 **FURTHER READING**의 참고 자료를 보라.

IPv4/TCP 패킷을 분류하기 위한 `bpf_asm` 형식의 간단한 예가 `foobar`라는 텍스트 파일에 저장되어 있다고 하자.

```text
ldh [12]
jne #0x800, drop
ldb [23]
jneq #6, drop
ret #-1
drop: ret #0
```

마찬가지로 다음과 같이 그 분류자를 적재할 수 있다.

```text
bpf_asm foobar > /var/bpf/tcp-syn
tc filter add dev em1 parent 1: bpf bytecode-file /var/bpf/tcp-sync flowid 1:1
```

BPF 분류자를 위해 리눅스 커널에서는 `tools/net/` 아래에 `bpf_dbg`라는 작은 BPF 디버거를 추가로 제공한다. 이를 이용해 pcap 파일에 분류자를 돌려 보고, 전통적 프로그램을 단계 실행 하거나 다양한 중지점을 추가하고, 런타임에 레지스터 내용을 찍어 볼 수 있다.

전통적 BPF로 행위를 구현하는 것에는 상당한 제약이 있다. 패킷 변경을 지원하지 않기 때문이다. 따라서 가능하다면 eBPF로 전환하기를 일반적으로 권장한다.

## FURTHER READING

리눅스 커널 소스 트리의 `Documentation/networking/filter.txt`에서 BPF 체계에 대한 더 많은 기술적 세부 내용을 볼 수 있다.

iproute2 소스 트리의 `examples/bpf/` 내에서 더 자세한 eBPF `tc(8)` 예들을 볼 수 있다.

## SEE ALSO

`tc(8)`, `tc-ematch(8)`, <tt>[[bpf(2)]]</tt>, `bpf(4)`

## AUTHORS

Daniel Borkmann이 맨 페이지 작성함.

수정 내지 개선 사항은 리눅스 커널 네트워킹 메일링 리스트 <netdev@vger.kernel.org>로 알려 주기 바란다.

----

2015년 5월 18일
