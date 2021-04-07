## NAME

raise - 호출자에게 시그널 보내기

## SYNOPSIS

```c
#include <signal.h>

int raise(int sig);
```

## DESCRIPTION

`raise()` 함수는 호출 프로세스 내지 스레드에게 시그널을 보낸다.  단일 스레드 프로그램에서는 다음과 동등하다.

```c
kill(getpid(), sig);
```

다중 스레드 프로그램에서는 다음과 동등하다.

```c
pthread_kill(pthread_self(), sig);
```

시그널로 인해 핸들러가 호출되는 경우 시그널 핸들러가 반환한 후에야 `raise()`가 반환하게 된다.

## RETURN VALUE

`raise()`는 성공 시 0을 반환하고 실패 시 0 아닌 값을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `raise()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99.

## NOTES

버전 2.3.3부터 glibc에서는 커널이 <tt>[[tgkill(2)]]</tt> 시스템 호출을 지원하는 경우 이를 호출하는 것으로 `raise()`를 구현한다. 그 전의 glibc 버전들에서는 <tt>[[kill(2)]]</tt>을 이용해 `raise()`를 구현했다.

## SEE ALSO

<tt>[[getpid(2)]]</tt>, <tt>[[kill(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[pthread_kill(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2015-08-08
