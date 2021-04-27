## NAME

strerror, strerrorname_np, strerrordesc_np, strerror_r, strerror_l - 오류 번호를 설명하는 문자열 반환

## SYNOPSIS

```c
#include <string.h>

char *strerror(int errnum);
const char *strerrorname_np(int errnum);
const char *strerrordesc_np(int errnum);

int strerror_r(int errnum, char *buf, size_t buflen);
               /* XSI 준수 */

char *strerror_r(int errnum, char *buf, size_t buflen);
               /* GNU 전용 */

char *strerror_l(int errnum, locale_t locale);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`strerrorname_np()`, `strerrordesc_np()`:
:   `_GNU_SOURCE`

`strerror_r()`:
:   다음 경우에 XSI 준수 버전 제공:<br>
    `    (_POSIX_C_SOURCE >= 200112L) && ! _GNU_SOURCE`<br>
    아니면 GNU 전용 버전 제공.

## DESCRIPTION

`strerror()` 함수는 `errnum` 인자로 준 오류 코드를 설명하는 문자열 포인터를 반환하며, 현재 로캘의 `LC_MESSAGES` 요소로 적절한 언어를 선택할 수도 있다. (예를 들어 `errnum`가 `EINVAL`이면 반환되는 설명이 "Invalid argument"이다.) 응용에서 그 문자열을 변경해선 안 되며, 후속 `strerror()` 내지 `strerror_l()` 호출에서 그 문자열을 변경할 수 있다. <tt>[[perror(3)]]</tt>를 포함한 다른 라이브러리 함수들은 이 문자열을 변경하지 않는다.

`strerrordesc_np()` 함수는 `strerror()`과 마찬가지로 `errnum` 인자로 준 오류 코드를 설명하는 문자열 포인터를 반환하는데, 반환되는 문자열이 현재 로캘에 따라 번역돼 있지 않다는 점이 다르다.

`strerrorname_np()` 함수는 `errnum` 인자로 준 오류 코드의 이름을 담은 문자열 포인터를 반환한다. 예를 들어 `EPERM`을 인자로 주면 문자열 "EPERM"의 포인터를 반환한다.

### `strerror_r()`

`strerror_r()` 함수는 `strerror()`와 비슷하되 스레드에 안전하다. 이 함수는 두 가지 버전이 있다. POSIX.1-2001에 명세된 XSI 준수 버전(glibc 2.3.4부터 사용 가능하지만 glibc 2.13까지는 XSI 준수 아님)과 GNU 전용 버전(glibc 2.0부터 사용 가능)이다. SYNOPSIS에 있는 기능 확인 매크로 구성으로는 XSI 준수 버전이 제공되고, 아니면 GNU 전용 버전이 제공된다. 명시적으로 어떤 기능 확인 매크로도 정의하지 않으면 (glibc 2.4부터) 기본적으로 `_POSIX_C_SOURCE`가 200112L 값으로 정의되고, 그래서 `strerror_r()`의 XSI 준수 버전이 기본적으로 제공된다.

이식 가능한 응용에서는 XSI 준수 `strerror_r()`가 바람직하다. 사용자가 제공하는 `buflen` 길이의 버퍼 `buf`로 오류 문자열을 반환한다.

GNU 전용 `strerror_r()`은 오류 메시지를 담은 문자열의 포인터를 반환한다. 함수에서 `buf`에 저장한 문자열의 포인터일 수도 있고 (`buf`를 쓰지 않는 경우에는) 어떤 (불변) 정적 문자열의 포인터일 수도 있다. 함수에서 `buf`에 문자열을 저장하는 경우에는 최대 `buflen` 바이트가 저장된다. (`buflen`이 너무 작고 `errnum`가 알 수 없는 값이면 문자열이 잘려 있을 수 있다.) 그 문자열에는 항상 종료용 널 바이트('\0')가 들어간다.

### `strerror_l()`

`strerror_l()`은 `strerror()`와 비슷하되 `errnum`을 `locale`에 지정한 로캘의 로캘 의존적 오류 메시지로 연결한다. `locale`이 특수 로캘 객체 `LC_GLOBAL_LOCALE`이거나 유효한 로캘 객체 핸들이 아닌 경우 `strerror_l()`의 동작 방식은 규정돼 있지 않다.

## RETURN VALUE

`strerror()`, `strerror_l()`, GNU 전용 `strerror_r()` 함수는 적절한 오류 설명 문자열을 반환한다. 알 수 없는 오류 번호면 "Unknown error nnn" 메시지를 반환한다.

성공 시 `strerrname_np()`와 `strerrordesc_np()`는 적절한 오류 설명 문자열을 반환한다. `errnum`이 유효하지 않은 오류 번호이면 NULL을 반환한다.

XSI 준수 `strerror_r()` 함수는 성공 시 0을 반환한다. 오류 시 (양수) 오류 번호를 반환하거나 (glibc 2.13부터), -1을 반환하고 오류를 나타내도록 `errno`를 설정한다 (glibc 버전 2.13 전).

POSIX.1-2001과 POSIX.1-2008에서는 `strerror()`나 `strerror_l()` 호출 성공 시 `errno`를 바꾸지 않아야 한다고 요구하고 있으며, 오류를 나타내기 위한 함수 반환 값이 없기 때문에 응용에서 호출 전에 `errno`를 0으로 설정하고 호출 후에 `errno`를 확인해야 한다고 언급한다.

## ERRORS

`EINVAL`
:   `errnum` 값이 유효한 오류 번호가 아니다.

`ERANGE`
:   제공 저장 공간이 오류 설명 문자열을 담기에 충분하지 않다.

## VERSIONS

glibc 2.6에서 `strerror_l()` 함수가 처음 등장했다.

glibc 2.32에서 `strerrorname_np()` 및 `strerrordesc_np()` 함수가 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strerror()` | 스레드 안전성 | MT-Unsafe race:strerror |
| `strerrorname_np()`, `strerrordesc_np()` | 스레드 안전성 | MT-Safe |
| `strerror_r()`, `strerror_l()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`strerror()`는 POSIX.1-2001, POSIX.1-2008, C89, C99에 명세돼 있다. `strerror_r()`은 POSIX.1-2001, POSIX.1-2008에 명세돼 있다.

`strerror_l()`은 POSIX.1-2008에 명세돼 있다.

GNU 전용 함수 `strerror_r()`, `strerrorname_np()`, `strerrordesc_np()`는 비표준 확장이다.

POSIX.1-2001에서는 `strerror()` 호출에서 오류를 만난 경우 `errno`를 설정하는 건 허용하지만 오류 시에 함수 결과로 어떤 값을 반환해야 하는지는 명시하지 않고 있다. 어떤 시스템에서는 알 수 없는 오류 번호인 경우 NULL을 반환한다. 다른 시스템에서는 알 수 없는 오류 번호인 경우 `strerror()`가 "Error nnn occurred" 비슷한 문자열을 반환하고 `errno`를 `EINVAL`로 설정한다. C99와 POSIX.1-2008에서는 반환 값이 NULL이 아니어야 한다고 요구한다.

## NOTES

GNU C 라이브러리에서는 `strerror()`에 1024개 문자로 된 버퍼를 쓴다. 이 크기면 `strerror_r()` 호출 시 `ERANGE` 오류를 피하기에 충분할 것이다.

`strerrorname_np()`와 `strerrordesc_np()`는 스레드 안전이고 비동기 시그널 안전이다.

## SEE ALSO

<tt>[[err(3)]]</tt>, <tt>[[errno(3)]]</tt>, <tt>[[error(3)]]</tt>, <tt>[[perror(3)]]</tt>, <tt>[[strsignal(3)]]</tt>, <tt>[[locale(7)]]</tt>

----

2021-03-22
