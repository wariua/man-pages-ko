## NAME

pthread_attr_setdetachstate, pthread_attr_getdetachstate - 스레드 속성 객체의 분리 상태 속성 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_setdetachstate()` 함수는 `attr`이 가리키는 스레드 속성 객체의 분리 상태 속성을 `detachstate`에 지정한 값으로 설정한다. 분리 상태 속성은 스레드 속성 객체 `attr`을 이용해 생성하는 스레드가 합류 가능이나 분리 상태 중 어느 쪽으로 생성될지 결정한다.

`detachstate`에 다음 값들을 지정할 수 있다.

`PTHREAD_CREATE_DETACHED`
:   `attr`을 이용해 생성하는 스레드가 분리 상태로 생성된다.

`PTHREAD_CREATE_JOINABLE`
:   `attr`을 이용해 생성하는 스레드가 합류 가능 상태로 생성된다.

새로 초기화 된 스레드 속성 객체에서 분리 상태 속성의 기본 설정은 `PTHREAD_CREATE_JOINABLE`이다.

`pthread_attr_getdetachstate()`는 스레드 속성 객체 `attr`의 분리 상태 속성을 `detachstate`가 가리키는 버퍼로 반환한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_attr_setdetachstate()`가 다음 오류로 실패할 수 있다.

`EINVAL`
:   `detachstate`에 유효하지 않은 값을 지정했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setdetachstate()`,<br>`pthread_attr_getdetachstate()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

분리된 스레드와 합류 가능한 스레드에 대한 더 자세한 내용은 <tt>[[pthread_create(3)]]</tt>를 보라.

합류 가능 상태로 생성한 스레드는 결국 <tt>[[pthread_join(3)]]</tt>으로 합류하거나 <tt>[[pthread_detach(3)]]</tt>로 분리해야 한다. <tt>[[pthread_create(3)]]</tt> 참고.

분리 상태로 생성한 스레드의 ID를 이후의 <tt>[[pthread_detach(3)]]</tt> 내지 <tt>[[pthread_join(3)]]</tt> 호출에서 지정하는 것은 오류이다.

## EXAMPLE

<tt>[[pthread_attr_init(3)]] 참고.

## SEE ALSO

<tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_detach(3)]]</tt>, <tt>[[pthread_join(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
