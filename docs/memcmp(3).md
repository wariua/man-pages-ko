## NAME

memcmp - 메모리 영역 비교하기

## SYNOPSIS

```c
#include <string.h>

int memcmp(const void *s1, const void *s2, size_t n);
```

## DESCRIPTION

`memcmp()` 함수는 메모리 영역 `s1`과 `s2`의 처음 `n` 개 바이트를 (각각 `unsigned char`로 해석해서) 비교한다.

## RETURN VALUE

`memcmp()` 함수는 `s1`의 처음 `n` 바이트가 `s2`의 처음 `n` 바이트보다 작거나, 일치하거나, 큰 경우에 0보다 작거나, 같거나, 큰 정수를 반환한다.

0 아닌 반환 값의 부호는 `s1`과 `s2`에서 첫 번째로 다른 (`unsigned char` 타입으로 해석한) 바이트 쌍의 값 차이의 부호에 의해 정해진다.

`n`이 0이면 반환 값이 0이다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `memcmp()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

암호학적 비밀값처럼 보안에 중요한 데이터를 비교하는 데 `memcmp()`를 쓰지 말아야 한다. 같은 바이트 수에 따라 비교에 필요한 CPU 시간이 달라지기 때문이다. 대신 고정된 시간에 비교를 수행하는 함수가 필요하다. 일부 운영 체제에서 그런 함수를 제공하지만 (예: NetBSD의 `consttime_memequal()`) POSIX에는 그런 함수가 명세돼 있지 않다. 리눅스에서 그런 함수를 직접 구현해야 할 수도 있다.

## SEE ALSO

<tt>[[bcmp(3)]]</tt>, <tt>[[bstring(3)]]</tt>, <tt>[[strcasecmp(3)]]</tt>, <tt>[[strcmp(3)]]</tt>, <tt>[[strcoll(3)]]</tt>, <tt>[[strncasecmp(3)]]</tt>, <tt>[[strncmp(3)]]</tt>, `wmemcmp(3)`

----

2021-03-22
