## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_condattr_destroy, pthread_condattr_init - 조건 변수 속성 객체 파기 및 초기화

## SYNOPSIS

```c
#include <pthread.h>

int pthread_condattr_destroy(pthread_condattr_t *attr);
int pthread_condattr_init(pthread_condattr_t *attr);
```

## DESCRIPTION

`pthread_condattr_destroy()` 함수는 조건 변수 속성 객체를 파기한다. 객체가 초기화 안 된 상태가 되는 효과가 있다.  구현 시 `attr`이 가리키는 객체를 `pthread_condattr_destroy()`에서 비유효 값으로 설정하도록 할 수도 있다. 파기된 `attr` 속성 객체를 `pthread_condattr_init()`으로 다시 초기화 할 수 있다. 파기된 객체를 그 외 방식으로 참조하는 결과는 규정되어 있지 않다.

`pthread_condattr_init()` 함수는 조건 변수 속성 객체 `attr`을 구현에서 정의하는 모든 속성에 대해 기본값으로 초기화 한다.

이미 초기화 된 `attr` 속성 객체를 지정해서 `pthread_condattr_init()`을 호출하는 경우의 결과는 규정되어 있지 않다.

조건 변수 속성 객체를 이용해 하나 이상의 조건 변수를 초기화 한 후에는 그 속성 객체에 영향을 주는 (파기를 포함한) 어떤 함수도 이미 초기화 된 조건 변수에 영향을 주지 않는다.

POSIX.1-2008의 이 권에서는 `clock` 속성과 `process-shared` 속성을 요구한다.

추가적인 속성과 그 기본값, 그리고 속성 값을 얻고 설정하기 위한 연계 함수의 이름은 구현에서 규정한다.

`pthread_condattr_destroy()`의 `attr` 인자로 지정한 값이 초기화 된 조건 변수 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_condattr_destroy()` 및 `pthread_condattr_init()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_condattr_init()` 함수가 실패한다.

`ENOMEM`
:   조건 변수 속성 객체를 초기화 하기에 충분한 메모리가 없다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

뮤텍스에서와 같은 이유로 조건 변수에 `process-shared` 속성이 규정되었다.

`pthread_condattr_destroy()`의 `attr` 인자로 지정한 값이 초기화 된 조건 변수 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_attr_destroy()` 및 `pthread_mutex_destroy()`도 참고.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_attr_destroy()]]</tt>, <tt>[[pthread_cond_destroy()]]</tt>, <tt>[[pthread_condattr_getpshared()]]</tt>, <tt>[[pthread_create()]]</tt>, <tt>[[pthread_mutex_destroy()]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
