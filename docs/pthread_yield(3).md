## NAME

pthread_yield - 프로세서 양보하기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <pthread.h>

int pthread_yield(void);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_yield()`는 호출 스레드가 CPU를 포기하게 한다. 스레드가 자기 고정 우선순위에 대한 큐의 끝으로 가고 다른 스레드가 스케줄 된다. 더 자세한 내용은 <tt>[[sched_yield(2)]]</tt>를 보라.

## RETURN VALUE

성공 시 `pthread_yield()`는 0을 반환한다. 오류 시 오류 번호를 반환한다.

## ERRORS

리눅스에서 이 호출은 항상 성공한다. (그렇기는 하지만 이식 가능하고 미래를 대비하는 응용에서는 가능한 오류 반환을 처리해야 할 것이다.)

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_yield()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 호출은 비표준이지만 여러 다른 시스템에 존재한다. 표준화된 <tt>[[sched_yield(2)]]</tt>를 대신 사용하라.

## NOTES

리눅스에서 이 함수는 <tt>[[sched_yield(2)]]</tt> 호출로 구현되어 있다.

`pthread_yield()`는 실시간 스케줄링 정책(즉 `SCHED_FIFO`나 `SCHED_RR`)에 사용하기 위한 것이다. `SCHED_OTHER` 같은 비결정적 스케줄링 정책에 `pthread_yield()`를 사용하는 것은 명세되어 있지 않으며 응용 설계에 문제가 있다는 뜻일 가능성이 높다.

## SEE ALSO

<tt>[[sched_yield(2)]]</tt>, <tt>[[pthread(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2017-11-26
