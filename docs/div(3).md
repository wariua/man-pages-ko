## NAME

div, ldiv, lldiv, imaxdiv - 정수 나눗셈의 몫과 나머지 계산

## SYNOPSIS

```c
#include <stdlib.h>

div_t div(int numerator, int denominator);
ldiv_t ldiv(long numerator, long denominator);
lldiv_t lldiv(long long numerator, long long denominator);

#include <inttypes.h>

imaxdiv_t imaxdiv(intmax_t numerator, intmax_t denominator);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`lldiv()`:
:   `_ISOC99_SOURCE || _POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`div()` 함수는 `numerator`/`denominator` 값을 계산해서 `quot`와 `rem`이라는 (순서가 명세돼 있지 않은) 두 정수 멤버가 있는 `div_t`라는 구조체로 몫과 나머지를 반환한다. 몫을 0에 가까워지게 내림한다. `quot`\*`denominator`+`rem` = `numerator` 관계가 만족된다.

`ldiv()`, `lldiv()`, `imaxdiv()`는 같은 동작을 하되, 표시된 타입의 수로 나눗셈을 해서 표시된 이름의 구조체로 결과를 반환한다. 그 구조체들의 `quot` 및 `rem` 필드가 함수 인자와 타입이 같다.

## RETURN VALUE

`div_t` 등은 구조체다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `div()`, `ldiv()`, `lldiv()`, `imaxdiv()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD. C99에서 `lldiv()` 및 `imaxdiv()` 함수가 추가되었다.

## EXAMPLES

다음 동작 후에

```c
div_t q = div(-5, 3);
```

`q.quot`와 `q.rem`의 값이 각각 -1과 -2이다.

## SEE ALSO

<tt>[[abs(3)]]</tt>, `remainder(3)`

----

2021-03-22
