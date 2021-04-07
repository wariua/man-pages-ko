## NAME

malloc_trim - 힙에서 노는 메모리 해제하기

## SYNOPSIS

```c
#include <malloc.h>

int malloc_trim(size_t pad);
```

## DESCRIPTION

`malloc_trim()` 함수는 (적절한 인자로 <tt>[[sbrk(2)]]</tt>나 <tt>[[madvise(2)]]</tt>를 호출해서) 힙에서 노는 메모리를 해제하려고 시도한다.

`pad` 인자는 힙 상단에 없애지 않고 남겨둘 빈 공간의 양을 나타낸다. 이 인자가 0이면 힙 상단에 최소한의 메모리만 (즉 한 페이지 이하만) 유지한다. 향후 <tt>[[sbrk(2)]]</tt>로 힙을 확장하지 않고도 할당을 할 수 있도록 하기 위해 0 아닌 인자를 사용해 힙 상단에 약간의 잔여 공간을 유지할 수도 있다.

## RETURN VALUE

`malloc_trim()` 함수는 실제로 메모리를 시스템으로 해제했으면 1을 반환하고 메모리를 더 해제할 수 없었으면 0을 반환한다.

## ERRORS

어떤 오류도 정의되어 있지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `malloc_trim()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 GNU 확장이다.

## NOTES

<tt>[[free(3)]]</tt>가 특정 상황에서 자동으로 이 함수를 호출한다. <tt>[[mallopt(3)]]</tt>의 `M_TOP_PAD` 및 `M_TRIM_THRESHOLD` 논의를 보라.

(<tt>[[sbrk(2)]]</tt>를 쓰는) 메인 힙에서만 `pad` 인자를 이용한다. 스레드 힙에선 무시한다.

glibc 2.8부터 이 함수는 모든 아레나에서, 그리고 유휴 페이지로만 된 모든 청크에서 메모리를 해제한다.

glibc 2.8 전에서 이 함수는 메인 아레나의 힙 상단에서만 메모리를 해제했다.

## SEE ALSO

<tt>[[sbrk(2)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[mallopt(3)]]</tt>

----

2019-05-09
