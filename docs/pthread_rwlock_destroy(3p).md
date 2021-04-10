## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_rwlock_destroy, pthread_rwlock_init - 읽기-쓰기 락 객체 파기 및 초기화

## SYNOPSIS

```c
#include <pthread.h>

int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
    const pthread_rwlockattr_t *restrict attr);
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;
```

## DESCRIPTION

`pthread_rwlock_destroy()` 함수는 `rwlock`이 가리키는 읽기-쓰기 락 객체를 파기하고 그 락에서 쓰는 모든 자원을 해제한다. 락을 또 다른 `pthread_rwlock_init()` 호출로 다시 초기화 하기 전까지는 이후 그 락 사용의 결과가 규정되어 있지 않다. 구현 시 `pthread_rwlock_destroy()`에서 `rwlock`이 가리키는 객체를 비유효 값으로 설정하도록 할 수도 있다. `rwlock`을 잡고 있는 스레드가 있을 때 `pthread_rwlock_destroy()`를 호출하는 경우의 결과는 규정되어 있지 않다. 초기화 안 된 읽기-쓰기 락을 파기하려 하는 결과는 규정되어 있지 않다.

`pthread_rwlock_init()` 함수는 `rwlock`이 가리키는 읽기-쓰기 락을 사용하기 위해 필요한 자원이 있으면 할당하고서 그 락을 `attr`이 가리키는 속성들로 풀린 상태로 초기화 한다. `attr`이 NULL이면 기본 읽기-쓰기 락 속성들을 사용한다. 즉 기본 읽기-쓰기 락 속성 객체의 주소를 전달하는 것과 효과가 같다. 한 번 초기화 하고 나면 재초기화 없이 몇 번이든 락을 쓸 수 있다. 이미 초기화 된 읽기-쓰기 락을 지정해서 `pthread_rwlock_init()`을 호출하는 경우의 결과는 규정되어 있지 않다. 읽기 쓰기 락을 먼저 초기화 하지 않고 쓰는 경우의 결과는 규정되어 있지 않다.

`pthread_rwlock_init()` 함수가 실패하면 `rwlock`이 초기화 되지 않으며 `rwlock`의 내용이 규정되어 있지 않다.

`rwlock`이 가리키는 객체만을 동기화 수행에 사용할 수 있다. 그 객체의 사본을 `pthread_rwlock_destroy()`, `pthread_rwlock_rdlock()`, `pthread_rwlock_timedrdlock()`, `pthread_rwlock_timedwrlock()`, `pthread_rwlock_tryrdlock()`, `pthread_rwlock_trywrlock()`, `pthread_rwlock_unlock()`, `pthread_rwlock_wrlock()` 호출에서 참조할 때의 결과는 규정되어 있지 않다.

기본 읽기-쓰기 락 속성들이 적합한 경우에는 매크로 `PTHREAD_RWLOCK_INITIALIZER`를 써서 읽기-쓰기 락을 초기화 할 수 있다. 매개변수 `attr`을 NULL로 지정해 `pthread_rwlock_init()`을 호출하는 동적 초기화와 효과가 동등하되 오류 검사를 수행하지 않는다는 점이 다르다.

`pthread_rwlock_init()`의 `attr` 인자로 지정한 값이 초기화 된 읽기-쓰기 락 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_rwlock_destroy()`와 `pthread_rwlock_init()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_rwlock_init()` 함수가 실패한다.

`EAGAIN`
:   읽기-쓰기 락을 새로 초기화 하는 데 필요한 (메모리 외의) 자원이 시스템에 부족하다.

`ENOMEM`
:   읽기-쓰기 락을 초기화 하기에 충분한 메모리가 없다.

`EPERM`
:   호출자에게 동작을 수행하기 위한 특권이 없다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

이 함수들과 관련 읽기-쓰기 락 함수들을 쓰는 응용에서 POSIX.1-2008 Base Definitions 권의 *3.287절 Priority Inversion*에서 논의하는 우선순위 역전을 겪을 수 있다.

## RATIONALE

`pthread_rwlock_destroy()`의 `rwlock` 인자로 지정한 값이 초기화 된 읽기-쓰기 락 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_rwlock_int()`의 `attr` 인자로 지정한 값이 초기화 된 읽기-쓰기 락 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_rwlock_destroy()`나 `pthread_rwlock_init()`의 `rwlock` 인자로 지정한 값이 잠겨 있는 읽기-쓰기 락 객체를 참고하고 있음을, 또는 `pthread_rwlock_init()`의 `rwlock` 인자로 지정한 값이 이미 초기화 된 읽기-쓰기 락 객체를 가리키고 있음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EBUSY]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_rwlock_rdlock()|pthread_rwlock_rdlock(3p)]]</tt>, <tt>[[pthread_rwlock_timedrdlock()|pthread_rwlock_timedrdlock(3p)]]</tt>, <tt>[[pthread_rwlock_timedwrlock()|pthread_rwlock_timedwrlock(3p)]]</tt>, <tt>[[pthread_rwlock_trywrlock()|pthread_rwlock_trywrlock(3p)]]</tt>, <tt>[[pthread_rwlock_unlock()|pthread_rwlock_unlock(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, *3.287절 Priority Inversion*, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
