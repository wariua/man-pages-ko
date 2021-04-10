## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_rwlock_rdlock, pthread_rwlock_tryrdlock - 읽기-쓰기 락 객체를 읽기용으로 잠그기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
```

## DESCRIPTION

`pthread_rwlock_rdlock()` 함수는 `rwlock`이 가리키는 읽기-쓰기 락에 읽기 락을 적용한다. 쓰기 쪽이 락을 잡고 있지 않고 락에 블록 된 쓰기 쪽이 없으면 호출 스레드가 읽기 락을 획득한다.

Thread Execution Scheduling 옵션을 지원하며 락에 관련된 스레드들이 스케줄링 정책 `SCHED_FIFO`나 `SCHED_RR`로 실행 중인 경우, 쓰기 쪽이 락을 잡고 있거나 더 높거나 같은 우선순위의 쓰기 쪽이 락에 블록 되어 있으면 호출 스레드가 락을 획득하지 않는다. 아니면 호출 스레드가 락을 획득한다.

Thread Execution Scheduling 옵션을 지원하며 락에 관련된 스레드들이 스케줄링 정책 `SCHED_SPORADIC`으로 실행 중인 경우, 쓰기 쪽이 락을 잡고 있거나 더 높거나 같은 우선순위의 쓰기 쪽이 락에 블록 되어 있으면 호출 스레드가 락을 획득하지 않는다. 아니면 호출 스레드가 락을 획득한다.

Thread Execution Scheduling 옵션을 지원하지 않는 경우, 쓰기 쪽이 락을 잡고 있지 않고 락에 블록 된 쓰기 쪽이 있을 때 호출 스레드가 락을 획득하는지 여부는 구현에서 규정한다. 쓰기 쪽이 락을 잡고 있으면 호출 스레드가 읽기 락을 획득하지 않는다. 읽기 락을 획득하지 않는 경우 호출 스레드는 락을 획득할 수 있게 될 때까지 블록 한다. 호출이 이뤄지는 시점에 호출 스레드가 쓰기 락을 잡고 있다면 교착이 일어날 수도 있다.

한 스레드에서 `rwlock`에 동시에 읽기 락을 여러 개 잡고 있을 수도 (즉 `pthread_rwlock_rdlock()` 함수를 `n`번 성공적으로 호출할 수) 있다. 그런 경우에 응용에서는 그 스레드가 대응하는 풀기를 수행하도록 (즉 `pthread_rwlock_unlock()` 함수를 `n`번 호출하도록) 해야 한다.

구현에서 읽기-쓰기 락에 적용 가능하다고 보장하는 동시 읽기 락 최대 수는 구현에서 규정한다. 그 최댓값을 초과하게 되는 경우 `pthread_rwlock_rdlock()` 함수가 실패할 수도 있다.

`pthread_rwlock_tryrdlock()` 함수는 `pthread_rwlock_rdlock()` 함수처럼 읽기 락을 적용하되 동등한 `pthread_rwlock_rdlock()` 호출이 호출 스레드를 블록 시키게 될 경우에 함수가 실패하게 된다. 어떤 경우에도 `pthread_rwlock_tryrdlock()` 함수는 절대 블록 하지 않는다. 락을 획득하거나 실패하고 즉시 반환한다.

초기화 안 된 읽기-쓰기 락으로 이 함수들을 호출하는 경우의 결과는 규정되어 있지 않다.

읽기-쓰기 락에 읽기용으로 대기 중인 스레드에게 시그널이 전달되는 경우 시그널 핸들러에서 반환한 스레드가 중단된 적 없는 것처럼 다시 읽기-쓰기 락에 읽기용으로 대기한다.

## RETURN VALUE

성공 시 `pthread_rwlock_rdlock()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

`pthread_rwlock_tryrdlock()` 함수는 `rwlock`이 가리키는 읽기-쓰기 락 객체에서 읽기용 락을 획득한 경우 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_rwlock_tryrdlock()` 함수가 실패한다.

`EBUSY`
:   쓰기 쪽이 락을 잡고 있거나 적절한 우선순위의 쓰기 쪽이 락에 블록 되어 있어서 읽기-쓰기 락을 획득할 수 없다.

다음 경우에 `pthread_rwlock_rdlock()` 및 `pthread_rwlock_tryrdlock()` 함수가 실패할 수도 있다.

`EAGAIN`
:   `rwlock`에 대한 읽기 락 최대 횟수를 초과했기 때문에 읽기 락을 획득할 수 없다.

다음 경우에 `pthread_rwlock_rdlock()` 함수가 실패할 수도 있다.

`EDEADLK`
:   교착 조건을 탐지했거나 현재 스레드가 이미 그 읽기-쓰기 락을 쓰기용으로 가지고 있다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

이 함수들을 쓰는 응용에서 POSIX.1-2008 Base Definitions 권의 *3.287절 Priority Inversion*에서 논의하는 우선순위 역전을 겪을 수 있다.

## RATIONALE

`pthread_rwlock_rdlock()`이나 `pthread_rwlock_tryrdlock()`의 `rwlock` 인자로 지정한 값이 초기화 된 읽기-쓰기 락 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_rwlock_destroy()|pthread_rwlock_destroy(3p)]]</tt>, <tt>[[pthread_rwlock_timedrdlock()|pthread_rwlock_timedrdlock(3p)]]</tt>, <tt>[[pthread_rwlock_timedwrlock()|pthread_rwlock_timedwrlock(3p)]]</tt>, <tt>[[pthread_rwlock_trywrlock()|pthread_rwlock_trywrlock(3p)]]</tt>, <tt>[[pthread_rwlock_unlock()|pthread_rwlock_unlock(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, *3.287절 Priority Inversion*, *4.11절 Memory Synchronization*, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
