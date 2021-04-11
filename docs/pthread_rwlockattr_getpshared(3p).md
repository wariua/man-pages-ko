## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_rwlockattr_getpshared, pthread_rwlockattr_setpshared - 읽기-쓰기 락 속성 객체의 프로세스 공유 속성 얻기 및 설정하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_rwlockattr_getpshared(const pthread_rwlockattr_t
    *restrict attr, int *restrict pshared);
int pthread_rwlockattr_setpshared(pthread_rwlockattr_t *attr,
    int pshared);
```

## DESCRIPTION

`pthread_rwlockattr_getpshared()` 함수는 `attr`이 가리키는 속성 객체로부터 `process-shared` 속성의 값을 얻는다. `pthread_rwlockattr_setpshared()` 함수는 `attr`이 가리키는 초기화 된 속성 객체의 `process-shared` 속성을 설정한다.

`process-shared` 속성을 `PTHREAD_PROCESS_SHARED`로 설정하면 읽기-쓰기 락을 할당한 메모리에 접근 가능한 아무 스레드나 그 읽기-쓰기 락을 조작할 수 있게 허용한다. `process-shared` 속성이 `PTHREAD_PROCESS_PRIVATE`이면 읽기-쓰기 락을 초기화 한 스레드와 같은 프로세스 내에 생성된 스레드만 그 읽기-쓰기 락을 조작할 수 있다. 그런 읽기-쓰기 락을 다른 프로세스의 스레드가 조작하려고 시도하는 경우의 동작 방식은 규정되어 있지 않다. `process-shared` 속성의 기본값은 `PTHREAD_PROCESS_PRIVATE`이다.

추가적인 속성과 그 기본값, 그리고 속성 값을 얻고 설정하기 위한 연계 함수의 이름은 구현에서 규정한다.

`pthread_rwlockattr_getpshared()`나 `pthread_rwlockattr_setpshared()`의 `attr` 인자로 지정한 값이 초기화 된 읽기-쓰기 락 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 완료 시 `pthread_rwlockattr_getpshared()` 함수는 0을 반환하며 `attr`의 `process-shared` 속성을 `pshared` 매개변수가 가리키는 객체에 저장한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

성공 시 `pthread_rwlockattr_setpshared()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_rwlockattr_setpshared()` 함수가 실패할 수도 있다.

`EINVAL`
:   속성에 지정한 새 값이 그 속성의 적법한 값 범위를 벗어난다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

없음.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_rwlock_destroy()]]</tt>, <tt>[[pthread_rwlockattr_destroy()]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
