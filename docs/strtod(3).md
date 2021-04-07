## NAME

strtod, strtof, strtold - ASCII 문자열을 부동소수점수로 변환하기

## SYNOPSIS

```c
#include <stdlib.h>

double strtod(const char *nptr, char **endptr);
float strtof(const char *nptr, char **endptr);
long double strtold(const char *nptr, char **endptr);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>strtof()</code>, <code>strtold()</code>:</dt>
<dd><code>_ISOC99_SOURCE || _POSIX_C_SOURCE >= 200112L</code></dd>
</dl>

## DESCRIPTION

`strtod()`, `strtof()`, `strtold()` 함수는 `nptr`이 가리키는 문자열의 처음 부분을 각각 `double`, `float`, `long double` 표현으로 변환한다.

기대하는 문자열 (시작 부분의) 형식은 선두의 선택적인 공백(`isspace(3)`로 인식), 선택적인 양수('+') 또는 음수('-') 부호, 그리고 (i) 10진수, (ii) 16진수, (iii) 무한대, (iv) NAN (not-a-number) 중 하나가 오는 것이다.

*10진수*는 소수점(로캘에 따라 다르지만 보통 '.')을 포함할 수도 있는 10진수 숫자로 된 비어 있지 않은 열이며, 선택적으로 10진 지수가 따라온다. 10진 지수는 'E' 또는 'e'에 이어 선택적으로 양수 또는 음수 부호가 오고 이어서 10진법 숫자로 된 비어 있지 않은 열이 오며, 10의 거듭제곱으로 곱한다는 뜻이다.

*16진수*는 "0x" 또는 "0X"에 이어 소수점을 포함할 수도 있는 16진수 숫자로 된 비어 있지 않은 열이 오고, 선택적으로 2진 지수가 따라온다. 2진 지수는 'P' 또는 'p'에 이어 선택적으로 양수 또는 음수 부호가 오고 이어서 10진법 숫자로 된 비어 있지 않은 열이 오며, 2의 거듭제곱으로 곱한다는 뜻이다. 소수점과 2진 지수 중 하나는 있어야 한다.

*무한대*는 "INF" 또는 "INFINITY"이며 대소문자 구별이 없다.

*NAN*는 "NAN"(대소문자 구별 없음)에 이어 선택적으로 문자열 `(n-char-sequence)`가 오는 것이다. `n-char-sequence`는 구현별로 다른 방식으로 NAN의 종류를 나타낸다. (NOTES 참고.)

## RETURN VALUE

이 함수들은 변환한 값이 있으면 그 값을 반환한다.

`endptr`이 NULL이 아니면 변환에 쓴 마지막 문자 다음 문자에 대한 포인터를 `endptr`이 가리키는 위치에 저장한다.

어떤 변환도 수행하지 않았으면 0을 반환하며 (`endptr`이 널이 아니면) `nptr`의 값을 `endptr`이 가리키는 위치에 저장한다.

값이 오버플로우를 일으키게 되면 (그 값의 부호에 따라) 양수 또는 음수 `HUGE_VAL`(`HUGE_VALF`, `HUGE_VALL`)을 반환하며 `errno`에 `ERANGE`를 저장한다. 값이 언더플로우를 일으키게 되면 0을 반환하며 `errno`에 `ERANGE`를 저장한다.

## ERRORS

<dl>
<dt><code>ERANGE</code></dt>
<dd>오버플로우나 언더플로우가 발생했다.</dd>
</dl>

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strtod()`, `strtof()`, `strtold()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C99.

`strtod()`는 C89에도 기술되어 있다.

## NOTES

성공과 실패 어느 쪽에서도 적법하게 0을 반환할 수 있으므로 호출 프로그램에서는 호출 전에 `errno`를 0으로 설정하고서 호출 후에 `errno`가 0 아닌 값인지 검사해서 오류가 발생했는지 확인해야 한다.

glibc 구현에서는 "NAN" 뒤에 선택적으로 오는 `n-char-sequence`를 정수로 해석하여 (선택적으로 앞에 '0'이나 '0x'를 붙여서 8진법이나 16진법 선택) 반환 값의 가수 부분에 집어넣는다.

## EXAMPLE

<tt>[[strtol(3)]]</tt> 매뉴얼 페이지의 예를 참고하라. 이 매뉴얼 페이지에서 기술하는 함수들과 사용 방식이 비슷하다.

## SEE ALSO

<tt>[[atof(3)]]</tt>, <tt>[[atoi(3)]]</tt>, <tt>[[atol(3)]]</tt>, `nan(3)`, `nanf(3)`, `nanl(3)`, <tt>[[strfromd(3)]]</tt>, <tt>[[strtol(3)]]</tt>, <tt>[[strtoul(3)]]</tt>

----

2017-09-15
