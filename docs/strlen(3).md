## NAME

strlen - 문자열의 길이 계산하기

## SYNOPSIS

```c
#include <string.h>

size_t strlen(const char *s);
```

## DESCRIPTION

`strlen()` 함수는 `s`가 가리키는 문자열의 길이를 계산한다. 종료 널 바이트('\0')는 제외한다.

## RETURN VALUE

`strlen()` 함수는 `s`가 가리키는 문자열의 바이트 수를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strlen()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, C11, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[string(3)]]</tt>, <tt>[[strnlen(3)]]</tt>, `wcslen(3)`, `wcsnlen(3)`

----

2021-03-22
