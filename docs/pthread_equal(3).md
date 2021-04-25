## NAME

pthread_equal - 스레드 ID 비교하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_equal(pthread_t t1, pthread_t t2);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_equal()` 함수는 두 스레드 식별자를 비교한다.

## RETURN VALUE

두 스레드 ID가 동일하면 `pthread_equal()`이 0 아닌 값을 반환한다. 그 외의 경우에는 0을 반환한다.

## ERRORS

이 함수는 항상 성공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_equal()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`pthread_equal()` 함수가 필요한 이유는 스레드 ID를 불투명한 것으로 보아야 하기 때문이다. 응용에서 두 `pthread_t` 값을 직접 비교할 수 있는 이식성 있는 방법이 없다.

## SEE ALSO

<tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_self(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
