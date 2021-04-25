## NAME

usleep - 마이크로초 단위 시간 동안 실행을 멈추기

## SYNOPSIS

```c
#include <unistd.h>

int usleep(useconds_t usec);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`usleep()`
:   glibc 2.12부터:
    :   `(_XOPEN_SOURCE >= 500) && ! (_POSIX_C_SOURCE >= 200809L)`<br>
        `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
        `    || /* glibc <= 2.19: */ _BSD_SOURCE`
 
    glibc 2.12 전:
    :   `_BSD_SOURCE || _XOPEN_SOURCE >= 500`

## DESCRIPTION

`usleep()` 함수는 (적어도) `usec` 마이크로초 동안 호출 스레드의 실행을 멈춘다. 어떤 시스템 활동이나 호출 처리에 소모된 시간에 의해, 또는 시스템 타이머 정밀도에 따라 잠드는 시간이 살짝 길어질 수도 있다.

## RETURN VALUE

`usleep()` 함수는 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINTR`
:   시그널에 의해 중단됨. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `usec`이 1000000 이상이다. (그게 오류라고 보는 시스템에서.)

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `usleep()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.3BSD, POSIX.1-2001. POSIX.1-2001에서 이 함수를 구식으로 선언했다. 대신 <tt>[[nanosleep(2)]]</tt>을 사용하라. POSIX.1-2008에서 `usleep()` 명세를 제거했다.

원래 BSD 구현에서, 그리고 glibc 버전 2.2.2 전에서는 이 함수의 반환 타입이 `void`이다. POSIX 버전은 `int`를 반환하는데 glibc 2.2.2부터 쓰는 원형이기도 하다.

SUSv2 및 POSIX.1-2001에서는 `EINVAL` 오류만 기록돼 있다.

## NOTES

`useconds_t` 타입은 [0,1000000] 범위의 정수를 담을 수 있는 부호 없는 정수 타입이다. 프로그램에서 이 타입을 명시적으로 쓰지 않는 쪽이 이식성이 나을 것이다. 즉 다음과 같이 쓰면 된다.

```c
#include <unistd.h>
...
    unsigned int usecs;
...
    usleep(usecs);
```

이 함수와 `SIGALARM` 시그널과의 상호작용, 그리고 <tt>[[alarm(2)]]</tt>, <tt>[[sleep(3)]]</tt>, <tt>[[nanosleep(2)]]</tt>, <tt>[[setitimer(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[timer_delete(2)]]</tt>, <tt>[[timer_getoverrun(2)]]</tt>, <tt>[[timer_gettime(2)]]</tt>, <tt>[[timer_settime(2)]]</tt>, <tt>[[ualarm(3)]]</tt> 같은 여타 타이머 함수와의 상호작용은 명세돼 있지 않다.

## SEE ALSO

<tt>[[alarm(2)]]</tt>, <tt>[[getitimer(2)]]</tt>, <tt>[[nanosleep(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[setitimer(2)]]</tt>, <tt>[[sleep(3)]]</tt>, <tt>[[ualarm(3)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
