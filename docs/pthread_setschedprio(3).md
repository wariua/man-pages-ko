## NAME

pthread_setschedprio - 스레드의 스케줄링 우선순위 설정하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_setschedprio(pthread_t thread, int prio);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_setschedprio()` 함수는 스레드 `thread`의 스케줄링 우선순위를 `prio`에 지정한 값으로 설정한다. (반면 <tt>[[pthread_setschedparam(3)]]</tt>은 스레드의 스케줄링 정책과 우선순위 모두를 바꾼다.)

## RETURN VALUE

성공 시 이 함수는 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다. `pthread_setschedprio()`가 실패한 경우 `thread`의 스케줄링 우선순위가 바뀌지 않는다.

## ERRORS

`EINVAL`
:   지정한 스레드의 스케줄링 정책에서 `prio`가 유효하지 않다.

`EPERM`
:   호출자가 지정한 우선순위를 설정하기 위한 적절한 특권을 가지고 있지 않다.

`ESRCH`
:   ID가 `thread`인 스레드를 찾을 수 없다.

POSIX.1에서는 `pthread_setschedparam(3)`에서 `ENOTSUP` 오류("우선순위를 지원하지 않는 값으로 설정하려고 시도했음")도 적고 있다.

## VERSIONS

glibc 버전 2.3.4부터 이 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_setschedprio()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

스레드의 스케줄링 우선순위를 바꾸는 데 필요한 권한과 그 효과, 각 스케줄링 정책에서 허용하는 우선순위 범위에 대한 설명은 <tt>[[sched(7)]]</tt>를 보라.

## SEE ALSO

<tt>[[getrlimit(2)]]</tt>, <tt>[[sched_get_priority_min(2)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setinheritsched(3)]]</tt>, <tt>[[pthread_attr_setschedparam(3)]]</tt>, <tt>[[pthread_attr_setschedpolicy(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_self(3)]]</tt>, <tt>[[pthread_setschedparam(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2021-03-22
