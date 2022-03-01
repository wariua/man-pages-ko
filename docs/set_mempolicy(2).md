## NAME

set_mempolicy - 스레드와 그 자식들에 기본 NUMA 메모리 정책 설정하기

## SYNOPSIS

```c
#include <numaif.h>

long set_mempolicy(int mode, const unsigned long *nodemask,
                   unsigned long maxnode);
```

`-lnuma`로 링크.

## DESCRIPTION

`set_mempolicy()`는 호출 스레드의 NUMA 메모리 정책을 설정한다. 정책은 정책 모드와 0개 이상의 노드들로 이뤄지며 이를 `mode`, `nodemask`, `maxnode` 인자로 지정한 값들로 설정한다.

NUMA 머신에는 CPU들과 거리가 다른 여러 개의 메모리 컨트롤러가 있다. 메모리 정책은 스레드를 위한 메모리를 어느 노드에서 할당할지 규정한다.

이 시스템 호출은 스레드의 기본 정책을 지정한다. <tt>[[mbind(2)]]</tt>로 설정한 더 구체적인 정책의 통제를 받는 메모리 범위들 외의 프로세스 주소 공간에서 스레드 정책이 페이지 할당을 제어한다. 스레드 기본 정책은 또한 <tt>[[mmap(2)]]</tt> 호출을 `MAP_PRIVATE` 플래그로 사용해 맵 해서 그 스레드에서만 읽는(적재하는) 메모리 맵 파일과 접근 방식과 무관하게 <tt>[[mmap(2)]]</tt> 호출을 `MAP_SHARED` 플래그로 사용해 맵 한 메모리 맵 파일의 페이지 할당을 제어한다. 스레드를 위해 새 페이지를 할당할 때에만 정책이 적용된다. 익명 메모리에서는 스레드가 페이지를 처음 건드릴 때이다.

`mode` 인자에는 `MPOL_DEFAULT`, `MPOL_BIND`, `MPOL_INTERLEAVE`, `MPOL_PREFERRED`, `MPOL_LOCAL` (아래에서 자세히 설명함) 중 하나를 지정해야 한다. `MPOL_DEFAULT`를 제외한 모든 모드에서 호출자가 `nodemask` 인자를 통해 모드를 적용할 노드를 지정해야 한다.

`mode` 인자에 선택적으로 *모드 플래그*가 포함될 수 있다. 지원하는 *모드 플래그*는 다음과 같다.

`MPOL_F_STATIC_NODES` (리눅스 2.6.26부터)
:   비어 있지 않은 `nodemask`가 물리적 노드 ID들을 나타낸다. 프로세스가 다른 cpuset 문맥으로 이동하거나 프로세스의 현재 cpuset 문맥이 허용하는 노드 집합이 바뀔 때 리눅스에서 `nodemask`를 재사상 하지 않는다.

`MPOL_F_RELATIVE_NODES` (리눅스 2.6.26부터)
:   비어 있지 않은 `nodemask`가 프로세스의 현재 cpuset이 허용하는 노드 ID 집합에 대한 상대적인 노드 ID들을 나타낸다.

`nodemask`는 노드 ID들의 비트 마스크를 가리키며 그 비트 마스크에는 최대 `maxnode` 개까지의 비트가 담겨 있다. 비트 마스크 크기를 `sizeof(unsigned long)`의 다음 배수로 올림 하지만 커널에서는 `maxnode` 개까지의 비트만 사용한다. `nodemask` 값이 NULL이거나 `maxnode` 값이 0이면 빈 노드 집합을 나타낸다. `maxnode`의 값이 0이면 `nodemask` 인자를 무시한다.

`nodemask`가 필수인 경우에 그 인자는 온라인이고, (`MPOL_F_STATIC_NODES` 모드 플래그를 지정하지 않았으면) 프로세스의 현재 cpuset 문맥이 허용하고, 메모리를 담고 있는 노드를 적어도 한 개는 포함해야 한다. `mode`에 `MPOL_F_STATIC_NODES`가 설정돼 있고 필수인 `nodemask`에 프로세스의 현재 cpuset 문맥이 허용하는 노드가 하나도 담겨 있지 않으면 메모리 정책이 *지역 할당*으로 되돌아간다. 즉 프로세스의 cpuset 문맥이 `nodemask`에 지정한 노드를 하나 이상 포함하게 될 때까지는 실질적으로 지정한 정책을 무시한다.

`mode` 인자에 다음 값들 중 하나가 포함돼야 한다.

`MPOL_DEFAULT`
:   이 모드는 기본과 다른 스레드 메모리 정책을 모두 제거해서 메모리 정책을 시스템 기본 정책으로 "후퇴"시키게 한다. 시스템 기본 정책은 "지역 할당"이다. 즉 할당을 촉발한 CPU의 노드에서 메모리를 할당한다. `nodemask`를 NULL로 지정해야 한다. "지역 노드"에 유휴 메모리가 없으면 "인근" 노드에서 메모리를 할당하려 시도하게 된다.

`MPOL_BIND`
:   이 모드는 메모리 할당을 `nodemask`에 지정한 노드들로 제약하는 엄격한 정책을 지정한다. `nodemask`에 노드가 여러 개 있으면 노드 ID의 숫자 값이 가장 낮은 노드에서 페이지 할당을 한다. 그 노드에 유휴 메모리가 없어지면 `nodemask`에 지정한 다음으로 낮은 ID의 노드에서 할당을 하고, 그런 식으로 지정한 노드들 어디에도 유휴 메모리가 없을 때까지 진행한다. `nodemask`에 지정하지 않은 어떤 노드에서도 페이지를 할당하지 않는다.

`MPOL_INTERLEAVE`
:   이 모드에서는 `nodemask`에 지정한 노드들 내에서 노드 ID 순서에 따라 교대로 페이지 할당을 한다. 페이지들과 그에 대한 메모리 접근을 여러 노드로 분산시킴으로써 지연보다는 대역폭을 최적화 한다. 하지만 단일 페이지에 대한 접근은 여전히 단일 노드 메모리 대역폭으로 제한된다.

`MPOL_PREFERRED`
:   이 모드는 할당 선호 노드를 설정한다. 커널이 먼저 그 노드에서 페이지를 할당하려고 시도하지만 그 선호 노드에 유휴 메모리가 부족하면 "인근" 노드로 후퇴한다. `nodemask`에 노드 ID를 여러 개 지정하면 마스크의 첫 번째 노드를 선호 노드로 선택하게 된다. `nodemask` 및 `maxnode` 인자가 빈 집합을 나타내는 경우에는 이 정책이 (위에 설명한 시스템 기본 정책 같은) "지역 할당"을 지정한다.

`MPOL_LOCAL` (리눅스 3.8부터)
:   이 모드는 "지역 할당"을 나타낸다. 즉 할당을 촉발한 CPU의 노드("지역 노드")에서 메모리를 할당한다. `nodemask` 및 `maxnode` 인자가 빈 집합을 나타내야 한다. "지역 노드"에 유휴 메모리가 부족한 경우 커널은 다른 노드에서 메모리 할당을 시도하게 된다. 사용 가능한 메모리가 있는 경우에는 커널이 항상 "지역 노드"에서 메모리를 할당하게 된다. 프로세스의 현재 cpuset 문맥이 "지역 노드"를 허용하지 않으면 커널이 다른 노드에서 메모리 할당을 시도하게 된다. 프로세스의 현재 cpuset 문맥에서 허용하게 되면 커널이 항상 "지역 노드"에서 메모리를 할당하게 된다.

<tt>[[execve(2)]]</tt>를 거치면서 스레드 메모리 정책이 유지되며, <tt>[[fork(2)]]</tt>나 <tt>[[clone(2)]]</tt>으로 생성한 자식이 정책을 물려받는다.

## RETURN VALUE

성공 시 `set_mempolicy()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `nodemask`와 `maxnode`로 지정한 메모리 범위의 일부 내지 전체가 접근 가능한 주소 공간 밖을 가리킨다.

`EINVAL`
:   `mode`가 유효하지 않다. 또는 `mode`가 `MPOL_DEFAULT`이고 `nodemask`가 비어 있지 않거나, `mode`가 `MPOL_BIND`나 `MPOL_INTERLEAVE`이고 `nodemask`가 비어 있다. 또는 `maxnode`가 나타내는 비트들이 한 페이지를 넘는다. 또는 지원하는 가장 큰 노드 ID보다 큰 노드 ID를 `nodemask`에 한 개 이상 지정했다. 또는 `nodemask`로 지정한 노드 ID들 중에서 온라인이고 프로세스의 현재 cpuset 문맥에서 허용되는 게 없거나, 지정한 노드들 중에서 메모리를 담고 있는 게 없다. 또는 `mode` 인자에 `MPOL_F_STATIC_NODES`와 `MPOL_F_RELATIVE_NODES`를 함께 지정했다.

`ENOMEM`
:   사용 가능한 커널 메모리가 충분하지 않다.

## VERSIONS

리눅스 커널 버전 2.6.7에서 `set_mempolicy()` 시스템 호출이 추가되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

페이지를 스왑으로 내보낼 때 메모리 정책을 기억해 두지 않는다. 그 페이지를 되돌릴 때는 페이지 할당 시점에 적용 중인 스레드 내지 메모리 범위 정책을 사용하게 된다.

라이브러리 지원에 대한 정보는 <tt>[[numa(7)]]</tt>를 보라.

## SEE ALSO

<tt>[[get_mempolicy(2)]]</tt>, <tt>[[getcpu(2)]]</tt>, <tt>[[mbind(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[numa(3)]]</tt>, <tt>[[cpuset(7)]]</tt>, <tt>[[numa(7)]]</tt>, `numactl(8)`

----

2020-12-21