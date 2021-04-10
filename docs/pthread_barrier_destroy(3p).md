## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_barrier_destroy, pthread_barrier_init - 배리어 객체 파기 및 초기화

## SYNOPSIS

```c
#include <pthread.h>

int pthread_barrier_destroy(pthread_barrier_t *barrier);
int pthread_barrier_init(pthread_barrier_barrier_t *restrict barrier,
    const pthread_barrierattr_t *restrict attr, unsigned count);
```

## DESCRIPTION

`pthread_barrier_destroy()` 함수는 `barrier`가 가리키는 배리어를 파기하고 그 배리어에서 쓰는 모든 자원을 해제한다. 배리어를 또 다른 `pthread_barrier_init()` 호출로 다시 초기화 하기 전까지는 이후 그 배리어 사용의 결과가 규정되어 있지 않다. 구현 시 이 함수에서 `barrier`를 비유효 값으로 설정하도록 할 수도 있다. 배리어에 블록 하고 있는 스레드가 있을 때 `pthread_barrier_destroy()`를 호출하거나 초기화 안 된 배리어로 이 함수를 호출하는 경우의 결과는 규정되어 있지 않다.

`pthread_barrier_init()` 함수는 `barrier`가 가리키는 배리어를 사용하기 위해 필요한 자원이 있으면 할당하고서 그 배리어를 `attr`이 가리키는 속성들로 초기화 한다. `attr`이 NULL이면 기본 배리어 속성들을 사용한다. 즉 기본 배리어 속성 객체의 주소를 전달하는 것과 효과가 같다. 배리어에 블록 하고 있는 (즉 `pthread_barrier_wait()` 호출에서 반환하지 않은) 함수가 있을 때 `pthread_barrier_init()`을 호출하는 경우의 결과는 규정되어 있지 않다. 배리어를 먼저 초기화 하지 않고 쓰는 경우의 결과는 규정되어 있지 않다. 이미 초기화 된 배리어를 지정해서 `pthread_barrier_init()`을 호출하는 경우의 결과는 규정되어 있지 않다.

`count` 인자는 `pthread_barrier_wait()` 호출이 성공 반환하기 위해 그 함수를 호출해야 하는 스레드의 수를 지정한다. `count`로 지정하는 값이 0보다 커야 한다.

`pthread_barrier_init()` 함수가 실패하면 배리어가 초기화 되지 않으며 `barrier`의 내용이 규정되어 있지 않다.

`barrier`가 가리키는 객체만을 동기화 수행에 사용할 수 있다. 그 객체의 사본을 `pthread_barrier_destroy()`나 `pthread_barrier_wait()` 호출에서 참조할 때의 결과는 규정되어 있지 않다.

## RETURN VALUE

성공 완료 시 이 함수들은 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_barrier_init()` 함수가 실패한다.

`EAGAIN`
:   배리어를 새로 초기화 하는 데 필요한 자원이 시스템에 부족하다.

`EINVAL`
:   `count`로 지정한 값이 0과 같다.

`ENOMEM`
:   배리어를 초기화 하기에 충분한 메모리가 없다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

`pthread_barrier_destroy()`의 `barrier` 인자로 지정한 값이 초기화 된 배리어 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_barrier_init()`의 `attr` 인자로 지정한 값이 초기화 된 배리어 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_barrier_destroy()`나 `pthread_barrier_init()`의 `barrier` 인자로 지정한 값을 다른 스레드가 사용 중임을 (가령 `pthread_barrier_wait()` 호출 내에 있음을), 또는 `pthread_barrier_init()`의 `barrier` 인자로 지정한 값이 이미 초기화 된 배리어 객체를 가리키고 있음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EBUSY]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_barrier_wait()|pthread_barrier_wait(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
