## NAME

strpbrk - 문자열에서 바이트들 중 아무거나 찾기

## SYNOPSIS

```c
#include <string.h>

char *strpbrk(const char *s, const char *accept);
```

## DESCRIPTION

`strpbrk()` 함수는 문자열 `accept`에 있는 바이트들 중 하나가 문자열 `s`에서 처음 등장하는 위치를 찾는다.

## RETURN VALUE

`strpbrk()` 함수는 `accept`의 바이트들 중 하나와 일치하는 `s`의 바이트에 대한 포인터를 반환한다. 그런 바이트를 찾지 못하면 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strpbrk()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[index(3)]]</tt>, <tt>[[memchr(3)]]</tt>, <tt>[[rindex(3)]]</tt>, <tt>[[strchr(3)]]</tt>, <tt>[[string(3)]]</tt>, <tt>[[strsep(3)]]</tt>, <tt>[[strspn(3)]]</tt>, <tt>[[strstr(3)]]</tt>, <tt>[[strtok(3)]]</tt>, `wcspbrk(3)`

----

2021-03-22
