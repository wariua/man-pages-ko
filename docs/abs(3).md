## NAME

abs, labs, llabs, imaxabs - 정수의 절댓값 계산

## SYNOPSIS

```c
#include <stdlib.h>

int abs(int j);
long labs(long j);
long long llabs(long long j);

#include <inttypes.h>

intmax_t imaxabs(intmax_t j);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`llabs()`:
:   `_ISOC99_SOURCE || _POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`abs()` 함수는 정수 인자 `i`의 절댓값을 계산한다. `labs()`, `llabs()`, `imaxabs()` 함수는 함수별 해당 정수 타입의 인자 `i`의 절댓값을 계산한다.

## RETURN VALUE

함수별 해당 타입의 정수 인자의 절댓값을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `abs()`, `labs()`, `llabs()`, `imaxabs()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD. C89에는 `abs()` 및 `labs()` 함수만 포함돼 있다. C99에서 `llabs()` 및 `imaxabs()` 함수가 추가되었다.

## NOTES

가장 작은 음수 정수의 절댓값을 취하려 할 때의 결과는 규정돼 있지 않다.

glibc 버전 2.0부터 `llabs()` 함수가 포함돼 있다. glibc 버전 2.1.1부터 `imaxabs()` 함수가 포함돼 있다.

`llabs()`가 선언돼 있게 하려면 어떤 표준 헤더도 포함시키기 전에 `_ISOC99_SOURCE`를 (또는 glibc 버전에 따라 `_ISOC9X_SOURCE`를) 정의해 주어야 할 수도 있다.

기본적으로 GCC는 `abs()`와 `labs()`, (그리고 GCC 3.0부터) `llabs()`와 `imaxabs()`를 내장 함수로 처리한다.

## SEE ALSO

`cabs(3)`, `ceil(3)`, `fabs(3)`, `floor(3)`, `rint(3)`

----

2021-03-22
