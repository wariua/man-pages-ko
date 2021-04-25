## NAME

memchr, memrchr, rawmemchr - 메모리에서 문자 탐색하기

## SYNOPSIS

```c
#include <string.h>

void *memchr(const void *s, int c, size_t n);
void *memrchr(const void *s, int c, size_t n);
void *rawmemchr(const void *s, int c);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`memrchr()`, `rawmemchr()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`memchr()` 함수는 `s`가 가리키는 메모리 영역의 처음 `n` 개 바이트에서 `c`의 첫 번째 인스턴스를 탐색한다. `s`가 가리키는 메모리 영역의 바이트들과 `c` 모두 `unsigned char`로 해석한다.

`memrchr()` 함수는 `memchr()`과 비슷하되 `s`가 가리키는 `n` 개 바이트의 시작부터 순방향이 아니라 끝부터 역방향으로 탐색한다.

`rawmemchr()` 함수는 `memchr()`과 비슷하되 `s`가 가리키는 위치에서 시작하는 메모리 영역 내의 어딘가에 `c`의 인스턴스가 있다고 상정한다. (즉, 그렇다는 것을 프로그래머가 확실히 알고 있다.) 그래서 `c`에 대한 최적화된 탐색(가령 탐색 범위 제한하는 카운트 인자 사용하지 않기)을 수행한다. `c`의 인스턴스를 찾지 못한 경우 그 결과는 예측 불가능이다. 다음 호출은 문자열의 종료용 널 바이트 위치를 찾는 빠른 방법이다.

```c
char *p = rawmemchr(s, '\0');
```

## RETURN VALUE

`memchr()` 및 `memrchr()` 함수는 일치하는 바이트에 대한 포인터를 반환한다. 주어진 메모리 영역 내에 그 문자가 없으면 NULL을 반환한다.

`rawmemchr()` 함수는 일치하는 바이트를 발견하면 그에 대한 포인터를 반환한다. 일치하는 바이트를 찾지 못하면 그 결과가 명세되어 있지 않다.

## VERSIONS

glibc 버전 2.1에서 `rawmemchr()`이 처음 등장했다.

glibc 버전 2.2에서 `memrchr()`이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `memchr()`, `memrchr()`, `rawmemchr()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`memchr()`: POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

`memrchr()` 함수는 GNU 확장이며 glibc 2.1.91부터 사용 가능하다.

`rawmemchr()` 함수는 GNU 확장이며 glibc 2.1부터 사용 가능하다.

## SEE ALSO

<tt>[[bstring(3)]]</tt>, <tt>[[ffs(3)]]</tt>, `index(3)`, <tt>[[memmem(3)]]</tt>, `rindex(3)`, `strchr(3)`, `strpbrk(3)`, `strrchr(3)`, <tt>[[strsep(3)]]</tt>, `strspn(3)`, `strstr(3)`, `wmemchr(3)`

----

2021-03-22
