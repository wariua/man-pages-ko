## NAME

strtoimax, strtoumax - 문자열을 정수로 변환하기

## SYNOPSIS

```c
#include <inttypes.h>

intmax_t strtoimax(const char *restrict nptr, char **restrict endptr,
                   int base);
uintmax_t strtoumax(const char *restrict nptr, char **restrict endptr,
                   int base);
```

## DESCRIPTION

이 함수들은 <tt>[[strtol(3)]]</tt> 및 <tt>[[strtoul(3)]]</tt>과 같되, 각각 `intmax_t` 및 `uintmax_t` 타입 값을 반환한다.

## RETURN VALUE

성공 시 변환한 값을 반환한다. 변환할 것이 없으면 0을 반환한다. 오버플로나 언더플로 시에는 `INTMAX_MAX`나 `INTMAX_MIN`, 또는 `UINTMAX_MAX`를 반환하며 `errno`를 `ERANGE`로 설정한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strtoimax()`, `strtoumax()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C99.

## SEE ALSO

`imaxabs(3)`, `imaxdiv(3)`, <tt>[[strtol(3)]]</tt>, <tt>[[strtoul(3)]]</tt>, `wcstoimax(3)`

----

2021-03-22
