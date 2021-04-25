## NAME

get_mempolicy - 스레드의 NUMA 메모리 정책 가져오기

## SYNOPSIS

```c
#include <numaif.h>

long get_mempolicy(int *mode, unsigned long *nodemask,
                   unsigned long maxnode, void *addr,
                   unsigned long flags);
```

`-lnuma`로 링크.

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`get_mempolicy()`는 `flags` 설정에 따라서 호출 스레드 또는 메모리 주소의 NUMA 정책을 가져온다.

NUMA 머신에는 CPU들과 거리가 다른 여러 개의 메모리 컨트롤러가 있다. 메모리 정책은 스레드를 위한 메모리를 어느 노드에서 할당할지 규정한다.

`flags`를 0으로 지정하면 호출 스레드의 (<tt>[[set_mempolicy(2)]]</tt>로 설정한) 기본 정책에 대한 정보를 `mode` 및 `nodemask`가 가리키는 버퍼로 반환한다. 이 인자들로 반환되는 값으로 <tt>[[set_mempolicy(2)]]</tt> 하면 `get_mempolicy()` 호출 시점의 상태로 스레드의 정책을 복원할 수 있다. `flags`가 0일 때 `addr`은 NULL로 지정해야 한다.

`flags`에 `MPOL_F_MEMS_ALLOWED`를 지정하면 (리눅스 2.6.24부터 사용 가능) `mode` 인자는 무시하며 노드(메모리)들의 집합을 `nodemask`로 반환하는데, 그 노드들을 스레드에서 이후 <tt>[[mbind(2)]]</tt>나 (*모드 플래그* 없는) <tt>[[set_mempolicy(2)]]</tt> 호출에서 지정할 수 있다. `MPOL_F_MEMS_ALLOWED`를 `MPOL_F_ADDR`이나 `MPOL_F_NODE`와 함께 쓸 수 없다.

`flags`에 `MPOL_F_ADDR`을 지정하면 `addr`에 준 메모리 주소에 적용되는 정책에 대한 정보를 반환한다. <tt>[[mbind(2)]]</tt>나 <tt>[[numa(3)]]</tt>에서 설명하는 헬퍼 함수들 중 하나를 사용해 `addr`을 포함한 메모리 범위에 정책을 설정했다면 그 정책이 스레드의 기본 정책과 다를 수 있다.

`mode` 인자가 NULL이 아닌 경우 `get_mempolicy()`는 요청 받은 NUMA 정책의 정책 모드와 선택적인 *모드 플래그*를 그 인자가 가리키는 위치에 저장하게 된다. `nodemask`가 NULL이 아니면 정책에 연계된 노드 마스크를 그 인자가 가리키는 위치에 저장하게 된다. `maxnode`는 `nodemask`에 저장할 수 있는 노드 ID 수를 나타낸다. 즉 가장 큰 노드 ID 더하기 1이다. `maxnode`로 지정한 값을 항상 `sizeof(unsigned long)*8`의 배수로 올림 한다.

`flags`에 `MPOL_F_NODE`와 `MPOL_F_ADDR`을 모두 지정한 경우 `get_mempolicy()`는 주소 `addr`이 할당된 노드의 노드 ID를 `mode`가 가리키는 위치로 반환한다. 지정한 주소에 대해 할당된 페이지가 아직 없으면 스레드에서 그 주소에 읽기 (적재) 접근을 수행한 것처럼 페이지를 할당하고서 그 페이지가 할당된 노드의 ID를 반환한다.

`flags`에 `MPOL_F_NODE`는 지정하고 `MPOL_F_ADDR`은 지정하지 않았으며 스레드의 현재 정책이 `MPOL_INTERLEAVE`인 경우에는 스레드를 위해 할당된 내부 커널 페이지들의 인터리빙에 쓰일 다음 노드의 노드 ID를 NULL 아닌 `mode` 인자가 가리키는 위치에 반환하게 된다. <tt>[[mmap(2)]]</tt> 호출을 `MAP_PRIVATE` 플래그로 사용해 읽기 접근용으로 맵 한 프로세스 메모리 범위들과 `MAP_SHARED` 플래그로 사용해 모든 접근용으로 맵 한 메모리 범위들 내의 메모리 맵 파일들에 대한 페이지들이 거기 포함된다.

다른 플래그 값들은 예약돼 있다.

가능한 정책들에 대한 소개는 <tt>[[set_mempolicy(2)]]</tt>를 보라.

## RETURN VALUE

성공 시 `get_mempolicy()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `nodemask`와 `maxnode`로 지정한 메모리 범위의 일부 내지 전체가 접근 가능한 주소 공간 밖을 가리킨다.

`EINVAL`
:   `maxnode`로 지정한 값이 시스템에서 지원하는 노드 ID 수보다 작다. 또는 `flags`에 `MPOL_F_NODE`와 `MPOL_F_ADDR` 외의 값을 지정했다. 또는 `flags`에 `MPOL_F_ADDR`을 지정했고 `addr`이 NULL이거나, `flags`에 `MPOL_F_ADDR`을 지정하지 않았고 `addr`이 NULL이 아니다. 또는 `flags`에 `MPOL_F_ADDR`이 아니라 `MPOL_F_NODE`를 지정했고 현재 스레드 정책이 `MPOL_INTERLEAVE`가 아니다. 또는 `flags`에 `MPOL_F_MEMS_ALLOWED`를 `MPOL_F_ADDR`이나 `MPOL_F_NODE`와 함께 지정했다. (또 다른 `EINVAL` 경우들이 있다.)

## VERSIONS

리눅스 커널 버전 2.6.7에서 `get_mempolicy()` 시스템 호출이 추가되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. 라이브러리 지원에 대한 정보는 <tt>[[numa(7)]]</tt>를 보라.

## SEE ALSO

<tt>[[getcpu(2)]]</tt>, <tt>[[mbind(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[set_mempolicy(2)]]</tt>, <tt>[[numa(3)]]</tt>, <tt>[[numa(7)]]</tt>, `numactl(8)`

----

2021-03-22
