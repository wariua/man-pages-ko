## NAME

getcpu - 호출 스레드가 현재 돌고 있는 CPU 및 NUMA 노드 알아내기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <sched.h>

int getcpu(unsigned int *cpu, unsigned int *node)
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`getcpu()` 시스템 호출은 호출 스레드 내지 프로세스가 현재 돌고 있는 프로세서 및 노드를 확인하여 이를 `cpu` 및 `node` 인자가 가리키는 정수들에 써넣는다. 프로세서는 CPU를 식별해 주는 유일한 작은 정수이다. 노드는 NUMA 노드를 식별해 주는 유일한 작은 식별자이다. `cpu`나 `node` 중 한쪽이 NULL이면 해당 포인터로는 써넣지 않는다.

`cpu`에 들어가는 정보가 최신이라고 보장되는 것은 호출 시점만이다. <tt>[[sched_setaffinity(2)]]</tt>를 이용해 CPU 친화성을 고정하지 않았다면 커널이 언제든 CPU를 바꾸었을 수 있다. (스케줄러에서 캐시 온도 유지를 위해 CPU 사이 이동을 최소화하려 하기 때문에 보통은 그런 일이 일어나지 않는다. 하지만 가능하다.) 호출자는 `cpu` 및 `node`로 반환된 정보가 호출 반환 시점에는 더 이상 최신이 아닐 가능성을 감안해야 한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   인자가 호출 프로세스의 주소 공간 밖을 가리키고 있다.

## VERSIONS

x86-64 및 i386에서는 커널 2.6.19에서 `getcpu()`가 추가되었다. glibc 2.29에서 라이브러리 지원이 추가되었다. (그 전 glibc 버전에서는 이 시스템 호출의 래퍼를 제공하지 않아서 <tt>[[syscall(2)]]</tt>을 써야 했다.)

## CONFORMING TO

`getcpu()`는 리눅스 전용이다.

## NOTES

리눅스에서는 이 호출을 가급적 빠르게 만들려고 노력한다. (일부 아키텍처에서는 <tt>[[vdso(7)]]</tt> 내 구현을 통해 그리한다.) `getcpu()`의 목적은 프로그램에서 CPU별 데이터 관련 최적화나 NUMA 최적화를 할 수 있게 하는 것이다.

### C 라이브러리/커널 차이

커널 시스템 호출에는 세 번째 인자가 있다.

```c
int getcpu(unsigned int *cpu, unsigned int *node,
           struct getcpu_cache *tcache);
```

`tcache` 인자는 리눅스 2.6.24 이후로 쓰이지 않는다. (시스템 호출을 직접 호출하는 경우) 리눅스 2.6.23 내지 이전 버전에 대한 이식성이 필요한 게 아니라면 NULL로 지정하는 게 좋다.

리눅스 2.6.23 내지 이전에서는 `tcache` 인자가 NULL이 아니면 스레드 로컬 저장소 내에 호출자가 할당한 버퍼에 대한 포인터를 나타냈으며 `getcpu()`를 위한 캐싱 메커니즘에 그 버퍼를 사용했다. 캐시를 사용하면 `getcpu()` 호출을 빠르게 만들 수 있었지만 그에 대한 비용은 반환된 정보가 구식이 될 가능성이 아주 조금 있다는 것이었다. CPU 사이에서 스레드를 이전할 때 캐싱 메커니즘이 문제를 유발한다고 보아서 지금은 그 인자를 무시한다.

## SEE ALSO

<tt>[[mbind(2)]]</tt>, <tt>[[sched_setaffinity(2)]]</tt>, <tt>[[set_mempolicy(2)]]</tt>, <tt>[[sched_getcpu(3)]]</tt>, <tt>[[cpuset(7)]]</tt>, <tt>[[vdso(7)]]</tt>

----

2021-03-22
