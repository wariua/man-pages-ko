## NAME

pthread_attr_setinheritsched, pthread_attr_getinheritsched - 스레드 속성 객체의 스케줄러 상속 속성 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setinheritsched(pthread_attr_t *attr,
                                 int inheritsched);
int pthread_attr_getinheritsched(const pthread_attr_t *attr,
                                 int *inheritsched);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_setinheritsched()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스케줄러 상속 속성을 `inheritsched`에 지정한 값으로 설정한다. 스케줄러 상속 속성은 스레드 속성 객체 `attr`을 이용해 생성하는 스레드가 스케줄링 속성들을 호출 스레드로부터 물려받을지 아니면 `attr`에서 가져올지를 결정한다.

다음 속성들이 스케줄러 상속 속성의 영향을 받는다: 스케줄링 정책(<tt>[[pthread_attr_setschedpolicy(3)]]</tt>), 스케줄링 우선순위(<tt>[[pthread_attr_setschedparam(3)]]</tt>), 경합 범위(<tt>[[pthread_attr_setscope(3)]]</tt>).

`inheritsched`에 다음 값들을 지정할 수 있다.

`PTHREAD_INHERIT_SCHED`
:   `attr`을 이용해 생성하는 스레드가 호출 스레드로부터 스케줄링 속성들을 물려받는다. `attr` 내의 스케줄링 정책들은 무시한다.

`PTHREAD_EXPLICIT_SCHED`
:   `attr`을 이용해 생성하는 스레드가 속성 객체에 지정된 값들로부터 스케줄링 속성들을 가져온다.

새로 초기화 된 스레드 속성 객체에서 스케줄러 상속 속성의 기본 설정은 `PTHREAD_INHERIT_SCHED`이다.

`pthread_attr_getinheritsched()`는 스레드 속성 객체 `attr`의 스케줄러 상속 속성을 `inheritsched`가 가리키는 버퍼로 반환한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_attr_setinheritsched()`가 다음 오류로 실패할 수 있다.

`EINVAL`
:   `inheritsched`에 유효하지 않은 값.

POSIX.1에서는 `pthread_attr_setinheritsched()`에서 선택적인 `ENOTSUP` 오류("속성을 지원하지 않는 값으로 설정하려고 시도했음")도 적고 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setinheritsched()`,<br>`pthread_attr_getinheritsched()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## BUGS

glibc 2.8 기준으로 <tt>[[pthread_attr_init(3)]]</tt>으로 스케줄링 속성 객체를 초기화 하면 속성 객체의 스케줄링 정책이 `SCHED_OTHER`로 설정되고 스케줄링 우선순위가 0으로 설정된다. 그런데 그 상태에서 스케줄러 상속 속성을 `PTHREAD_EXPLICIT_SCHED`로 설정하면 그 속성 객체로 생성된 스레드가 생성을 하는 스레드로부터 스케줄링 속성들을 물려받는다. <tt>[[pthread_create(3)]]</tt> 호출 전에 명시적으로 스레드 속성의 스케줄링 정책이나 스케줄링 우선순위를 설정하면 이 버그가 발생하지 않는다.

## EXAMPLES

<tt>[[pthread_setschedparam(3)]]</tt> 참고.

## SEE ALSO

<tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setschedparam(3)]]</tt>, <tt>[[pthread_attr_setschedpolicy(3)]]</tt>, <tt>[[pthread_attr_setscope(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_setschedparam(3)]]</tt>, <tt>[[pthread_setschedprio(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2021-03-22
