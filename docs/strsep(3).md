## NAME

strsep - 문자열에서 토큰 추출하기

## SYNOPSIS

```c
#include <string.h>

char *strsep(char **restrict stringp, const char *restrict delim);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`strsep()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE`

## DESCRIPTION

`*stringp`가 NULL이면 `strsep()` 함수는 NULL을 반환하며 다른 아무것도 하지 않는다. 그 외의 경우에 이 함수는 문자열 `*stringp` 내에서 문자열 `delim`의 바이트들 중 하나로 구분되는 첫 번째 토큰을 찾는다. 구분자를 널 바이트(`'\0'`)로 덮어 써서 토큰 끝을 표시하며 그 토큰 다음을 가리키도록 `*stringp`를 갱신한다. 구분자를 찾지 못한 경우에는 문자열 `*stringp` 전체가 토큰이 되며 `*stringp`를 NULL로 만든다.

## RETURN VALUE

`strsep()` 함수는 토큰에 대한 포인터를 반환한다. 즉, `*stringp`의 원래 값을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strsep()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.4BSD.

## NOTES

`strsep()` 함수는 빈 필드를 다루지 못하는 <tt>[[strtok(3)]]</tt>을 대체하기 위해 도입되었다. 하지만 <tt>[[strtok(3)]]</tt>은 C89/C99를 준수하기 때문에 이식성이 더 좋다.

## BUGS

이 함수를 사용할 때는 조심해야 한다. 꼭 사용하겠다면 다음에 유념해야 한다.

* 이 함수는 첫 번째 인자를 변경한다.

* 이 함수는 상수 문자열에 사용할 수 없다. 

* 구분 문자의 원래 값을 알 수 없게 된다.

## SEE ALSO

`index(3)`, <tt>[[memchr(3)]]</tt>, `rindex(3)`, `strchr(3)`, <tt>[[string(3)]]</tt>, `strpbrk(3)`, `strspn(3)`, `strstr(3)`, <tt>[[strtok(3)]]</tt>

----

2021-03-22
