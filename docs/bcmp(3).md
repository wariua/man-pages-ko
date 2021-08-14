## NAME

bcmp - 바이트 열 비교하기

## SYNOPSIS

```c
#include <strings.h>

int bcmp(const void *s1, const void *s2, size_t n);
```

## DESCRIPTION

`bcmp()` 함수는 각각 길이가 `n`인 바이트 열 `s1`과 `s2`를 비교한다. 같으면, 특히 `n`이 0이면 `bcmp()`가 0을 반환한다. 아니면 0 아닌 결과를 반환한다.

## RETURN VALUE

`bcmp()` 함수는 바이트 열이 같으면 0을 반환한다. 아니면 0 아닌 결과를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `bcmp()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.3BSD. 이 함수는 제거 예정이다. (POSIX.1-2001에서 LEGACY로 표시됨.) 새 프로그램에서는 <tt>[[memcmp(3)]]</tt>를 사용하라. POSIX.1-2008에서 `bcmp()` 명세를 제거하였다.

## SEE ALSO

<tt>[[bstring(3)]]</tt>, <tt>[[memcmp(3)]]</tt>, <tt>[[strcasecmp(3)]]</tt>, <tt>[[strcmp(3)]]</tt>, <tt>[[strcoll(3)]]</tt>, <tt>[[strncasecmp(3)]]</tt>, <tt>[[strncmp(3)]]</tt>

----

2021-03-22
