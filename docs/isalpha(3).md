## NAME

isalnum, isalpha, isascii, isblank, iscntrl, isdigit, isgraph, islower, isprint, ispunct, isspace, isupper, isxdigit, isalnum_l, isalpha_l, isascii_l, isblank_l, is_cntrl_l, isdigit_l, isgraph_l, islower_l, isprint_l, ispunct_l, isspace_l, isupper_l, isxdigit_l - 문자 분류 함수들

## SYNOPSIS

```c
#include <ctype.h>

int isalnum(int c);
int isalpha(int c);
int iscntrl(int c);
int isdigit(int c);
int isgraph(int c);
int islower(int c);
int isprint(int c);
int ispunct(int c);
int isspace(int c);
int isupper(int c);
int isxdigit(int c);

int isascii(int c);
int isblank(int c);

int isalnum_l(int c, locale_t locale);
int isalpha_l(int c, locale_t locale);
int isblank_l(int c, locale_t locale);
int iscntrl_l(int c, locale_t locale);
int isdigit_l(int c, locale_t locale);
int isgraph_l(int c, locale_t locale);
int islower_l(int c, locale_t locale);
int isprint_l(int c, locale_t locale);
int ispunct_l(int c, locale_t locale);
int isspace_l(int c, locale_t locale);
int isupper_l(int c, locale_t locale);
int isxdigit_l(int c, locale_t locale);

int isascii_l(int c, locale_t locale);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`isascii()`:
:   `_XOPEN_SOURCE`<br>
    `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19 */ _SVID_SOURCE`

`isblank()`:
:   `ISOC99_SOURCE || _POSIX_C_SOURCE >= 200112L`

`isalnum_l()`, `isalpha_l()`, `isblank_l()`, `iscntrl_l()`, `isdigit_l()`, `isgraph_l()`, `islower_l()`, `isprint_l()`, `ispunct_l()`, `isspace_l()`, `isupper_l()`, `isxdigit_l()`:
:   glibc 2.10부터:
    :   `_XOPEN_SOURCE >= 700`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

`isascii_l()`:
:   glibc 2.10부터:
    :   `_XOPEN_SOURCE >= 700 && (_SVID_SOURCE || _BSD_SOURCE)`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

이 함수들은 지정한 로캘에서 (`unsigned char` 값이거나 `EOF`여야 하는) `c`가 특정 문자 유형에 속하는지 확인한다. 뒤에 "\_l"이 붙지 않은 함수들은 현재 로캘을 가지고 확인한다.

뒤에 "\_l"이 붙은 함수들은 로캘 객체 `locale`이 나타내는 로캘에 따라 검사를 수행한다. `locale`이 특수 로캘 객체 `LC_GLOBAL_LOCALE`(<tt>[[duplocale(3)]]</tt> 참고)이거나 유효한 로캘 객체 핸들이 아닌 경우에 이 함수들의 동작은 규정돼 있지 않다.

아래 목록에선 뒤에 "\_l"이 붙지 않은 함수들의 동작을 설명한다. 뒤에 "\_l"이 붙은 함수들은 현재 로캘 대신 로캘 객체 `locale`을 쓴다는 점만 다르다.

`isalnum()`
:   영문자 또는 숫자 문자인지 확인. `(isalpha(c) || isdigit(c))`와 동등하다.

`isalpha()`
:   영문자인지 확인. 표준 "C" 로캘에선 `(isupper(c) || islower(c))`와 동등하다. 일부 로캘에선 대문자도 아니고 소문자도 아니면서 `isalpha()`가 참인 문자가 추가로 있을 수 있다.

`isascii()`
:   `c`가 ASCII 문자 집합에 들어가는 7비트 `unsigned char` 값인지 확인.

`isblank()`
:   빈칸 문자인지 (공백이나 탭인지) 확인.

`iscntrl()`
:   제어 문자인지 확인.

`isdigit()`
:   (0에서 9까지) 숫자인지 확인.

`isgraph()`
:   공백을 제외한 출력 가능 문자인지 확인.

`islower()`
:   소문자인지 확인.

`isprint()`
:   공백을 포함한 출력 가능 문자인지 확인.

`ispunct()`
:   공백, 대소문자, 숫자 문자가 아닌 출력 가능 문자인지 확인.

`isspace()`
:   여백 문자인지 확인. "C" 및 "POSIX" 로캘에서 공백, 폼 피드(`'\f'`), 개행(`'\n'`), 캐리지 리턴(``\r'`), 수평 탭(`'\t'`), 수직 탭(`'\v'`)이다.

`isupper()`
:   대문자인지 확인.

`isxdigit()`
:   16진수 숫자인지 확인. 즉 다음 중 하나인지 확인.

    ```
    0 1 2 3 4 5 6 7 8 9 a b c d e f A B C D E F
    ```

## RETURN VALUE

문자 `c`가 검사 대상 유형에 속하면 0 아닌 값을 반환한다. 아니면 0을 반환한다.

## VERSIONS

glibc 2.3부터 `isalnum_l()`, `isalpha_l()`, `isblank_l()`, `iscntrl_l()`, `isdigit_l()`, `isgraph_l()`, `islower_l()`, `isprint_l()`, `ispunct_l()`, `isspace_l()`, `isupper_l()`, `isxdigit_l()`, `isascii_l()`을 이용할 수 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `isalnum()`, `isalpha()`, `isascii()`, `isblank()`,<br>`iscntrl()`, `isdigit()`, `isgraph()`, `islower()`,<br>`isprint()`, `ispunct()`, `isspace()`, `isupper()`,<br>`isxdigit()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

C89에서 `isalnum()`, `isalpha()`, `iscntrl()`, `isdigit()`, `isgraph()`, `islower()`, `isprint()`, `ispunct()`, `isspace()`, `isupper()`, `isxdigit()`를 명세하고 있으며 `isascii()`와 `isblank()`는 명세하고 있지 않다. POSIX.1-2001에선 그 함수들뿐 아니라 `isascii()`(XSI 확장)와 `isblank()`도 명세하고 있다. C99에선 `isascii()`를 제외하고 앞선 함수들 모두를 명세하고 있다.

POSIX.1-2008에서 `isascii()`를 구식으로 표시하면서 지역화된 응용에 이식성 있게 사용할 수 없다고 언급하고 있다.

POSIX.1-2008에서 `isalnum_l()`, `isalpha_l()`, `isblank_l()`, `iscntrl_l()`, `isdigit_l()`, `isgraph_l()`, `islower_l()`, `isprint_l()`, `ispunct_l()`, `isspace_l()`, `isupper_l()`, `isxdigit_l()`를 명세하고 있다.

`isascii_l()`은 GNU 확장이다.

## NOTES

표준들에선 이 함수들의 `c` 인자가 `EOF`이거나 `unsigned char` 타입으로 표현 가능한 값이기를 요구한다. `c` 인자가 `char` 타입이라면 다음 예처럼 `unsigned char`로 변환해야 한다.

```c
char c;
...
res = toupper((unsigned char) c);
```

이렇게 해야 하는 이유는 `char`이 `signed char`와 동등할 수 있고, 그 경우 최상위 비트가 설정된 바이트를 `int`로 변환할 때 부호 확장이 이뤄져서 `unsigned char` 범위 밖의 값이 나오기 때문이다.

어느 문자가 어떤 유형에 속하는지는 로캘에 따라 정해진다. 예를 들어 기본 `C` 로캘에선 `isupper()`가 A-움라우트(Ä)를 대문자로 인식하지 않는다.

## SEE ALSO

`iswalnum(3)`, `iswalpha(3)`, `iswblank(3)`, `iswcntrl(3)`, `iswdigit(3)`, `iswgraph(3)`, `iswlower(3)`, `iswprint(3)`, `iswpunct(3)`, `iswspace(3)`, `iswupper(3)`, `iswxdigit(3)`, <tt>[[newlocale(3)]]</tt>, <tt>[[setlocale(3)]]</tt>, `toascii(3)`, <tt>[[tolower(3)]]</tt>, <tt>[[toupper(3)]]</tt>, <tt>[[uselocale(3)]]</tt>, `ascii(7)`, <tt>[[locale(7)]]</tt>

----

2021-03-22
