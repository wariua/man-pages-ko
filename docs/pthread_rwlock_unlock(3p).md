## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_rwlock_unlock - 읽기-쓰기 락 객체 풀기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

## DESCRIPTION

`pthread_rwlock_unlock()` 함수는 `rwlock`이 가리키는 읽기-쓰기 락 객체에 잡은 락을 놓는다. 읽기-쓰기 락 `rwlock`을 호출 스레드가 잡고 있지 않은 경우의 결과는 규정되어 있지 않다.

읽기-쓰기 락에서 읽기 락을 놓기 위해 이 함수를 호출하는 것이고 현재 그 읽기-쓰기 락 객체에 누군가 잡고 있는 다른 읽기 락이 있는 경우에는 읽기-쓰기 락 객체가 계속 읽기 잠김 상태로 남는다. 이 함수가 그 읽기-쓰기 락 객체의 마지막 읽기 락을 놓는 경우에는 읽기-쓰기 락 객체가 소유자 없이 풀린 상태가 된다.

읽기-쓰기 락 객체의 쓰기 락을 놓기 위해 이 함수를 호출하는 경우에는 읽기-쓰기 락 객체가 풀린 상태가 된다.

락이 사용 가능해질 때 그 락에 블록 된 스레드들이 있으면 스케줄링 정책에 따라 그 락을 획득할 스레드(들)이 정해진다. Thread Execution Scheduling 옵션을 지원하는 경우, 스케줄링 정책 `SCHED_FIFO`, `SCHED_RR`, `SCHED_SPORADIC`으로 실행 중인 스레드들이 락에 대기 중이면 락이 사용 가능해질 때 스레드들이 우선순위 순서로 락을 획득한다. 같은 우선순위의 스레드에서는 쓰기 락이 읽기 락보다 우선한다. Thread Execution Scheduling 옵션을 지원하지 않는 경우, 쓰기 락이 읽기 락보다 우선하는지 여부는 구현에서 규정한다.

초기화 안 된 읽기-쓰기 락으로 이 함수를 호출하는 경우의 결과는 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_rwlock_unlock()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

`pthread_rwlock_unlock()` 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

`pthread_rwlock_unlock()`의 `rwlock` 인자로 지정한 값이 초기화 된 읽기-쓰기 락 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_rwlock_unlock()`의 `rwlock` 인자로 지정한 값이 현재 스레드가 잡고 있지 않은 읽기-쓰기 락 객체를 가리키고 있음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EPERM]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_rwlock_destroy()|pthread_rwlock_destroy(3p)]]</tt>, <tt>[[pthread_rwlock_rdlock()|pthread_rwlock_rdlock(3p)]]</tt>, <tt>[[pthread_rwlock_timedrdlock()|pthread_rwlock_timedrdlock(3p)]]</tt>, <tt>[[pthread_rwlock_timedwrlock()|pthread_rwlock_timedwrlock(3p)]]</tt>, <tt>[[pthread_rwlock_trywrlock()|pthread_rwlock_trywrlock(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, *4.11절 Memory Synchronization*, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
