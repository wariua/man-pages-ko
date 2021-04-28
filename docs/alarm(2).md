## NAME

alarm - 시그널 전달되도록 알람 시계 설정하기

## SYNOPSIS

```c
#include <unistd.h>

unsigned int alarm(unsigned int seconds);
```

## DESCRIPTION

`alarm()`은 `seconds` 초 후에 호출 프로세스에게 `SIGALRM` 시그널이 전달되도록 한다.

`seconds`가 0인 경우에는 대기 상태 알람이 있으면 취소하기만 한다.

어느 경우든 앞서 설정한 `alarm()`이 있으면 취소된다.

## RETURN VALUE

전달되기로 예약된 알람이 있었으면 그 시점까지 남아 있던 초 수를 `alarm()`이 반환한다. 예약된 알람이 없었으면 0을 반환한다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

## NOTES

`alarm()`과 <tt>[[setitimer(2)]]</tt>는 같은 타이머를 공유한다. 즉 한쪽을 호출하면 다른 쪽 사용에 영향을 주게 된다.

`alarm()`으로 생성된 알람이 <tt>[[execve(2)]]</tt>를 거치면서 유지된다. <tt>[[fork(2)]]</tt>를 통해 생긴 자식들이 물려받지 않는다.

<tt>[[sleep(3)]]</tt>이 `SIGALRM`을 이용해 구현돼 있을 수 있다. 즉 `alarm()`과 <tt>[[sleep(3)]]</tt>을 섞어 쓰는 건 좋지 않다.

언제나 그렇듯 스케줄링 지연 때문에 프로세스 실행이 임의 시간만큼 지연될 수 있다.

## SEE ALSO

<tt>[[gettimeofday(2)]]</tt>, <tt>[[pause(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[setitimer(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[timerfd_create(2)]]</tt>, <tt>[[sleep(3)]]</tt>, <tt>[[time(7)]]</tt>

----

2017-05-03
