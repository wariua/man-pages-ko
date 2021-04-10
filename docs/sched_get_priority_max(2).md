## NAME

sched_get_priority_max, sched_get_priority_min - 고정 우선순위 범위 얻기

## SYNOPSIS

```c
#include <sched.h>

int sched_get_priority_max(int policy);

int sched_get_priority_min(int policy);
```

## DESCRIPTION

`sched_get_priority_max()`는 `policy`가 나타내는 스케줄링 알고리듬과 함께 사용할 수 있는 우선순위 최댓값을 반환한다. `sched_get_priority_min()`은 `policy`가 나타내는 스케줄링 알고리듬과 함께 사용할 수 있는 우선순위 최솟값을 반환한다. 지원하는 `policy` 값은 `SCHED_FIFO`, `SCHED_RR`, `SCHED_OTHER`, `SCHED_BATCH`, `SCHED_IDLE`, `SCHED_DEADLINE`이다. 이 정책들에 대한 더 자세한 내용을 <tt>[[sched(7)]]</tt>에서 볼 수 있다.

높은 수치 우선순위의 프로세스가 낮은 수치 우선순위의 프로세스보다 먼저 스케줄링 된다. 따라서 `sched_get_priority_max()`가 반환하는 값이 `sched_get_priority_min()`이 반환하는 값보다 크게 된다.

리눅스에서는 `SCHED_FIFO` 및 `SCHED_RR` 정책에 1에서 99까지의 고정 우선순위 범위를, 그리고 나머지에 우선순위 0을 준다. 여러 정책들의 스케줄링 우선순위 범위는 변경 불가능하다.

다른 POSIX 시스템에서는 스케줄링 우선순위 범위가 다를 수 있으므로 이식 가능한 응용에서는 가상의 우선순위 범위를 사용하여 그 범위를 `sched_get_priority_max()` 및 `sched_get_priority_min()`이 준 구간으로 사상할 수 있겠다. POSIX.1에서는 `SCHED_FIFO` 및 `SCHED_RR`에 대한 최솟값과 최댓값 사이 범위가 적어도 우선순위 32개만큼은 되기를 요구한다.

`sched_get_priority_max()`와 `sched_get_priority_min()`을 사용할 수 있는 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_PRIORITY_SCHEDULING`이 정의되어 있다.

## RETURN VALUE

성공 시 `sched_get_priority_max()`와 `sched_get_priority_min()`은 지정한 스케줄링 정책에서의 최대/최소 우선순위 값을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`EINVAL`
:   `policy` 인자가 정의되어 있는 스케줄링 정책을 나타내지 않는다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## SEE ALSO

<tt>[[sched_getaffinity(2)]]</tt>, <tt>[[sched_getparam(2)]]</tt>, <tt>[[sched_getscheduler(2)]]</tt>, <tt>[[sched_setaffinity(2)]]</tt>, <tt>[[sched_setparam(2)]]</tt>, <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[sched(7)]]</tt>

----

2017-09-15
