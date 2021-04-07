## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_mutexattr_getprioceiling, pthread_mutexattr_setprioceiling - 뮤텍스 속성 객체의 prioceiling 속성 얻기 및 설정하기 (<strong>실시간 스레드</strong>)

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutexattr_getprioceiling(const pthread_mutexattr_t
    *restrict attr, int *restrict prioceiling);
int pthread_mutexattr_setprioceiling(pthread_mutexattr_t *attr,
    int prioceiling);
```

## DESCRIPTION

`pthread_mutexattr_getprioceiling()` 및 `pthread_mutexattr_setprioceiling()` 함수는 앞서 `pthread_mutexattr_init()` 함수로 생성한 `attr`이 가리키는 뮤텍스 속성 객체의 우선순위 상한 속성을 얻고 설정한다.

`prioceiling` 속성은 초기화 되는 뮤텍스의 우선순위 상한값을 담는다. `prioceiling` 값은 `SCHED_FIFO`에서 규정하는 최대 우선순위 범위 내에 있다.

`prioceiling` 속성은 초기화 되는 뮤텍스의 우선순위 상한을 규정하는데, 이는 그 뮤텍스가 보호하는 임계 구역이 실행되는 최소 수준 우선순위이다. 우선순위 역전을 피하기 위해선 뮤텍스의 우선순위 상한을 그 뮤텍스를 잠글 수도 있는 스레드 전체의 최고 우선순위보다 높거나 같게 설정해야 한다. `prioceiling` 값은 `SCHED_FIFO` 스케줄링 정책 하에 정의된 최대 우선순위 범위 내에 있다.

`pthread_mutexattr_getprioceiling()` 내지 `pthread_mutexattr_setprioceiling()`의 `attr` 인자로 지정한 값이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 완료 시 `pthread_mutexattr_getprioceiling()` 및 `pthread_mutexattr_setprioceiling()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 이 함수들이 실패할 수도 있다.

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>prioceiling</code>으로 지정한 값이 유효하지 않다.</dd>
<dt><code>EPERM</code></dt>
<dd>호출자가 동작을 수행하기 위한 특권을 가지고 있지 않다.</dd>
</dl>

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

`pthread_mutexattr_getprioceiling()` 내지 `pthread_mutexattr_setprioceiling()`의 `attr` 인자로 지정한 값이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_cond_destroy()|pthread_cond_destroy(3p)]]</tt>, <tt>[[pthread_create()|pthread_create(3)]]</tt>, <tt>[[pthread_mutex_destroy()|pthread_mutex_destroy(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
