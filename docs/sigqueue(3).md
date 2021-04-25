## NAME

sigqueue - 프로세스에게 시그널과 데이터 큐잉 하기

## SYNOPSIS

```c
#include <signal.h>

int sigqueue(pid_t pid, int sig, const union sigval value);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sigqueue()`:
:   `_POSIX_C_SOURCE >= 199309L`

## DESCRIPTION

`sigqueue()`는 `pid`에 준 PID의 프로세스에게 `sig`에 지정한 시그널을 보낸다. 시그널을 보내기 위해 필요한 권한은 <tt>[[kill(2)]]</tt>과 같다. <tt>[[kill(2)]]</tt>에서처럼 널 시그널(0)을 이용해 해당 PID의 프로세스가 존재하는지 확인할 수 있다.

시그널에 동반해 보낼 데이터 항목(정수 또는 포인터 값)을 `value` 인자로 지정하며, 다음 타입이다.

```c
union sigval {
    int   sival_int;
    void *sival_ptr;
};
```

수신 프로세스에서 <tt>[[sigaction(2)]]</tt>에 `SA_SIGINFO` 플래그를 써서 이 시그널에 대한 핸들러를 설치해 두었다면 핸들러 두 번째 인자로 전달되는 `siginfo_t` 구조체의 `si_value` 필드를 통해 이 데이터를 얻을 수 있다. 더불어 그 구조체의 `si_code` 필드가 `SI_QUEUE`로 설정될 것이다.

## RETURN VALUE

성공 시 `sigqueue()`는 0을 반환하여 시그널이 수신 프로세스에게 성공적으로 큐잉 되었음을 나타낸다. 그 외의 경우에는 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EAGAIN`
:   큐에 넣을 수 있는 시그널 개수 한계에 도달했다. (자세한 내용은 <tt>[[signal(7)]]</tt>을 보라.)

`EINVAL`
:   `sig`가 유효하지 않다.

`EPERM`
:   프로세스가 수신 프로세스로 시그널을 보낼 권한을 가지고 있지 않다. 필요한 권한에 대해선 <tt>[[kill(2)]]</tt>을 보라.

`ESRCH`
:   `pid`에 일치하는 PID의 프로세스가 없다.

## VERSIONS

리눅스 2.2에서 `sigqueue()`와 기반 되는 <tt>[[rt_sigqueueinfo(2)]]</tt> 시스템 호출이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `sigqueue()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

이 함수로 인해 호출한 프로세스에게 시그널을 보내게 되는데 호출 스레드가 그 시그널을 막고 있지 않았고, 다른 어떤 스레드도 (비차단으로 두거나 <tt>[[sigwait(3)]]</tt>으로 기다려서) 그 시그널을 처리하려는 의사가 없었으면 이 함수가 반환하기 전에 이 스레드에게 적어도 어떤 시그널이 전달되어야 한다.

### C 라이브러리/커널 차이

리눅스에서 `sigqueue()`는 <tt>[[rt_sigqueueinfo(2)]]</tt> 시스템 호출을 이용해 구현되어 있다. 그 시스템 호출은 세 번째 인자가 `siginfo_t` 구조체인 점이 다른데, 이 구조체가 수신 프로세스의 시그널 핸들러에게 제공되거나 수신 프로세스의 <tt>[[sigtimedwait(2)]]</tt> 호출에 의해 반환된다. glibc의 `sigqueue()` 래퍼에서 이 인자 `uinfo`를 다음과 같이 설정한다.

```c
uinfo.si_signo = sig;      /* sigqueue()에게 준 인자 */
uinfo.si_code = SI_QUEUE;
uinfo.si_pid = getpid();   /* 송신자의 프로세스 ID */
uinfo.si_uid = getuid();   /* 송신자의 실제 UID */
uinfo.si_value = val;      /* sigqueue()에게 준 인자 */
```

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[rt_sigqueueinfo(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[pthread_sigqueue(3)]]</tt>, <tt>[[sigwait(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
