## NAME

pthread_attr_setschedparam, pthread_attr_getschedparam - 스레드 속성 객체의 스케줄링 매개변수 속성 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setschedparam(pthread_attr_t *attr,
                               const struct sched_param *param);
int pthread_attr_getschedparam(const pthread_attr_t *attr,
                               struct sched_param *param);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_setschedparam()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스케줄링 매개변수 속성들을 `param`이 가리키는 버퍼에 지정한 값들로 설정한다. 이 속성들이 스레드 속성 객체 `attr`을 이용해 생성하는 스레드의 스케줄링 매개변수들을 결정한다.

`pthread_attr_getschedparam()`은 스레드 속성 객체 `attr`의 스케줄링 매개변수 속성들을 `param`이 가리키는 버퍼로 반환한다.

다음 구조체에 스케줄링 매개변수들을 담는다.

```c
struct sched_param {
    int sched_priority;     /* 스케줄링 우선순위 */
};
```

보다시피 한 가지 스케줄링 매개변수만 지원한다. 각 스케줄링 정책에서 허용하는 스케줄링 우선순위 범위에 대한 세부 내용은 <tt>[[sched(7)]]</tt>를 보라.

`pthread_attr_setschedparam()`으로 설정한 매개변수가 <tt>[[pthread_create(3)]]</tt> 호출 때 효과가 있으려면 호출자가 <tt>[[pthread_attr_setinheritsched(3)]]</tt>를 사용해 속성 객체 `attr`의 스케줄러 상속 속성을 `PTHREAD_EXPLICIT_SCHED`로 설정해야 한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_attr_setschedparam()`이 다음 오류로 실패할 수 있다.

`EINVAL`
:   `attr`의 현재 스케줄링 정책에서 `param`에 지정한 우선순위가 말이 되지 않음.

POSIX.1에서는 `pthread_attr_setschedparam()`에서 `ENOTSUP` 오류도 적고 있다. 리눅스에서는 절대 이 값을 반환하지 않는다. (그렇기는 하지만 이식 가능하고 미래를 대비하는 응용에서는 이 오류 반환 값을 처리해야 할 것이다.)

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setschedparam()`,<br>`pthread_attr_getschedparam()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

리눅스에서 지원하는 스레드 스케줄링 정책들의 목록은 <tt>[[pthread_attr_setschedpolicy(3)]]</tt>을 보라.

## EXAMPLE

<tt>[[pthread_setschedparam(3)]]</tt> 참고.

## SEE ALSO

<tt>[[sched_get_priority_min(2)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setinheritsched(3)]]</tt>, <tt>[[pthread_attr_setschedpolicy(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_setschedparam(3)]]</tt>, <tt>[[pthread_setschedprio(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2017-09-15
