## NAME

malloc_usable_size - 힙에서 할당한 메모리 블록의 크기 얻기

## SYNOPSIS

```c
#include <malloc.h>

size_t malloc_usable_size(void *ptr);
```

## DESCRIPTION

`malloc_usable_size()` 함수는 `ptr`이 가리키는 블록 내에서 사용 가능한 바이트 수를 반환한다. `ptr`은 <tt>[[malloc(3)]]</tt>이나 관련 함수로 할당한 메모리 블록에 대한 포인터이다.

## RETURN VALUE

`malloc_usable_size()`는 `ptr`이 가리키는 할당 메모리 블록에서 사용 가능한 바이트 수를 반환한다. `ptr`이 NULL이면 0을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `malloc_usable_size()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 GNU 확장이다.

## NOTES

정렬과 최소 크기 제한 때문에 `malloc_usable_size()`가 반환하는 값이 할당 요청 크기보다 클 수도 있다. 응용이 초과 바이트들에 쓰기를 해도 부작용이 없기는 하지만 좋은 프로그래밍 방식이 아니다. 할당에서 초과 바이트의 수는 기반 구현에 따라 달라진다.

이 함수의 주된 용도는 디버깅과 내성(introspection)이다.

## SEE ALSO

<tt>[[malloc(3)]]</tt>

----

2017-09-15
