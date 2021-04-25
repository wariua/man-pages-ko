## NAME

pthread_join - 종료한 스레드와 합류하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_join(pthread_t thread, void **retval);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_join()` 함수는 `thread`로 지정한 스레드가 종료하기를 기다린다. 스레드가 이미 종료했으면 `pthread_join()`이 즉시 반환한다. `thread`로 지정한 스레드가 합류 가능해야 한다.

`retval`이 NULL이 아니면 `pthread_join()`은 대상 스레드의 종료 상태(즉 대상 스레드가 <tt>[[pthread_exit(3)]]</tt>에 준 값)를 `retval`이 가리키는 위치로 복사한다. 대상 스레드가 취소됐으면 `retval`이 가리키는 위치에 `PTHREAD_CANCELED`가 들어간다.

여러 스레드가 동시에 동일 스레드와 합류하려고 하는 경우의 결과는 규정되어 있지 않다. `pthread_join()`을 호출 중인 스레드가 취소되는 경우에는 대상 스레드가 계속 합류 가능 상태로 남는다. (즉 분리되지 않는다.)

## RETURN VALUE

성공 시 `pthread_join()`은 0을 반환한다. 오류 시 오류 번호를 반환한다.

## ERRORS

`EDEADLK`
:   교착을 탐지했다. (가령 두 스레드가 서로와 합류를 시도했다.) 또는 `thread`가 호출 스레드를 나타낸다.

`EINVAL`
:   `thread`가 합류 가능 스레드가 아니다.

`EINVAL`
:   다른 스레드가 이미 그 스레드와 합류하려고 기다리는 중이다.

`ESRCH`
:   ID가 `thread`인 스레드를 찾을 수 없다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_join()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`pthread_join()` 성공 호출 후에 호출자에게는 대상 스레드가 종료했다는 것이 보장된다. 그러면 호출자에서 스레드 종료 후에 필요한 어떤 정리 작업을 (가령 대상 스레드에게 할당했던 메모리나 기타 자원 해제를) 하기로 할 수 있다.

이미 합류한 스레드와 합류하는 결과는 규정되어 있지 않다.

합류 가능한 (즉 분리되지 않은) 스레드와 합류하지 않으면 "좀비 스레드"가 생긴다. 이를 피해야 한다. 각 좀비 스레드는 얼마간 시스템 자원을 소모하며, 좀비 스레드가 많이 쌓이면 새 스레드를 (또는 프로세스를) 생성하는 게 불가능해진다.

`waitpid(-1, &status, 0)`의 pthreads 등가물, 즉 "종료한 아무 스레드와 합류하기"는 없다. 이런 기능이 필요하다고 생각한다면 아마 응용 설계를 다시 생각해 봐야 할 것이다.

프로세스 내의 모든 스레드는 동료이다. 즉 어느 스레드든 프로세스 내 다른 모든 스레드와 합류할 수 있다.

## EXAMPLES

<tt>[[pthread_create(3)]]</tt> 참고.

## SEE ALSO

<tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_detach(3)]]</tt>, <tt>[[pthread_exit(3)]]</tt>, <tt>[[pthread_tryjoin_np(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
