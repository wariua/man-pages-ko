## NAME

strxfrm - 문자열 변형

## SYNOPSIS

```c
#include <string.h>

size_t strxfrm(char *restrict dest, const char *restrict src,
               size_t n);
```

## DESCRIPTION

`strxfrm()` 함수는 두 문자열을 `strxfrm()`으로 변형한 후의 <tt>[[strcmp(3)]]</tt> 결과가 변형 전 두 문자열에 대한 <tt>[[strcoll(3)]]</tt> 결과와 같도록 `src`를 변형한다. 변형된 문자열의 처음 `n` 바이트가 `dest`로 들어간다. 프로그램의 `LC_COLLATE` 부문 현재 로캘에 따라 변형한다. (<tt>[[setlocale(3)]]</tt> 참고.)

## RETURN VALUE

`strxfrm()` 함수는 변형된 문자열을 `dest`에 저장하기 위해 필요한 종료 널 바이트('\0')를 뺀 바이트 수를 반환한다. 반환된 값이 `n`이거나 그보다 큰 경우 `dest`의 내용은 정해져 있지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strxfrm()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[bcmp(3)]]</tt>, <tt>[[memcmp(3)]]</tt>, <tt>[[setlocale(3)]]</tt>, <tt>[[strcasecmp(3)]]</tt>, <tt>[[strcmp(3)]]</tt>, <tt>[[strcoll(3)]]</tt>, <tt>[[string(3)]]</tt>

----

2021-03-22
