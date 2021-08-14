## NAME

strcoll - 현재 로캘로 두 문자열 비교하기

## SYNOPSIS

```c
#include <string.h>

int strcoll(const char *s1, const char *s2);
```

## DESCRIPTION

`strcoll()` 함수는 두 문자열 `s1`과 `s2`를 비교한다. `s1`이 `s2`보다 작거나, 일치하거나, 큰 경우에 0보다 작거나, 같거나, 큰 정수를 반환한다. 프로그램의 `LC_COLLATE` 부문 현재 로캘에 맞게 해석한 문자열을 가지고 비교한다.

## RETURN VALUE

`strcoll()`은 현재 로캘에 맞게 해석했을 때 `s1`이 `s2`보다 작거나, 일치하거나, 큰 경우에 0보다 작거나, 같거나, 큰 정수를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strcoll()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

`POSIX`나 `C` 로캘에서 `strcoll()`은 <tt>[[strcmp(3)]]</tt>와 동등하다.

## SEE ALSO

<tt>[[bcmp(3)]]</tt>, <tt>[[memcmp(3)]]</tt>, <tt>[[setlocale(3)]]</tt>, <tt>[[strcasecmp(3)]]</tt>, <tt>[[strcmp(3)]]</tt>, <tt>[[string(3)]]</tt>, <tt>[[strxfrm(3)]]</tt>

----

2021-03-22
