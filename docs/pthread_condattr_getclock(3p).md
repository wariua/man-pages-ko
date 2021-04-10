## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_condattr_getclock, pthread_condattr_setclock - 클럭 선택 조건 변수 속성 얻기 및 설정하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_condattr_getclock(const pthread_condattr_t *restrict attr,
    clockid_t *restrict clock_id);
int pthread_condattr_setclock(pthread_condattr_t *attr,
    clockid_t clock_id);
```

## DESCRIPTION

`pthread_condattr_getclock()` 함수는 `attr`이 가리키는 속성 객체로부터 `clock` 속성의 값을 얻는다.

`pthread_condattr_setclock()` 함수는 `attr`이 가리키는 초기화 된 속성 객체의 `clock` 속성을 설정한다. CPU 시간 클럭을 가리키는 `clock_id` 인자로 `pthread_condattr_setclock()`을 호출하면 호출이 실패한다.

`clock` 속성은 `pthread_cond_timedwait()`의 타임아웃 기능 측정에 쓰이는 클럭의 클럭 ID이다. `clock` 속성의 기본값은 시스템 클럭을 가리킨다.

`pthread_condattr_getclock()`이나 `pthread_condattr_setclock()`의 `attr` 인자로 지정한 값이 초기화 된 조건 변수 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_condattr_getclock()` 함수는 0을 반환하며 `attr`의 클럭 속성 값을 `clock_id` 인자가 가리키는 객체에 저장한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

성공 시 `pthread_condattr_setclock()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_condattr_setclock()` 함수가 실패할 수도 있다.

`EINVAL`
:   `clock_id`로 지정한 값이 알려진 클럭을 가리키고 있지 않거나 CPU 시간 클럭이다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

`pthread_condattr_getclock()`이나 `pthread_condattr_setclock()`의 `attr` 인자로 지정한 값이 초기화 된 조건 변수 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_cond_destroy()|pthread_cond_destroy(3p)]]</tt>, <tt>[[pthread_cond_timedwait()|pthread_cond_timedwait(3p)]]</tt>, <tt>[[pthread_condattr_destroy()|pthread_condattr_destroy(3p)]]</tt>, <tt>[[pthread_condattr_getpshared()|pthread_condattr_getpshared(3p)]]</tt>, <tt>[[pthread_create()|pthread_create(3)]]</tt>, <tt>[[pthread_mutex_destroy()|pthread_mutex_destroy(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
