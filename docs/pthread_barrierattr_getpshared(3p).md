## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_barrierattr_getpshared, pthread_barrierattr_setpshared - 배리어 속성 객체의 프로세스 공유 속성 얻기 및 설정하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_barrierattr_getpshared(const pthread_barrierattr_t
    *restrict attr, int *restrict pshared);
int pthread_barrierattr_setpshared(pthread_barrierattr_t *attr,
    int pshared);
```

## DESCRIPTION

`pthread_barrierattr_getpshared()` 함수는 `attr`이 가리키는 속성 객체로부터 `process-shared` 속성의 값을 얻는다. `pthread_barrierattr_setpshared()` 함수는 `attr`이 가리키는 초기화 된 속성 객체의 `process-shared` 속성을 설정한다.

`process-shared` 속성을 `PTHREAD_PROCESS_SHARED`로 설정하면 배리어를 할당한 메모리에 접근 가능한 아무 스레드나 그 배리어를 조작할 수 있게 허용한다. `process-shared` 속성이 `PTHREAD_PROCESS_PRIVATE`이면 배리어를 초기화 한 스레드와 같은 프로세스 내에 생성된 스레드만 그 배리어를 조작할 수 있다. 그런 배리어를 다른 프로세스의 스레드가 조작하려고 시도하는 경우의 동작 방식은 규정되어 있지 않다. 속성의 기본값은 `PTHREAD_PROCESS_PRIVATE`이다. 두 상수 `PTHREAD_PROCESS_SHARED`와 `PTHREAD_PROCESS_PRIVATE`이 `<pthread.h>`에 정의되어 있다.

추가적인 속성과 그 기본값, 그리고 속성 값을 얻고 설정하기 위한 연계 함수의 이름은 구현에서 규정한다.

`pthread_barrierattr_getpshared()`나 `pthread_barrierattr_setpshared()`의 `attr` 인자로 지정한 값이 초기화 된 배리어 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_barrierattr_getpshared()` 함수는 0을 반환하며 `attr`의 `process-shared` 속성을 `pshared` 매개변수가 가리키는 객체에 저장한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

성공 시 `pthread_barrierattr_setpshared()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_barrierattr_setpshared()` 함수가 실패할 수도 있다.

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>process-shared</code> 속성에 지정한 새 값이 적법한 값인 <code>PTHREAD_PROCESS_SHARED</code>나 <code>PTHREAD_PROCESS_PRIVATE</code> 중 하나가 아니다.</dd>
</dl>

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

없음.

## APPLICATION USAGE

`pthread_barrierattr_getpshared()` 및 `pthread_barrierattr_setpshared()` 함수는 Thread Process-Shared Synchronization 옵션의 일부이므로 모든 구현에서 제공할 필요는 없다.

## RATIONALE

`pthread_barrierattr_getpshared()`나 `pthread_barrierattr_setpshared()`의 `attr` 인자로 지정한 값이 초기화 된 배리어 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_barrier_destroy()|pthread_barrier_destroy(3p)]]</tt>, <tt>[[pthread_barrierattr_destroy()|pthread_barrierattr_destroy(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
