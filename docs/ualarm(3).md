## NAME

ualarm - 주어진 마이크로초 후로 시그널 예약하기

## SYNOPSIS

```c
#include <unistd.h>

useconds_t ualarm(useconds_t usecs, useconds_t interval);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>ualarm()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.12부터:</dt>
 <dd>
<code>(_XOPEN_SOURCE >= 500) && ! (_POSIX_C_SOURCE >= 200809L)</code><br>
<code>    || /* glibc 2.19부터: */ _DEFAULT_SOURCE</code><br>
<code>    || /* glibc 버전 <= 2.19: */ _BSD_SOURCE</code>
 </dd>
 <dt>glibc 2.12 전:</dt>
 <dd><code>_BSD_SOURCE || _XOPEN_SOURCE >= 500</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`ualarm()` 함수는 `usecs` 마이크로초 (이상) 후에 호출 프로세스에게 `SIGALRM` 시그널이 가게 한다. 어떤 시스템 활동이나 호출 처리에 소모된 시간에 의해, 또는 시스템 타이머 정밀도에 따라 지연 시간이 살짝 길어질 수도 있다.

잡거나 무시하지 않으면 `SIGALRM` 시그널은 프로세스를 종료시킨다.

`interval` 인자가 0이 아니면 첫 번째 시그널 후에도 `interval` 마이크로초마다 `SIGALRM` 시그널을 계속 보낸다.

## RETURN VALUE

이 함수는 앞서 설정돼 있던 알람에 대해 남은 마이크로초 수를 반환한다. 미처리 알람이 없으면 0을 반환한다.

## ERRORS

<dl>
<dt><code>EINTR</code></dt>
<dd>시그널에 의해 중단됨. <tt>[[signal(7)]]</tt> 참고.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>usec</code>나 <code>interval</code>이 1000000 미만이 아니다. (그게 오류라고 보는 시스템에서.)</dd>
</dl>

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `ualarm()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.3BSD, POSIX.1-2001. POSIX.1-2001에서 이 함수를 구식으로 표시했다. POSIX.1-2008에서 `ualarm()` 명세를 제거했다. 4.3BSD, SUSv2, POSIX에서는 어떤 오류도 규정하고 있지 않다.

## NOTES

POSIX.1-2001에서는 `usecs` 인자가 0일 때 어떻게 되는지 명세하고 있지 않다. 리눅스에서 (그리고 아마 대다수의 다른 시스템에서) 그 효과는 미처리 알람이 있으면 취소하는 것이다.

`useconds_t` 타입은 [0,1000000] 범위의 정수를 담을 수 있는 부호 없는 정수 타입이다. 원래 BSD 구현에서, 그리고 glibc 버전 2.1 전에서는 `ualarm()` 인자들에 `unsigned int` 타입을 썼다. 프로그램에서 `useconds_t`를 명시적으로 쓰지 않는 게 이식성 측면에서 나을 것이다.

이 함수와 <tt>[[alarm(2)]]</tt>, <tt>[[sleep(3)]]</tt>, <tt>[[nanosleep(2)]]</tt>, <tt>[[setitimer(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[timer_delete(2)]]</tt>, <tt>[[timer_getoverrun(2)]]</tt>, <tt>[[timer_gettime(2)]]</tt>, <tt>[[timer_settime(2)]]</tt>, <tt>[[usleep(3)]]</tt> 등의 여타 타이머 함수들과의 상호작용은 명세돼 있지 않다.

이 함수는 구식이다. 대신 <tt>[[setitimer(2)]]</tt>나 POSIX 간격 타이머(<tt>[[timer_create(2)]]</tt> 등)를 써야 한다.

## SEE ALSO

<tt>[[alarm(2)]]</tt>, <tt>[[getitimer(2)]]</tt>, <tt>[[nanosleep(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[setitimer(2)]]</tt>, <tt>[[usleep(3)]]</tt>, <tt>[[time(7)]]</tt>

----

2017-09-15
