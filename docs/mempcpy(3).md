## NAME

mempcpy, wmempcpy - 메모리 영역 복사하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <string.h>

void *mempcpy(void *restrict dest, const void *restrict src, size_t n);
```

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <wchar.h>

wchar_t *wmempcpy(wchar_t *restrict dest, const wchar_t *restrict src,
                  size_t n);
```

## DESCRIPTION

`mempcpy()` 함수는 <tt>[[memcpy(3)]]</tt> 함수와 거의 동일하다. `src`에서 시작하는 객체에서 `dest`가 가리키는 객체로 `n`개 바이트를 복사한다. 그런데 `dest` 값을 반환하는 게 아니라 마지막으로 써넣은 바이트 다음 바이트에 대한 포인터를 반환한다.

연속하는 메모리 위치에 여러 객체들을 복사해야 하는 경우에 이 함수가 유용하다.

`wmempcpy()` 함수는 위와 동일하되 `wchar_t` 타입 인자들을 받아서 `n`개 확장 문자를 복사한다.

## RETURN VALUE

`dest` + `n`.

## VERSIONS

glibc 버전 2.1에서 `mempcpy()`가 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mempcpy()`, `wmempcpy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 GNU 확장이다.

## EXAMPLES

```c
void *
combine(void *o1, size_t s1, void *o2, size_t s2)
{
    void *result = malloc(s1 + s2);
    if (result != NULL)
        mempcpy(mempcpy(result, o1, s1), o2, s2);
    return result;
}
```

## SEE ALSO

<tt>[[memccpy(3)]]</tt>, <tt>[[memcpy(3)]]</tt>, <tt>[[memmove(3)]]</tt>, <tt>[[wmemcpy(3)]]</tt>

----

2021-03-22
