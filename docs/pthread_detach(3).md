## NAME

pthread_detach - 스레드 분리하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_detach(pthread_t thread);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_detach()` 함수는 `thread`가 나타내는 스레드를 분리된 것으로 표시한다. 분리된 스레드가 종료하면 자원이 자동으로 시스템으로 해제된다. 즉 다른 스레드가 그 종료 스레드와 합류할 필요가 없다.

이미 분리된 스레드를 분리하려는 시도는 명세되지 않은 동작을 유발한다.

## RETURN VALUE

성공 시 `pthread_detach()`는 0을 반환한다. 오류 시 오류 번호를 반환한다.

## ERRORS

`EINVAL`
:   `thread`가 합류 가능 스레드가 아니다.

`ESRCH`
:   ID가 `thread`인 스레드를 찾을 수 없다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_detach()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

스레드가 일단 분리되고 나면 <tt>[[pthread_join(3)]]</tt>으로 합류하거나 다시 합류 가능하게 만들 수 없다.

<tt>[[pthread_attr_setdetachstate(3)]]</tt>를 이용해 <tt>[[pthread_create(3)]]</tt>의 `attr` 인자의 분리 속성을 설정하면 새 스레드를 분리 상태로 생성할 수 있다.

분리 속성은 스레드가 종료할 때 시스템을 동작 방식을 결정할 뿐이다. 프로세스가 <tt>[[exit(3)]]</tt>으로 종료하는 경우에 (또는 그와 동등하게 메인 스레드가 반환하는 경우) 스레드가 종료되지 않게 해 주지 않는다.

응용에서 생성하는 각 스레드에 <tt>[[pthread_join(3)]]</tt>과 `pthread_detach()` 중 하나를 호출해서 그 스레드의 시스템 자원이 해제될 수 있게 해야 한다. (하지만 둘 중 한 동작을 하지 않은 스레드가 있더라도 프로세스가 종료할 때 그 자원이 해제된다.)

## EXAMPLE

다음 문은 호출 스레드를 분리한다.

```c
pthread_detach(pthread_self());
```

## SEE ALSO

<tt>[[pthread_attr_setdetachstate(3)]]</tt>, <tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_exit(3)]]</tt>, <tt>[[pthread_join(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
