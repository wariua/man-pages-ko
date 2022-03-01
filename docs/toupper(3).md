## NAME

toupper, tolower, toupper_l, tolower_l - 대문자나 소문자로 바꾸기

## SYNOPSIS

```c
#include <ctype.h>

int toupper(int c);
int tolower(int c);

int toupper_l(int c, locale_t locale);
int tolower_l(int c, locale_t locale);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`toupper_l()`, `tolower_l()`:
:   glibc 2.10부터:
    :   `_XOPEN_SOURCE >= 700`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

이 함수들은 소문자를 대문자로, 그리고 그 반대로 변환한다.

`c`가 소문자인 경우에, 현재 로캘에서 대응하는 대문자 표현이 존재하면 `toupper()`가 그 대문자를 반환한다. 아니면 `c`를 반환한다. `toupper_l()` 함수는 같은 작업을 수행하되 로캘 핸들 `locale`이 가리키는 로캘을 쓴다.

`c`가 대문자인 경우에, 현재 로캘에서 대응하는 소문자 표현이 존재하면 `tolower()`가 그 소문자를 반환한다. 아니면 `c`를 반환한다. `tolower_l()` 함수는 같은 작업을 수행하되 로캘 핸들 `locale`이 가리키는 로캘을 쓴다.

`c`가 `unsigned char` 값이나 `EOF`가 아닌 경우 이 함수들의 동작은 규정돼 있지 않다.

`locale`이 특수 로캘 객체 `LC_GLOBAL_LOCALE`(<tt>[[duplocale(3)]]</tt> 참고)이거나 유효한 로캘 객체 핸들이 아닌 경우에 `toupper_l()` 및 `tolower_l()`의 동작은 규정돼 있지 않다.

## RETURN VALUE

변환된 글자의 값을 반환한다. 변환이 불가능하면 `c`를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `toupper()`, `tolower()`, `toupper_l()`, `tolower_l()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`toupper()`, `tolower()`: C89, C99, 4.3BSD, POSIX.1-2001, POSIX.1-2008.

`toupper_l()`, `tolower_l()`: POSIX.1-2008.

## NOTES

표준들에선 이 함수들의 `c` 인자가 `EOF`이거나 `unsigned char` 타입으로 표현 가능한 값이기를 요구한다. `c` 인자가 `char` 타입이라면 다음 예처럼 `unsigned char`로 변환해야 한다.

```c
char c;
...
res = toupper((unsigned char) c);
```

이렇게 해야 하는 이유는 `char`이 `signed char`와 동등할 수 있고, 그 경우 최상위 비트가 설정된 바이트를 `int`로 변환할 때 부호 확장이 이뤄져서 `unsigned char` 범위 밖의 값이 나오기 때문이다.

어느 문자가 대문자나 소문자인지는 로캘에 따라 정해진다. 예를 들어 기본 `"C"` 로캘은 움라우트를 인식하지 못하므로 변환이 이뤄지지 않는다.

일부 비영어 로캘에는 대응하는 대문자가 없는 소문자가 있다. 독일어의 에스체트가 한 예다.

## SEE ALSO

<tt>[[isalpha(3)]]</tt>, <tt>[[newlocale(3)]]</tt>, <tt>[[setlocale(3)]]</tt>, `towlower(3)`, `towupper(3)`, <tt>[[uselocale(3)]]</tt>, <tt>[[locale(7)]]</tt>

----

2021-03-22
