## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_getspecific, pthread_setspecific - 스레드별 데이터 관리

## SYNOPSIS

```c
#include <pthread.h>

void *pthread_getspecific(pthread_key_t key);
int pthread_setspecific(pthread_key_t key, const void *value);
```

## DESCRIPTION

`pthread_getspecific()` 함수는 현재 호출 스레드에 대해서 지정한 `key`에 결속되어 있는 값을 반환한다.

`pthread_setspecific()` 함수는 앞선 `pthread_key_create()` 호출을 통해 얻은 `key`에 스레드별 `value`를 연계한다. 같은 키에 스레드마다 다른 값을 결속할 수 있다. 보통 그 값은 호출 스레드에서 사용하려고 준비한 동적 할당 메모리 블록에 대한 포인터이다.

`pthread_key_create()`로 얻은 것이 아닌 `key` 값에 대해서, 또는 `pthread_key_delete()`로 `key`를 삭제한 후에 `pthread_getspecific()`이나 `pthread_setspecific()`을 호출하는 결과는 규정되어 있지 않다.

`pthread_getspecific()`과 `pthread_setspecific()` 모두 스레드별 데이터 소멸자 함수 내에서 호출 가능하다. 소멸 중인 스레드별 데이터 키에 대해 `pthread_getspecific()`을 호출하면 (소멸자 시작 후에) `pthread_setspecific()`으로 그 값을 바꾸지 않았다면 NULL 값을 반환한다. 스레드별 데이터 소멸자 루틴 내에서 `pthread_setspecific()`을 호출하는 결과는 (최소 `PTHREAD_DESTRUCTOR_ITERATIONS`번의 소멸 시도 후) 저장 값 유실이나 무한 루프일 수 있다.

두 함수 모두 매크로로 구현되어 있을 수 있다.

## RETURN VALUE

`pthread_getspecific()` 함수는 지정한 `key`에 연계된 스레드별 데이터 값을 반환한다. `key`에 연계된 스레드별 데이터 값이 없으면 NULL 값을 반환한다.

성공 시 `pthread_setspecific()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

`pthread_getspecific()`은 어떤 오류도 반환하지 않는다.

다음 경우에 `pthread_setspecific()` 함수가 실패한다.

`ENOMEM`
:   NULL 아닌 값을 키에 연계하기에 충분한 메모리가 없다.

`pthread_setspecific()` 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

스레드별 데이터에 상태를 유지하는 기능들에는 `pthread_getspecific()`의 성능과 편의성이 아주 중요하다. 의무적으로 탐지해야 하는 오류가 없으며 탐지할 수 있을 유일한 오류는 유효하지 않은 키 사용뿐이기 때문에 `pthread_getspecific()` 기능은 오류 보고보다 속도와 단순함을 선호하도록 설계되어 왔다.

`pthread_setspecific()`의 `key` 인자로 지정한 값이 `pthread_key_create()`으로 얻은 키 값을 가리키지 않거나 `pthread_key_delete()`로 삭제된 키를 가리키고 있음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_key_create()|pthread_key_create(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
