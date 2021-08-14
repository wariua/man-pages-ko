## NAME

strstr, strcasestr - 하위 문자열 찾기

## SYNOPSIS

```c
#include <string.h>

char *strstr(const char *haystack, const char *needle);

#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <string.h>

char *strcasestr(const char *haystack, const char *needle);
```

## DESCRIPTION

`strstr()` 함수는 문자열 `haystack`에서 하위 문자열 `needle`의 첫 번째 위치를 찾는다. 종료 널 바이트('\0')는 비교하지 않는다.

`strcasestr()` 함수는 `strstr()`과 같되, 두 인자의 대소문자를 무시한다.

## RETURN VALUE

이 함수들은 찾은 하위 문자열 시작점에 대한 포인터를 반환한다. 하위 문자열을 찾지 못하면 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strstr()` | 스레드 안전성 | MT-Safe |
| `strcasestr()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

`strstr()`: POSIX.1-2001, POSIX.1-2008, C89, C99.

`strcasestr()` 함수는 비표준 확장이다.

## SEE ALSO

<tt>[[index(3)]]</tt>, <tt>[[memchr(3)]]</tt>, <tt>[[memmem(3)]]</tt>, <tt>[[rindex(3)]]</tt>, <tt>[[strcasecmp(3)]]</tt>, <tt>[[strchr(3)]]</tt>, <tt>[[string(3)]]</tt>, <tt>[[strpbrk(3)]]</tt>, <tt>[[strsep(3)]]</tt>, <tt>[[strspn(3)]]</tt>, <tt>[[strtok(3)]]</tt>, `wcsstr(3)`

----

2021-03-22
