## NAME

pthread_attr_setschedpolicy, pthread_attr_getschedpolicy - 스레드 속성 객체의 스케줄링 정책 속성 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setschedpolicy(pthread_attr_t *attr, int policy);
int pthread_attr_getschedpolicy(const pthread_attr_t *restrict attr,
                                int *restrict policy);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_setschedpolicy()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스케줄링 정책 속성을 `policy`에 지정한 값으로 설정한다. 이 속성은 스레드 속성 객체 `attr`을 이용해 생성하는 스레드의 스케줄링 정책을 결정한다.

`policy`에 지원하는 값은 `SCHED_FIFO`, `SCHED_RR`, `SCHED_OTHER`이며 <tt>[[sched(7)]]</tt>에서 그 의미를 기술한다.

`pthread_attr_getschedpolicy()`는 스레드 속성 객체 `attr`의 스케줄링 정책 속성을 `policy`가 가리키는 버퍼로 반환한다.

`pthread_attr_setschedpolicy()`로 설정한 정책이 <tt>[[pthread_create(3)]]</tt> 호출 때 효과가 있으려면 호출자가 <tt>[[pthread_attr_setinheritsched(3)]]</tt>를 사용해 속성 객체 `attr`의 스케줄러 상속 속성을 `PTHREAD_EXPLICIT_SCHED`로 설정해야 한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_attr_setschedpolicy()`가 다음 오류로 실패할 수 있다.

`EINVAL`
:   `policy`에 유효하지 않은 값.

POSIX.1에서는 `pthread_attr_setschedpolicy()`에서 선택적인 `ENOTSUP` 오류("속성을 지원하지 않는 값으로 설정하려고 시도했음")도 적고 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setschedpolicy()`,<br>`pthread_attr_getschedpolicy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## EXAMPLES

<tt>[[pthread_setschedparam(3)]]</tt> 참고.

## SEE ALSO

<tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setinheritsched(3)]]</tt>, <tt>[[pthread_attr_setschedparam(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_setschedparam(3)]]</tt>, <tt>[[pthread_setschedprio(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2021-03-22
