## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_mutex_timedlock - 뮤텍스 잠그기

## SYNOPSIS

```c
#include <pthread.h>
#include <time.h>

int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex,
    const struct timespec *restrict abstime);
```

## DESCRIPTION

`pthread_mutex_timedlock()` 함수는 `mutex`가 가리키는 뮤텍스 객체를 잠근다. 뮤텍스가 이미 잠겨 있으면 `pthread_mutex_lock()` 함수에서처럼 뮤텍스가 사용 가능해질 때까지 호출 스레드가 블록 한다. 다른 스레드가 뮤텍스를 푸는 걸 기다리지 않고는 뮤텍스를 잠글 수 없는 경우에 지정한 타임아웃 만료 시 그 대기를 끝낸다.

타임아웃 기준 클럭으로 측정해서 `abstime`으로 지정한 절대 시간이 지날 때 (즉 그 클럭의 값이 `abstime`과 같거나 그보다 커질 때), 또는 `abstime`으로 지정한 절대 시간이 호출 시점에 이미 지나 있는 경우에 타임아웃이 만료된다.

타임아웃은 `CLOCK_REALTIME` 클럭을 기준으로 한다. 타임아웃의 해상도는 기준 클럭의 해상도이다. `<time.h>` 헤더에 `timespec` 데이터 타입이 정의되어 있다.

뮤텍스를 즉시 잠글 수 있으면 어떤 경우에도 타임아웃 때문에 함수가 실패하지 않는다. 뮤텍스를 즉시 잠글 수 있다면 `abstime` 매개변수의 유효성을 검사할 필요가 없다.

(`PRIO_INHERIT` 프로토콜로 초기화 한 뮤텍스에서) 우선순위 상속 규칙에 따라서, 시간 지정 뮤텍스 대기가 타임아웃 만료로 끝나는 경우에 뮤텍스 소유자가 더는 그 뮤텍스를 기다리는 스레드들 중 하나가 아니라는 점을 반영하여 그 스레드의 우선순위를 적절히 조정한다.

`mutex`가 견고 뮤텍스이고 뮤텍스 락을 잡고 있는 동안 소유 스레드가 포함된 프로세스가 종료했으면 `pthread_mutex_timedlock()` 호출이 오류 값 `[EOWNERDEAD]`를 반환한다. `mutex`가 견고 뮤텍스이고 뮤텍스 락을 잡고 있는 동안 소유 스레드가 종료했으면 소유 스레드가 속한 프로세스가 종료하지 않았더라도 `pthread_mutex_timedlock()`이 오류 값 `[EOWNERDEAD]`를 반환할 수 있다. 이 경우 그 스레드에 의해 뮤텍스가 잠기지만 뮤텍스가 보호하는 상태가 비일관이라고 표시된다. 재사용을 위해선 응용에서 그 상태를 정상으로 만들어야 하며, 완료됐을 때 `pthread_mutex_consistent()`를 호출하면 된다. 응용에서 상태를 복구할 수 없으면 `pthread_mutex_consistent()` 호출 없이 뮤텍스를 풀어야 하며, 그러면 그 뮤텍스는 영구히 사용 불가능한 것으로 표시된다.

`mutex`가 초기화 된 뮤텍스 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_mutex_timedlock()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_mutex_timedlock()` 함수가 실패한다.

`EAGAIN`
:   `mutex`의 재귀 락 최대 횟수를 초과했기 때문에 뮤텍스를 획득할 수 없다.

`EDEADLK`
:   뮤텍스 유형이 `PTHREAD_MUTEX_ERRORCHECK`이며 현재 스레드가 이미 그 뮤텍스를 소유하고 있다.

`EINVAL`
:   프로토콜 속성 값을 `PTHREAD_PRIO_PROTECT`로 해서 `mutex`를 생성했으며 호출 스레드의 우선순위가 뮤텍스의 현재 우선순위 상한보다 높다.

`EINVAL`
:   프로세스 내지 스레드가 블록 하려 했는데 `abstime` 매개변수의 나노초 필드가 0보다 작거나 10억 이상인 값으로 지정되어 있다.

`ENOTRECOVERABLE`
:   뮤텍스가 보호하는 상태가 복구 가능하지 않다.

`EOWNERDEAD`
:   뮤텍스가 견고 뮤텍스이며 이전 소유자 스레드를 포함한 프로세스가 뮤텍스 락을 잡은 채로 종료했다. 호출 스레드가 뮤텍스 락을 획득하게 되며 상태를 정상으로 만드는 것은 새 소유자의 몫이다.

`ETIMEDOUT`
:   지정한 타임아웃 만료 전에 뮤텍스를 잠글 수 없었다.

다음 경우에 `pthread_mutex_timedlock()` 함수가 실패할 수도 있다.

`EDEADLK`
:   교착 조건을 탐지했다.

`EOWNERDEAD`
:   뮤텍스가 견고 뮤텍스이며 이전 소유자 스레드가 뮤텍스 락을 잡은 채로 종료했다. 호출 스레드가 뮤텍스 락을 획득하게 되며 상태를 정상으로 만드는 것은 새 소유자의 몫이다.

이 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

0 아닌 반환 값이 오류라고 가정했던 응용에서 견고 뮤텍스를 쓰려면 변경이 필요할 것이다. 현재 비일관인 상태를 보호하는 뮤텍스를 스레드가 얻을 때 `[EOWNERDEAD]`가 유효 반환이기 때문이다. 그런 오류가 발생할 가능성을 배제하기에 오류 반환을 검사하지 않은 응용에서는 견고 뮤텍스를 사용하지 말아야 한다. 응용에서 일반 뮤텍스와 견고 뮤텍스 모두를 사용할 수 있어야 한다면 모든 반환 값의 오류 조건을 검사하고 필요시 적절한 동작을 취해야 한다.

## RATIONALE

`pthread_mutex_lock()` 참고.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_mutex_destroy()]]</tt>, <tt>[[pthread_mutex_lock()]]</tt>, `time()`

POSIX.1-2008 Base Definitions 권, *4.11절 Memory Synchronization*, `<pthread.h>`, `<time.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
