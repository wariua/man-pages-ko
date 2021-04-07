## NAME

atof - 문자열을 double로 변환하기

## SYNOPSIS

```c
#include <stdlib.h>

double atof(const char *nptr);
```

## DESCRIPTION

`atof()` 함수는 `nptr`이 가리키는 문자열의 처음 부분을 `double`로 변환한다. 다음과 동작이 같되, `atof()`에서는 오류를 감지하지 않는다.

```c
strtod(nptr, NULL);
```

## RETURN VALUE

변환한 값.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `atof()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[atoi(3)]]</tt>, <tt>[[atol(3)]]</tt>, <tt>[[strfromd(3)]]</tt>, <tt>[[strtod(3)]]</tt>, <tt>[[strtol(3)]]</tt>, <tt>[[strtoul(3)]]</tt>

----

2016-12-12
