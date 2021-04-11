## NAME

lfind, lsearch - 배열 순차 탐색

## SYNOPSIS

```c
#include <search.h>

void *lfind(const void *key, const void *base, size_t *nmemb,
         size_t size, int(*compar)(const void *, const void *));

void *lsearch(const void *key, void *base, size_t *nmemb,
         size_t size, int(*compar)(const void *, const void *));
```

## DESCRIPTION

`lfind()`와 `lsearch()`는 각 `size` 바이트인 `*nmemb` 개 항목의 배열 `base`에서 `key`에 대해 순차 탐색을 수행한다. `compar`가 가리키는 비교 함수에는 두 인자가 있어서 차례로 `key` 객체와 배열 항목을 가리키게 돼 있으며, `key` 객체가 배열 항목과 일치하면 0을 반환하고 아니면 0 아닌 값을 반환한다.

`lsearch()`에서 일치 항목을 찾지 못하면 테이블 끝에 `key` 객체를 삽입하고 `*nmemb`를 증가시킨다. 일치 항목이 확실히 존재하든지 아니면 여유 공간이 있든지 해야 한다.

## RETURN VALUE

`lfind()`는 일치하는 배열 항목에 대한 포인터를 반환하며 일치 항목이 없으면 NULL을 반환한다. `lsearch()`는 일치하는 배열 항목에 대한 포인터를 반환하며 일치 항목이 없으면 새로 추가한 항목에 대한 포인터를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `lfind()`, `lsearch()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD. libc-4.6.27부터 libc에 있음.

## BUGS

이름이 적절치 않다.

## SEE ALSO

<tt>[[bsearch(3)]]</tt>, <tt>[[hsearch(3)]]</tt>, <tt>[[tsearch(3)]]</tt>

----

2017-09-15
