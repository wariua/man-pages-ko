## NAME

pthread_self - 호출 스레드의 ID 얻기

## SYNOPSIS

```c
#include <pthread.h>

pthread_t pthread_self(void);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_self()` 함수는 호출 스레드의 ID를 반환한다. 이 스레드를 생성할 때 <tt>[[pthread_create(3)]]</tt> 호출의 `*thread`로 반환되었던 것과 같은 값이다.

## RETURN VALUE

이 함수는 항상 성공하며 호출 스레드의 ID를 반환한다.

## ERRORS

이 함수는 항상 성공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_self()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

POSIX.1에서는 스레드 ID를 나타내는 데 사용할 타입 선정에 있어 구현체에게 폭넓은 자유를 허용한다. 예를 들어 산술 타입을 이용한 표현이나 구조체를 이용한 표현 어느 쪽도 허용된다. 따라서 `pthread_t` 타입의 변수들을 C의 등호 연산자(`==`)를 이용해 이식성 있게 비교할 수 없다. 대신 <tt>[[pthread_equal(3)]]</tt>을 사용해야 한다.

스레드 식별자는 불투명한 것으로 보아야 한다. 스레드 ID를 pthreads 호출들 외에서 사용하려는 모든 시도는 이식성이 없으며 명세되지 않은 결과로 이어질 수 있다.

스레드 ID는 프로세스 내에서만 유일함이 보장된다. 종료된 스레드가 합류되거나 분리된 스레드가 종료된 후에는 스레드 ID가 재사용될 수도 있다.

`pthread_self()`가 반환하는 스레드 ID는 <tt>[[gettid(2)]]</tt> 호출이 반환하는 커널 스레드 ID와 같은 것이 아니다.

## SEE ALSO

<tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_equal(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
