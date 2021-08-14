## NAME

index, rindex - 문자열에서 문자 찾기

## SYNOPSIS

```c
#include <strings.h>

char *index(const char *s, int c);
char *rindex(const char *s, int c);
```

## DESCRIPTION

`index()` 함수는 문자열 `s`에서 문자 `c`의 첫 번째 위치에 대한 포인터를 반환한다.

`rindex()` 함수는 문자열 `s`에서 문자 `c`의 마지막 위치에 대한 포인터를 반환한다.

종료 널 바이트('\0')를 문자열의 일부로 본다.

## RETURN VALUE

`index()`와 `rindex()` 함수는 일치한 문자에 대한 포인터를 반환한다. 그 문자를 찾지 못하면 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `index()`, `rindex()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.3BSD. POSIX.1-2001에서 LEGACY로 표시됨. POSIX.1-2008에서 `index()`와 `rindex()`의 명세를 제거하였으며 대신 <tt>[[strchr(3)]]</tt>과 <tt>[[strrchr(3)]]</tt>을 권고한다.

## SEE ALSO

<tt>[[memchr(3)]]</tt>, <tt>[[strchr(3)]]</tt>, <tt>[[string(3)]]</tt>, <tt>[[strpbrk(3)]]</tt>, <tt>[[strrchr(3)]]</tt>, <tt>[[strsep(3)]]</tt>, <tt>[[strspn(3)]]</tt>, <tt>[[strstr(3)]]</tt>, <tt>[[strtok(3)]]</tt>

----

2021-03-22
