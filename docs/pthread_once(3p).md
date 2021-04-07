## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_once - 패키지 동적 초기화

## SYNOPSIS

```c
#include <pthread.h>

int pthread_once(pthread_once_t *once_control,
    void (*init_routine)(void));
pthread_once_t once_control = PTHREAD_ONCE_INIT;
```

## DESCRIPTION

프로세스 내의 한 스레드에 의해 어떤 `once_control`로 `pthread_once()`가 처음 호출되는 것이면 인자 없이 `init_routine`을 호출한다. 이후 같은 `once_control`로 `pthread_once()` 호출을 하면 `init_routine`을 호출하지 않는다. `pthread_once()`에서 반환 시에는 `init_routine`이 완료되어 있다. `once_control` 매개변수를 통해 관련 초기화 루틴을 호출했는지 판단한다.

`pthread_once()` 함수는 취소점이 아니다. 하지만 `init_routine`이 취소점이어서 취소가 되는 경우 `once_control`에 대한 작용은 `pthread_once()`가 전혀 호출되지 않은 경우와 같다.

상수 `PTHREAD_ONCE_INIT`이 `<pthread.h>` 헤더에 정의되어 있다.

`once_control`의 저장 기간이 자동이거나 `PTHREAD_ONCE_INIT`으로 초기화 되어 있지 않은 경우 `pthread_once()`의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 완료 시 `pthread_once()`는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

`pthread_once()` 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

어떤 C 라이브러리들은 동적 초기화를 하도록 설계되어 있다. 즉, 라이브러리 내의 첫 번째 프로시저가 불릴 때 라이브러리 전역 초기화를 수행한다. 단일 스레드 프로그램에서는 보통 다음과 같이 정적 변수를 두고 루틴 진입 시마다 확인하는 식으로 구현한다.

```c
static int random_is_initialized = 0;
extern int initialize_random();

int random_function()
{
    if (random_is_initialized == 0) {
        initialize_random();
        random_is_initialized = 1;
    }
    ... /* 초기화 후 수행하는 동작들 */
}
```

다중 스레드 프로그램에서 같은 구조를 유지하려면 새로운 구성 요소가 필요하다. 안 그러면 라이브러리 사용에 앞서 라이브러리에서 내보이는 초기화 함수를 명시적으로 호출하는 방식으로 라이브러리 초기화를 해야 한다.

다중 스레드 프로세스에서의 동적 라이브러리 초기화를 위해선 단순한 초기화 플래그로는 충분하지 않다. 라이브러리를 동시에 호출하는 여러 스레드들에 대해서 플래그를 보호할 필요가 있기 때문이다. 플래그 보호를 위해선 뮤텍스를 사용해야 하는데 뮤텍스를 쓰려면 먼저 초기화를 해야 한다. 그리고 뮤텍스를 한 번만 초기화 되도록 하기 위해선 이 문제에 대한 순환적인 해법이 필요하다.

`pthread_once()` 사용은 구현에서 보장하는 동적 초기화 수단을 제공하는 것만이 아니라 다중 스레드이며 실시간인 시스템을 믿을 만하게 만드는 데 도움을 준다. 앞의 예가 다음과 같이 된다.

```c
#include <pthread.h>
static pthread_once_t random_is_initialized = PTHREAD_ONCE_INIT;
extern int initialize_random();

int random_function()
{
    (void) pthread_once(&random_is_initialized, initialize_random);
    ... /* 초기화 후 수행하는 동작들 */
}
```

참고로 일부 컴파일러에서 `&<배열_이름>` 구문을 허용하지 않으므로 `pthread_once_t`가 배열일 수 없다.

`pthread_once()`의 `once_control` 인자로 지정한 값이 `PTHREAD_ONCE_INIT`으로 초기화 한 `pthread_once_t` 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
