## NAME

strtoul, strtoull, strtouq - 문자열을 부호 없는 long 정수로 변환하기

## SYNOPSIS

```c
#include <stdlib.h>

unsigned long strtoul(const char *restrict nptr,
                      char **restrict endptr, int base);
unsigned long long strtoull(const char *restrict nptr,
                      char **restrict endptr, int base);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`strtoull()`:
:   `_ISOC99_SOURCE`<br>
    `    || /* glibc <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE`

## DESCRIPTION

`strtoul()` 함수는 `nptr`에 있는 문자열의 처음 부분을 지정한 `base`에 따라 `unsigned long` 값으로 변환한다. 기수는 2와 36 사이이거나 특수한 값 0이어야 한다.

문자열이 임의 개수의 공백(`isspace(3)`으로 판단)으로 시작할 수 있으며 그 다음에 선택적으로 '+' 내지 '-' 부호 한 개가 올 수 있다. `base`가 0이나 16이면 문자열에서 다음에 "0x" 접두부가 있을 수 있으며, 그러면 수를 16진수로 읽어 들인다. 그렇지 않고 다음 문자가 '0'이면 `base` 0을 8로 (8진수로) 받아들이며, 아니면 10으로 (10진수로) 받아들인다.

문자열 나머지를 명백한 방식으로 `unsigned long` 값으로 변환하며 해당 기수에서 유효 숫자가 아닌 첫 번째 문자에서 멈춘다. (기수가 10보다 큰 경우 대소문자 글자 'A'가 10을, 'B'가 11을 나타내며, 그런 식으로 'Z'가 35를 나타낸다.)

`endptr`이 NULL이 아니면 `strtoul()`은 첫 번째 비유효 문자의 주소를 `*endptr`에 저장한다. 숫자가 전혀 없었으면 `strtoul()`은 `nptr`의 원래 값을 `*endptr`에 저장한다. (그리고 0을 반환한다.) 특히 `*nptr`이 '\0'이 아닌데 반환 시 `**endptr`이 '\0'이면 문자열 전체가 유효한 것이다.

`strtoull()` 함수는 `strtoul()` 함수처럼 동작하되 `unsigned long long` 값을 반환한다.

## RETURN VALUE

원래 값이 오버플로우 되지 않으면 `strtoul()` 함수는 변환 결과를, 또는 앞에 음수 부호가 있었으면 부호 없는 값으로 표현한 변환 결과의 반수를 반환한다. 오버플로우가 일어나면 `strtoul()`은 `ULONG_MAX`를 반환하며 `errno`를 `ERANGE`로 설정한다. 같은 내용이 `strtoull()`에 (`ULONG_MAX` 대신 `ULLONG_MAX`로) 적용된다.

## ERRORS

`EINVAL`
:   (C99에는 없음) 지정한 `base`가 지원하지 않는 값을 담고 있다.

`ERANGE`
:   결과 값이 범위를 벗어났다.

구현에서 변환을 전혀 수행하지 않은 경우에 (숫자 없음, 0 반환) `errno`를 `EINVAL`로 설정할 수도 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strtoul()`, `strtoull()`, `strtouq()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

`strtoul()`: POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4.

`strtoull()`: POSIX.1-2001, POSIX.1-2008, C99.

## NOTES

`strtoul()`이 성공과 실패 어느 쪽에서도 적법하게 0이나 `ULONG_MAX`를 (`strtoull()`에선 `ULLONG_MAX`) 반환할 수 있으므로 호출 프로그램에서는 호출 전에 `errno`를 0으로 설정하고서 호출 후에 `errno`가 0 아닌 값인지 검사해서 오류가 발생했는지 확인해야 한다.

"C" 외의 로캘에서 다른 숫자 문자열들을 받아들일 수도 있다. (예를 들어 현재 로캘의 천 단위 구분자를 지원할 수도 있다.)

BSD에는 완전히 유사하게 정의된 다음 함수가 있다.

```c
u_quad_t strtouq(const char *nptr, char **endptr, int base);
```

현재 아키텍처의 워드 크기에 따라 `strtoull()`이나 `strtoul()`과 동등할 수 있다.

음수 값을 유효한 것으로 보아 조용히 동등한 `unsigned long` 값으로 변환한다.

## EXAMPLES

<tt>[[strtol(3)]]</tt> 매뉴얼 페이지의 예를 참고하라. 이 매뉴얼 페이지에서 기술하는 함수들과 사용 방식이 비슷하다.

## SEE ALSO

`a64l(3)`, <tt>[[atof(3)]]</tt>, <tt>[[atoi(3)]]</tt>, <tt>[[atol(3)]]</tt>, <tt>[[strtod(3)]]</tt>, <tt>[[strtol(3)]]</tt>, <tt>[[strtoumax(3)]]</tt>

----

2021-03-22
