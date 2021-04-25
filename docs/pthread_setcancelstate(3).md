## NAME

pthread_setcancelstate, pthread_setcanceltype - 취소 가능성 상태와 유형 설정하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_setcancelstate(int state, int *oldstate);
int pthread_setcanceltype(int type, int *oldtype);
```

`-pthread`로 링크.

## DESCRIPTION

`pthread_setcancelstate()`는 호출 스레드의 취소 가능성 상태를 `state`에 준 값으로 설정한다. 스레드의 이전 취소 가능성 상태를 `oldstate`가 가리키는 버퍼로 반환한다. `state` 인자는 다음 값들 중 하나여야 한다.

`PTHREAD_CANCEL_ENABLE`
:   스레드가 취소 가능하다. 최초 스레드를 포함한 모든 새 스레드의 기본 취소 가능성 상태이다. 스레드의 취소 가능성 유형이 최소 가능 스레드가 취소 요청에 언제 반응하게 될지 결정한다.

`PTHREAD_CANCEL_DISABLE`
:   스레드가 취소 가능하지 않다. 취소 요청을 수신하면 취소 가능성이 활성화될 때까지 막아 둔다.

`pthread_setcanceltype()`은 호출 스레드의 취소 가능성 유형을 `type`에 준 값으로 설정한다. 스레드의 이전 취소 가능성 유형을 `oldtype`이 가리키는 버퍼로 반환한다. `type` 인자는 다음 값들 중 하나여야 한다.

`PTHREAD_CANCEL_DEFERRED`
:   스레드가 취소점(<tt>[[pthreads(7)]]</tt> 참고)인 함수를 호출할 때까지 취소 요청을 연기한다. 최초 스레드를 포함한 모든 새 스레드의 기본 취소 가능성 유형이다.

    취소가 연기된 경우에도 비동기 시그널 핸들러 내의 취소점이 작용할 수 있으며, 그 효과는 비동기 취소가 일어난 것과 같다.

`PTHREAD_CANCEL_ASYNCHRONOUS`
:   스레드가 언제든 취소될 수 있다. (보통은 취소 요청 수신 즉시 취소되지만 시스템에서 이를 보장하지는 않는다.)

이 함수들 각각이 수행하는 설정하기-얻기 동작은 같은 함수를 호출하는 프로세스 내 다른 스레드에 대해 원자적이다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_setcancelstate()`가 다음 오류로 실패할 수 있다.

`EINVAL`
:   `state`에 유효하지 않은 값.

`pthread_setcanceltype()`이 다음 오류로 실패할 수 있다.

`EINVAL`
:   `type`에 유효하지 않은 값.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_setcancelstate()`,<br>`pthread_setcanceltype()` | 스레드 안전성 | MT-Safe |
| `pthread_setcancelstate()`,<br>`pthread_setcanceltype()` | 비동기 취소 안전성 | AC-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

스레드가 취소될 때 어떤 일이 일어나는지에 대한 자세한 내용은 <tt>[[pthread_cancel(3)]]</tt>을 보라.

취소 요청으로 중단돼선 안 되는 어떤 중요한 동작을 스레드에서 수행할 때 취소 가능성을 잠시 꺼 두는 게 유용하다. 긴 기간 동안, 또는 오래 블록 할 수 있는 동작에서 취소 가능성을  끄지 않도록 조심해야 한다. 스레드가 취소 요청에 묵묵부답이 된다.

### 비동기 취소 가능성

취소 가능성 유형을 `PTHREAD_CANCEL_ASYNCHRONOUS`로 설정하는 게 유용한 경우는 잘 없다. 스레드가 *언제든* 취소될 수 있으므로 안전하게 자원을 예약(가령 <tt>[[malloc(3)]]</tt>으로 메모리 할당)하거나 뮤텍스나 세마포어, 락 등을 획득할 수 없다. 자원 예약이 안전하지 않은 이유는 스레드가 취소될 때 그 자원들의 상태가 어떤지를 (즉 자원 예약 전에 취소가 일어났는지, 아니면 예약한 상태에서, 또는 해제한 후에 일어났는지를) 응용에서 알 방법이 없기 때문이다. 또한 함수 호출 중에 취소가 발생하는 경우 어떤 내부 자료 구조(가령 <tt>[[malloc(3)]]</tt> 계열 함수들에서 관리하는 유휴 블록 연결 리스트)가 비일관 상태로 남을 수도 있다. 그 결과 정리 핸들러가 더는 쓸모가 없게 된다.

안전하게 비동기적으로 취소할 수 있는 함수를 *비동기 취소 안전 함수*라고 부른다. POSIX.1-2001과 POSIX.1-2008에서는 <tt>[[pthread_cancel(3)]]</tt>, `pthread_setcancelstate()`, `pthread_setcanceltype()`이 비동기 취소 안전이기만을 요구한다. 일반적으로 비동기적으로 취소될 수 있는 스레드에서 다른 라이브러리 함수들을 안전하게 호출할 수 없다.

비동기 취소 가능성이 쓸모 있는 몇 안 되는 경우 중 하나는 순수한 연산 위주 루프 내에 있는 스레드를 취소할 때이다.

### 이식성 관련 사항

리눅스 스레딩 구현에서는 `pthread_setcancelstate()`의 `oldstate` 인자가 NULL인 것을 허용하며, 그 경우 이전 취소 가능성 상태에 대한 정보가 호출자에게 반환되지 않는다. 다른 여러 구현에서도 NULL인 `oldstate` 인자를 허용한다. 하지만 POSIX.1에서는 이 점을 명세하고 있지 않으므로 이식 가능한 응용에서는 항상 `oldstate`에 NULL 아닌 값을 지정하는 게 좋다. 정확히 같은 내용이 `pthread_setcanceltype()`의 `oldtype` 인자에도 적용된다.

## EXAMPLES

<tt>[[pthread_cancel(3)]]</tt> 참고.

## SEE ALSO

<tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_cleanup_push(3)]]</tt>, <tt>[[pthread_testcancel(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
