## NAME

pthread_testcancel - 미처리 취소 요청 전달 요청하기

## SYNOPSIS

```c
#include <pthread.h>

void pthread_testcancel(void);
```

`-pthread`로 링크.

## DESCRIPTION

`pthread_testcancel()` 호출은 호출 스레드 내에 취소점을 만들어서 취소점 없는 코드를 실행 중인 스레드가 취소 요청에 반응할 수 있게 한다.

취소 가능성이 (<tt>[[pthread_setcancelstate(3)]]</tt>로) 비활성화되어 있거나 미처리 취소 요청이 없으면 `pthread_testcancel()` 호출의 효과가 없다.

## RETURN VALUE

이 함수는 값을 반환하지 않는다. 이 함수 호출의 결과로 호출 스레드가 취소되는 경우에는 함수가 반환하지 않는다.

## ERRORS

이 함수는 항상 성공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_testcancel()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## EXAMPLE

<tt>[[pthread_cleanup_push(3)]]</tt> 참고.

## SEE ALSO

<tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_cleanup_push(3)]]</tt>, <tt>[[pthread_setcancelstate(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
