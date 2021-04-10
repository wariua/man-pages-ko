## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_rwlock_trywrlock, pthread_rwlock_wrlock - 읽기-쓰기 락 객체를 쓰기용으로 잠그기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
```

## DESCRIPTION

`pthread_rwlock_trywrlock()` 함수는 `pthread_rwlock_wrlock()` 함수처럼 쓰기 락을 적용하되 현재 `rwlock`을 (읽기용이나 쓰기용으로) 잡고 있는 스레드가 있으면 함수가 실패한다.

`pthread_rwlock_wrlock()` 함수는 `rwlock`이 가리키는 읽기-쓰기 락에 쓰기 락을 적용한다. 읽기-쓰기 락 `rwlock`을 잡고 있는 다른 (읽기 또는 쓰기) 스레드가 없으면 호출 스레드가 쓰기 락을 획득한다. 호출이 이뤄지는 시점에 호출 스레드가 읽기-쓰기 락을 (읽기 락이든 쓰기 락이든) 잡고 있다면 교착이 일어날 수도 있다.

구현에서 쓰기 쪽 굶주림을 피하기 위해 쓰기 쪽을 읽기 쪽보다 우대할 수도 있다.

초기화 안 된 읽기-쓰기 락으로 이 함수들을 호출하는 경우의 결과는 규정되어 있지 않다.

읽기-쓰기 락에 쓰기용으로 대기 중인 스레드에게 시그널이 전달되는 경우 시그널 핸들러에서 반환한 스레드가 중단된 적 없는 것처럼 다시 읽기-쓰기 락에 쓰기용으로 대기한다.

## RETURN VALUE

`pthread_rwlock_trywrlock()` 함수는 `rwlock`이 가리키는 읽기-쓰기 락 객체에서 쓰기용 락을 획득한 경우 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

성공 시 `pthread_rwlock_wrlock()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_rwlock_trywrlock()` 함수가 실패한다.

`EBUSY`
:   이미 읽기용이나 쓰기용으로 잠겨 있어서 읽기-쓰기 락을 쓰기용으로 획득할 수 없다.

다음 경우에 `pthread_rwlock_wrlock()` 함수가 실패할 수도 있다.

`EDEADLK`
:   교착 조건을 탐지했거나 현재 스레드가 이미 그 읽기-쓰기 락을 쓰기용이나 읽기용으로 가지고 있다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

이 함수들을 쓰는 응용에서 POSIX.1-2008 Base Definitions 권의 *3.287절 Priority Inversion*에서 논의하는 우선순위 역전을 겪을 수 있다.

## RATIONALE

`pthread_rwlock_trywrlock()`이나 `pthread_rwlock_wrlock()`의 `rwlock` 인자로 지정한 값이 초기화 된 읽기-쓰기 락 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_rwlock_destroy()|pthread_rwlock_destroy(3p)]]</tt>, <tt>[[pthread_rwlock_rdlock()|pthread_rwlock_rdlock(3p)]]</tt>, <tt>[[pthread_rwlock_timedrdlock()|pthread_rwlock_timedrdlock(3p)]]</tt>, <tt>[[pthread_rwlock_timedwrlock()|pthread_rwlock_timedwrlock(3p)]]</tt>, <tt>[[pthread_rwlock_unlock()|pthread_rwlock_unlock(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, *3.287절 Priority Inversion*, *4.11절 Memory Synchronization*, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
