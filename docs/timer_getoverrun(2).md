## NAME

timer_getoverrun - POSIX 프로세스별 타이머에 대해 초과 횟수 얻기

## SYNOPSIS

```c
#include <time.h>

int timer_getoverrun(timer_t timerid);
```

`-lrt`로 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>timer_getoverrun()</code>:</dt>
<dd><code>_POSIX_C_SOURCE >= 199309L</code></dd>
</dl>

## DESCRIPTION

`timer_getoverrun()`은 `timerid`가 가리키는 타이머에 대한 "초과 횟수(overrun count)"를 반환한다. 응용에서 그 초과 횟수를 이용해 주어진 시간 간격 동안 발생했을 타이머 만료 횟수를 정확히 계산할 수 있다. 시그널을 통해 (`SIGEV_SIGNAL`) 만료 알림을 수신할 때와 스레드를 통해 (`SIGEV_THREAD`) 수신할 때 모두에서 타이머 초과가 발생할 수 있다.

만료 알림이 시그널을 통해 전달될 때 다음과 같이 초과가 발생할 수 있다. 타이머 알림에 실시간 알림을 쓰는지 여부와 상관없이 시스템은 타이머마다 최대 한 개 시그널을 큐에 넣는다. (이는 POSIX.1에서 명세하는 동작 방식이다. 이와 달리 각 타이머 만료마다 시그널 하나를 큐에 넣는다면 큐에 넣을 수 있는 시그널 개수에 대한 시스템 상의 한계를 쉽게 넘을 수 있을 것이다.) 시스템 스케줄링 지연 때문에, 또는 시그널이 일시적으로 차단되어 있을 수도 있기 때문에 알림 시그널이 생성되는 시간과 (가령 시그널 핸들러에 잡혀서) 전달되거나 (가령 <tt>[[sigwaitinfo(2)]]</tt>로) 수용되는 시간 사이에 지연이 있을 수 있다. 그리고 그 시간 동안 타이머 만료가 추가로 발생할 수도 있다. 타이머 초과 횟수는 시그널이 생성된 시간과 전달 내지 수용된 시간 사이에 추가로 발생한 타이머 만료 횟수이다.

스레드 호출을 통해 만료 알림을 전달할 때 타이머 넘침이 발생할 수 있다. 타이머 만료와 알림 스레드 호출 사이에 임의적 지연이 있을 수 있으며 그 지연 시간 중에 추가 타이머 만료가 발생할 수도 있기 때문이다.

## RETURN VALUE

성공 시 `timer_getoverrun()`은 지정한 타이머의 초과 횟수를 반환한다. 초과가 일어나지 않았으면 이 횟수가 0일 수도 있다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>timerid</code>가 유효한 타이머 ID가 아니다.</dd>
</dl>

## VERSIONS

리눅스 2.6부터 이 시스템 호출이 사용 가능하다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

리눅스에서 시그널을 통해 (`SIGEV_SIGNAL`) 타이머 알림을 전달할 때에 `siginfo_t` 구조체(<tt>[[sigaction(2)]]</tt> 참고)의 `si_overrun` 필드를 통해 초과 횟수를 얻는 것도 가능하다. 이렇게 하면 응용에서 초과 횟수를 얻기 위해 시스템 호출을 하는 오버헤드를 피할 수 있다. 하지만 이는 POSIX.1에 대한 확장이며 이식성이 없다.

POSIX.1에서는 시그널을 이용한 타이머 알림 맥락에서만 타이머 초과를 논의한다.

## BUGS

POSIX.1에서는 타이머 초과 횟수가 구현에서 정의하는 최댓값 `DELAYTIMER_MAX`와 같거나 더 크면 `timer_getoverrun()`이 `DELAYTIMER_MAX`를 반환해야 한다고 명세한다. 하지만 리눅스에서는 이 기능을 구현하고 있지 않다. 대신 타이머 초과 값이 표현 가능한 정수 최댓값을 초과하면 카운터가 되돌아서 낮은 값부터 다시 시작한다.

## EXAMPLE

<tt>[[timer_create(2)]]</tt> 참고.

## SEE ALSO

<tt>[[clock_gettime(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signalfd(2)]]</tt>, <tt>[[sigwaitinfo(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[timer_delete(2)]]</tt>, <tt>[[timer_settime(2)]]</tt>, <tt>[[signal(7)]]</tt>, <tt>[[time(7)]]</tt>

----

2017-09-15
