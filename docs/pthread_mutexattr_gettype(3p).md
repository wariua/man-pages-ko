## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_mutexattr_gettype, pthread_mutexattr_settype - 뮤텍스 유형 속성 얻기 및 설정하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutexattr_gettype(const pthread_mutexattr_t *restrict attr,
    int *restrict type);
int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type);
```

## DESCRIPTION

`pthread_mutexattr_gettype()` 및 `pthread_mutexattr_settype()` 함수는 뮤텍스 `type` 속성을 얻고 설정한다. 이 함수들의 `type` 매개변수에 이 속성을 설정한다. `type` 속성의 기본값은 `PTHREAD_MUTEX_DEFAULT`이다.

뮤텍스 속성의 `type` 속성에는 뮤텍스의 유형이 담긴다. 유효한 뮤텍스 유형들에는 다음이 포함된다.

* `PTHREAD_MUTEX_NORMAL`
* `PTHREAD_MUTEX_ERRORCHECK`
* `PTHREAD_MUTEX_RECURSIVE`
* `PTHREAD_MUTEX_DEFAULT`

뮤텍스 유형은 뮤텍스를 잠그고 푸는 호출의 동작 방식에 영향을 준다. 자세한 내용은 `pthread_mutex_lock()`을 보라. 구현에서 `PTHREAD_MUTEX_DEFAULT`를 다른 한 뮤텍스 유형으로 사상할 수도 있다.

`pthread_mutexattr_gettype()` 내지 `pthread_mutexattr_settype()`의 `attr` 인자로 지정한 값이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 완료 시 `pthread_mutexattr_gettype()` 함수는 0을 반환하며 `attr`의 `type` 속성 값을 `type` 매개변수가 가리키는 객체에 저장한다. . 아니면 오류를 나타내는 오류 번호를 반환한다.

성공 시 `pthread_mutexattr_settype()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_mutexattr_settype()` 함수가 실패한다.

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>type</code> 값이 유효하지 않다.</dd>
</dl>

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

없음.

## APPLICATION USAGE

응용에서 `PTHREAD_MUTEX_RECURSIVE` 뮤텍스를 조건 변수와 함께 사용하지 않기를 권한다. `pthread_cond_timedwait()` 내지 `pthread_cond_wait()`을 위해 수행하는 묵시적 풀기에서 (뮤텍스가 여러 번 잠겨 있으면) 실제로는 뮤텍스를 놓지 않을 수도 있기 때문이다. 그렇게 되면 다른 어떤 스레드도 술어 조건을 충족할 수 없다.

## RATIONALE

`pthread_mutexattr_gettype()` 내지 `pthread_mutexattr_setpriotype()`의 `attr` 인자로 지정한 값이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_cond_timedwait()|pthread_cond_timedwait(3p)]]</tt>, <tt>[[pthread_mutex_lock()|pthread_mutex_lock(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
