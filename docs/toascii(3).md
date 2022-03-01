## NAME

toascii - 문자를 ASCII로 바꾸기

## SYNOPSIS

```c
#include <ctype.h>

int toascii(int c);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`toascii()`:
:   `_XOPEN_SOURCE`<br>
    `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19 */ _SVID_SOURCE || _BSD_SOURCE`

## DESCRIPTION

`toascii()`는 `c`의 상위 비트를 지워서 ASCII 문자 집합에 들어가는 7비트 `unsigned char` 값으로 변환한다.

## RETURN VALUE

변환된 문자의 값을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `toascii()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

SVr4, BSD, POSIX.1-2001. POSIX.1-2008에서 `toascii()`를 구식으로 표시하면서 지역화된 응용에 이식성 있게 사용할 수 없다고 언급하고 있다.

## BUGS

이 함수를 사용하면 사람들이 안 좋아할 것이다. 이 함수는 악센트 붙은 글자를 무작위한 문자로 변환한다.

## SEE ALSO

<tt>[[isascii(3)]]</tt>, <tt>[[tolower(3)]]</tt>, <tt>[[toupper(3)]]</tt>

----

2021-03-22
