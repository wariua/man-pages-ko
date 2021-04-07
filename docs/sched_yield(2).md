## NAME

sched_yield - 프로세서 양보하기

## SYNOPSIS

```c
#include <sched.h>

int sched_yield(void);
```

## DESCRIPTION

`sched_yield()`는 호출 스레드가 CPU를 포기하게 한다. 스레드가 자기 고정 우선순위에 대한 큐의 끝으로 이동하며 새 스레드가 돌게 된다.

## RETURN VALUE

성공 시 `sched_yield()`는 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

리눅스 구현에서는 `sched_yield()`가 항상 성공한다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

호출 스레드가 그 시점에 가장 높은 우선순위 목록 내의 유일한 스레드이면 `sched_yield()` 호출 후에도 계속 돌게 된다.

`sched_yield()`를 사용할 수 있는 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_PRIORITY_SCHEDULING`이 정의되어 있다.

경쟁이 (많이) 있는 자원(가령 뮤텍스)을 호출자가 놓았을 때 다른 스레드 내지 프로세스에게 실행 기회를 주는 전략적 `sched_yield()` 호출로 성능을 향상시킬 수 있다. 불필요하게나 부적절하게 (가령 스케줄링 가능한 다른 스레드에게 필요한 자원을 호출자가 아직 잡고 있을 때) `sched_yield()`를 호출하는 것을 피해야 한다. 불필요한 문맥 전환이 발생하여 시스템 성능이 저하된다.

`sched_yield()`는 실시간 스케줄링 정책(즉 `SCHED_FIFO`나 `SCHED_RR`)에 사용하기 위한 것이다. `SCHED_OTHER` 같은 비결정적 스케줄링 정책에 `sched_yield()`를 사용하는 것은 명세되어 있지 않으며 응용 설계에 문제가 있다는 뜻일 가능성이 높다.

## SEE ALSO

<tt>[[sched(7)]]</tt>

----

2017-09-15
