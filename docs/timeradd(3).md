## NAME

timeradd, timersub, timercmp, timerclear, timerisset - timeval 연산

## SYNOPSIS

```c
#include <sys/time.h>

void timeradd(struct timeval *a, struct timeval *b,
              struct timeval *res);

void timersub(struct timeval *a, struct timeval *b,
              struct timeval *res);

void timerclear(struct timeval *tvp);

int timerisset(struct timeval *tvp);

int timercmp(struct timeval *a, struct timeval *b, CMP);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt>위의 모든 함수:</dt>
<dd>
 <dl>
 <dt>glibc 2.19부터:</dt>
 <dd><code>_DEFAULT_SOURCE</code></dd>
 <dt>glibc 2.19 및 이전:</dt>
 <dd><code>_BSD_SOURCE</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`<sys/time.h>`에 다음처럼 정의돼 있는 `timeval` 구조체에 대해 동작하는 매크로들이다.

```c
struct timeval {
    time_t      tv_sec;     /* 초 */
    suseconds_t tv_usec;    /* 마이크로초 */
};
```

`timeradd()`는 `a`와 `b`의 시간 값을 더해서 그 합을 `res`가 가리키는 `timeval`에 넣는다. `res->tv_usec`이 0에서 999,999 범위의 값을 가지도록 결과가 정규화 된다.

`timersub()`는 `a`의 시간 값에서 `b`의 시간 값을 빼서 그 결과를 `res`가 가리키는 `timeval`에 넣는다. `res->tv_usec`이 0에서 999,999 범위의 값을 가지도록 결과가 정규화 된다.

`timerclear()`는 `tvp`가 가리키는 `timeval` 구조체를 0으로 채워서 에포크 1970-01-01 00:00:00 +0000 (UTC)를 나타내도록 한다.

`timerisset()`은 `tvp`가 가리키는 `timeval` 구조체의 어느 필드라도 0 아닌 값을 담고 있으면 참(0 아닌 값)을 반환한다.

`timercmp()`는 비교 연산자 `CMP`를 써서 `a`와 `b`의 시간 값을 비교해서 그 결과에 따라 참(0 아닌 값)이나 거짓(0)을 반환한다. 일부 시스템(리눅스/glibc는 아님)에서는 `timercmp()` 구현이 불완전해서 `CMP`가 `>=`, `<=`, `==`이면 제대로 동작하지 않는다. 이식 가능한 응용에서는 대신 다음 방식을 쓸 수 있다.

```c
!timercmp(..., <)
!timercmp(..., >)
!timercmp(..., !=)
```

## RETURN VALUE

`timerisset()`과 `timercmp()`는 참(0 아닌 값) 또는 거짓(0)을 반환한다.

## ERRORS

어떤 오류도 정의되어 있지 않다.

## CONFORMING TO

POSIX.1에는 없다. 대부분의 BSD 파생 시스템에 존재한다.

## SEE ALSO

<tt>[[gettimeofday(2)]]</tt>, <tt>[[time(7)]]</tt>

----

2017-09-15
