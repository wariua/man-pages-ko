## NAME

bcmp, bcopy, bzero, memccpy, memchr, memcmp, memcpy, memfrob, memmem, memmove, memset - 바이트 열 연산

## SYNOPSIS

```c
#include <string.h>

int bcmp(const void *s1, const void *s2, size_t n);

void bcopy(const void *src, void *dest, size_t n);

void bzero(void *s, size_t n);

void *memccpy(void *dest, const void *src, int c, size_t n);

void *memchr(const void *s, int c, size_t n);

int memcmp(const void *s1, const void *s2, size_t n);

void *memcpy(void *dest, const void *src, size_t n);

void *memfrob(void *s, size_t n);

void *memmem(const void *haystack, size_t haystacklen,
             const void *needle, size_t needlelen);

void *memmove(void *dest, const void *src, size_t n);

void *memset(void *s, int c, size_t n);
```

## DESCRIPTION

바이트 열 함수들은 꼭 널 종료는 아닌 문자열(바이트 배열)에 대해 동작을 수행한다. 각 함수에 대한 설명은 개별 맨 페이지를 보라.

## NOTES

`bcmp()`, `bcopy()`, `bzero()` 함수는 구식이다. 대신 `memcmp()`, `memcpy()`, `memset()`을 사용하라.

## SEE ALSO

`bcmp(3)`, `bcopy(3)`, <tt>[[bzero(3)]]</tt>, <tt>[[memccpy(3)]]</tt>, <tt>[[memchr(3)]]</tt>, `memcmp(3)`, `memcpy(3)`, <tt>[[memfrob(3)]]</tt>, <tt>[[memmem(3)]]</tt>, `memmove(3)`, `memset(3)`, <tt>[[string(3)]]</tt>

----

2021-03-22
