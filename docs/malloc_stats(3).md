## NAME

malloc_stats - 메모리 할당 통계 찍기

## SYNOPSIS

```c
#include <malloc.h>

void malloc_stats(void);
```

## DESCRIPTION

`malloc_stats()` 함수는 <tt>[[malloc(3)]]</tt> 및 관련 함수들이 할당한 메모리에 대한 통계를 (표준 오류로) 찍는다. 각 아레나(할당 영역)별로 할당한 메모리 총량과 사용 중인 할당들이 소모하는 바이트 총수를 찍는다. (이 두 값은 <tt>[[mallinfo(3)]]</tt>로 얻는 `arena` 및 `uordblks` 필드에 대응한다.) 추가로 모든 아레나들에 대한 두 통계치의 합을 찍고, <tt>[[mmap(2)]]</tt>으로 동시에 최대로 할당했던 블록 및 바이트 수를 찍는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `malloc_stats()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 GNU 확장이다.

## NOTES

<tt>[[mallinfo(3)]]</tt>를 이용하면 메인 아레나에서의 메모리 할당에 대한 더 자세한 정보를 얻을 수 있다.

## SEE ALSO

<tt>[[mmap(2)]]</tt>, <tt>[[mallinfo(3)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[malloc_info(3)]]</tt>, <tt>[[mallopt(3)]]</tt>

----

2017-09-15
