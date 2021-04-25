## NAME

strfromd, strfromf, strfroml - 부동소수점 값을 문자열로 변환하기

## SYNOPSIS

```c
#include <stdlib.h>

int strfromd(char *restrict str, size_t n,
             const char *restrict format, double fp);
int strfromf(char *restrict str, size_t n,
             const char *restrict format, float fp);
int strfroml(char *restrict str, size_t n,
             const char *restrict format, long double fp);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`strfromd()`, `strfromf()`, `strfroml()`:
:   `__STDC_WANT_IEC_60559_BFP_EXT__`

## DESCRIPTION

이 함수들은 설정 가능한 `format`에 따라 부동소수점 값 `fp`를 문자열 `str`으로 변환한다. `str`에 최대 `n` 개 문자를 저장한다.

`n`이 충분히 큰 경우에만 종료용 널 바이트(`'\0'`)를 써 넣는다. 그렇지 않으면 `n` 개 문자에서 잘린 문자열을 써 넣는다.

`strfromd()`, `strfromf()`, `strfroml()` 함수는 `format` 문자열을 빼면 다음과 동등하다.

```c
snprintf(str, n, format, fp);
```

### 서식 문자열의 형식

`format` 문자열은 문자 '%'로 시작해야 한다. 그 다음에 선택적으로 정밀도가 오는데, 마침표 문자(.)로 시작하고 선택적으로 10진법 정수가 따라온다. 마침표 다음에 정수를 지정하지 않으면 정밀도를 0으로 한다. 마지막으로 서식 문자열에는 변환 지정자 `a`, `A`, `e`, `E`, `f`, `F`, `g`, `G` 중 하나가 있어야 한다.

함수 뒷부분이 나타내는 부동소수점 종류에 따라 변환 지정자를 적용한다. 따라서 `snprintf()`와는 달리 서식 문자열에 길이 수식자 문자가 없다. 변환 지정자들에 대한 자세한 설명은 <tt>[[snprintf(3)]]</tt>를 보라.

NaN 및 무한대 값의 변환에서 C99 표준을 준수한다.

> `fp`가 NaN, +NaN, -NaN이고 변환 지정자가 `f`(또는 `a`, `e`, `g`)이면 변환 결과는 각각 "nan", "nan", "-nan"이다. 변환 지정자가 `F`(또는 `A`, `E`, `G`)이면 변환 결과는 "NAN"이나 "-NAN"이다.
>
> 마찬가지로 `fp`가 무한대이면 [-]inf나 [-]INF로 변환된다.

잘못된 형식의 `format` 문자열로 인한 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

`strfromd()`, `strfromf()`, `strfroml()` 함수는 `n`이 충분히 클 때 `str`에 써 넣게 되는 문자 개수를 반환한다. 종료용 널 바이트는 세지 않는다. 따라서 반환 값이 `n` 이상이면 출력이 잘렸다는 뜻이다.

## VERSIONS

glibc 버전 2.25부터 `strfromd()`, `strfromf()`, `strfroml()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>와 GNU C 라이브러리 매뉴얼 **POSIX Safety Concepts** 절을 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strfromd()`,<br>`strfromf()`,<br>`strfroml()` | 스레드 안전성        | MT-Safe locale |
| `strfromd()`,<br>`strfromf()`,<br>`strfroml()` | 비동기 시그널 안전성 | AS-Unsafe heap |
| `strfromd()`,<br>`strfromf()`,<br>`strfroml()` | 비동기 취소 안전성   | AC-Unsafe mem  |

참고: 예비 속성들이다.

## CONFORMING TO

C99, ISO/IEC TS 18661-1.

## NOTES

`strfromd()`, `strfromf()`, `strfroml()` 함수에서는 현재 로캘의 `LC_NUMERIC` 범주를 따진다.

## EXAMPLE

`float` 타입 값 12.1을 10진법 문자열로 변환하면 "12.100000"가 나온다.

```c
#define __STDC_WANT_IEC_60559_BFP_EXT__
#include <stdlib.h>
int ssize = 10;
char s[ssize];
strfromf(s, ssize, "%f", 12.1);
```

`float` 타입 값 12.3456을 정밀도 2자리인 10진법 문자열로 변환하면 "12.35"가 나온다.

```c
#define __STDC_WANT_IEC_60559_BFP_EXT__
#include <stdlib.h>
int ssize = 10;
char s[ssize];
strfromf(s, ssize, "%.2f", 12.3456);
```

`double` 타입 값 12.345e19를 정밀도 0자리인 과학적 기수법 문자열로 변환하면 "1E+20"이 나온다.

```c
#define __STDC_WANT_IEC_60559_BFP_EXT__
#include <stdlib.h>
int ssize = 10;
char s[ssize];
strfromd(s, ssize, "%.E", 12.345e19);
```

## SEE ALSO

<tt>[[atof(3)]]</tt>, <tt>[[snprintf(3)]]</tt>, <tt>[[strtod(3)]]</tt>

----

2021-03-22
