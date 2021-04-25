## NAME

pause - 시그널 기다리기

## SYNOPSIS

```c
#include <unistd.h>

int pause(void);
```

## DESCRIPTION

`pause()`는 프로세스를 종료시키거나 시그널 잡기 함수 호출을 일으키는 시그널이 전달될 때까지 호출 프로세스를 (또는 스레드를) 잠들게 한다.

## RETURN VALUE

시그널을 잡고 시그널 잡기 함수가 반환했을 때에만 `pause()`가 반환한다. 이 경우 `pause()`는 -1을 반환하며 `errno`를 `EINTR`로 설정한다.

## ERRORS

`EINTR`
:   시그널을 잡았으며 시그널 잡기 함수가 반환했다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[sigsuspend(2)]]</tt>

----

2021-03-22
