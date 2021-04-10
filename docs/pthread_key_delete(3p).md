## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_key_delete - 스레드별 데이터 키 삭제

## SYNOPSIS

```c
#include <pthread.h>

int pthread_key_delete(pthread_key_t key);
```

## DESCRIPTION

`pthread_key_delete()` 함수는 앞서 `pthread_key_create()`가 반환한 스레드별 데이터 키를 삭제한다. `pthread_key_delete()`를 호출하는 시점에 `key`에 연계된 스레드별 데이터 값들이 NULL일 필요는 없다. 삭제되는 키 내지 연계된 스레드별 데이터와 관련된 자료 구조를 위한 응용 저장 공간을 해제하거나 정리 동작을 수행할 책임은 응용에게 있다. 그 정리는 `pthread_key_delete()` 호출 전에 이뤄질 수도 있고 후에 이뤄질 수도 있다. `pthread_key_delete()` 호출 후에 `key`를 사용하려는 시도가 유발하는 동작은 규정되어 있지 않다.

소멸자 함수 내에서 `pthread_key_delete()` 함수를 호출할 수 있다. `pthread_key_delete()`에서는 어떤 소멸자 함수도 호출하지 않는다. `key`에 연계된 소멸자 함수가 있었다면 스레드 종료 때 더 이상 호출되지 않는다.

## RETURN VALUE

성공 시 `pthread_key_delete()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

`pthread_key_delete()` 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

스레드별 데이터 키 삭제 함수가 포함된 것은 안 쓰는 데이터 키에 연계된 자원을 해제할 수 있도록 하기 위해서이다. 안 쓰는 스레드별 데이터 키가 생길 수 있는 한 경우는 키를 할당했던 동적 적재 모듈이 내려갈 때이다.

스레드별 데이터 값이 가리키는 데이터를 포함해서 삭제될 키에 연계된 자료 구조에 대해 필요한 정리 동작을 수행할 책임은 응용에게 있다. `pthread_key_delete()`는 어떤 정리 작업도 하지 않는다. 특히 소멸자 함수를 호출하지 않는다. 책임을 이렇게 나눈 여러 이유가 있다.

1. 스레드 종료 시점에 스레드별 데이터를 해제하는 데 사용하는 연계 소멸자 함수는 스레드별 데이터를 할당한 스레드 내에서 호출될 때에만 올바른 동작이 보장된다. (소멸자 자체에서 스레드별 데이터를 활용할 수도 있다.) 따라서 키 삭제 시점에 소멸자를 사용해 다른 스레드의 스레드별 데이터를 해제할 수 없다. 키 삭제 시점에 다른 스레드에서 소멸자를 호출하게 하려면 다른 스레드를 비동기적으로 중단시켜야 할 것이다. 하지만 중단된 스레드는 임의 상태에 있을 수 있을 것이고, 가령 소멸자 실행에 필요한 락을 잡고 있을 수도 있다. 따라서 이 방식은 실패하게 된다. 일반적으로 구현에서 키 삭제 시점에 스레드별 데이터를 안전하게 해제할 수 있는 메커니즘은 없다.

2. 삭제할 키에 연계된 스레드별 데이터를 안전하게 해제할 방법이 있다고 하더라도 그러기 위해선 구현에서 NULL 아닌 데이터를 가진 스레드를 열거할 수 있어야 하며 키 삭제가 일어나는 동안 스레드들이 스레드별 데이터를 추가로 생성하지 못하게 막을 수 있어야 한다. 이런 특수 경우 때문에 달리 필요치 않을 동기화가 정상 경우에도 추가로 생길 수 있다.

키를 삭제해도 안전하다는 것을 응용에서 확신하려면 그 키를 사용할 가능성이 조금이라도 있는 모든 스레드에서 다시는 그 키를 사용하지 않을 것임을 알아야 한다. 예를 들어 내리려는 모듈이 있을 때 모든 클라이언트 스레드가 정리 프로시저를 호출해서 참조 카운트를 0으로 설정하여 모듈에 더는 볼일이 없다고 선언한다면 안전을 확신할 수 있을 것이다.

`pthread_key_delete()`의 `key` 인자로 지정한 값이 `pthread_key_create()`으로 얻은 키 값을 가리키지 않거나 `pthread_key_delete()`로 삭제된 키를 가리키고 있음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

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
