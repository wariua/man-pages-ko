## NAME

atoi, atol, atoll - 문자열을 정수로 변환하기

## SYNOPSIS

```c
#include <stdlib.h>

int atoi(const char *nptr);
long atol(const char *nptr);
long long atoll(const char *nptr);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`atoll()`:
:   `_ISOC99_SOURCE`<br>
    `    || /* glibc 버전 <= 2.19: */ _BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

`atoi()` 함수는 `nptr`이 가리키는 문자열의 처음 부분을 `int`로 변환한다. 다음과 동작이 같되, `atoi()`에서는 오류를 감지하지 않는다.

```c
strtol(nptr, NULL, 10);
```

`atol()`과 `atoll()` 함수는 `atoi()`와 동일하게 동작하되 문자열의 처음 부분을 반환 타입인 `long`이나 `long long`으로 변환한다.

## RETURN VALUE

변환한 값.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `atoi()`, `atol()`, `atoll()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C99, SVr4, 4.3BSD. C89와 POSIX.1-1996에는 함수 `atoi()`와 `atol()`만 있다.

## NOTES

리눅스 libc에서 `atoll()`의 구식 이름인 `atoq()`를 제공했다. glibc에서는 `atoq()`를 제공하지 않는다.

## SEE ALSO

<tt>[[atof(3)]]</tt>, <tt>[[strtod(3)]]</tt>, <tt>[[strtol(3)]]</tt>, <tt>[[strtoul(3)]]</tt>

----

2016-03-15
