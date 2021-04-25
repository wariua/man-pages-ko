## NAME

sched_setparam, sched_getparam - 스케줄링 매개변수를 설정하고 얻기

## SYNOPSIS

```c
#include <sched.h>

int sched_setparam(pid_t pid, const struct sched_param *param);
int sched_getparam(pid_t pid, struct sched_param *param);

struct sched_param {
    ...
    int sched_priority;
    ...
};
```

## DESCRIPTION

`sched_setparam()`은 `pid`에 스레드 ID가 지정된 스레드의 스케줄링 정책에 연계된 스케줄링 매개변수를 설정한다. `pid`가 0이면 호출 스레드의 매개변수를 설정한다. `param` 인자의 해석 방식은 `pid`가 나타내는 스레드의 스케줄링 정책에 따라 달라진다. 리눅스에서 지원하는 스케줄링 정책들에 대한 설명은 <tt>[[sched(7)]]</tt>를 보라.

`sched_getparam()`은 `pid`가 나타내는 스레드의 스케줄링 매개변수를 가져온다. `pid`가 0이면 호출 스레드의 매개변수를 가져온다.

`sched_setparam()`은 스레드의 스케줄링 정책에 대해 `param`의 유효성을 확인한다. `param->sched_priority` 값이 <tt>[[sched_get_priority_min(2)]]</tt>과 <tt>[[sched_get_priority_max(2)]]</tt>에 의한 범위 내에 있어야 한다.

스케줄링 우선순위와 정책에 관련된 특권과 자원 제한에 대한 설명은 <tt>[[sched(7)]]</tt>를 보라.

`sched_setparam()`과 `sched_getparam()`을 사용할 수 있는 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_PRIORITY_SCHEDULING`이 정의되어 있다.

## RETURN VALUE

성공 시 `sched_setparam()`과 `sched_getparam()`은 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   유효하지 않은 인자: `param`이 NULL이거나 `pid`가 음수이다.

`EINVAL`
:   (`sched_setparam()`) 현재 스케줄링 정책에 대해 `param` 인자가 말이 되지 않는다.

`EPERM`
:   (`sched_setparam()`) 호출자가 적절한 특권을 가지고 있지 않다. (리눅스: `CAP_SYS_NICE` 역능을 가지고 있지 않다.)

`ESRCH`
:   ID가 `pid`인 스레드를 찾을 수 없다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## SEE ALSO

<tt>[[getpriority(2)]]</tt>, <tt>[[gettid(2)]]</tt>, <tt>[[nice(2)]]</tt>, <tt>[[sched_get_priority_max(2)]]</tt>, <tt>[[sched_get_priority_min(2)]]</tt>, <tt>[[sched_getaffinity(2)]]</tt>, <tt>[[sched_getscheduler(2)]]</tt>, <tt>[[sched_setaffinity(2)]]</tt>, <tt>[[sched_setattr(2)]]</tt>, <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[setpriority(2)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2021-03-22
