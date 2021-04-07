## NAME

signalfd - 시그널을 받기 위한 파일 디스크립터 만들기

## SYNOPSIS

```c
#include <sys/signalfd.h>

int signalfd(int fd, const sigset_t *mask, int flags);
```

## DESCRIPTION

`signalfd()`는 파일 디스크립터를 생성하며, 이를 이용해 호출자를 대상으로 하는 시그널을 받을 수 있다. 시그널 핸들러나 <tt>[[sigwaitinfo(2)]]</tt> 사용의 대안이 되는데, <tt>[[select(2)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>로 파일 디스크립터를 감시할 수 있다는 장점이 있다.

`mask` 인자는 호출자가 파일 디스크립터를 통해 받고 싶은 시그널들의 집합을 나타낸다. <tt>[[sigsetops(3)]]</tt>에서 기술하는 매크로들로 그 시그널 집합의 내용을 초기화 할 수 있다. 보통은 그 시그널 집합을 <tt>[[sigprocmask(2)]]</tt>로 차단해야 하는데, 파일 디스크립터를 통해 수신할 시그널들이 기본 처리 방식에 따라 처리되는 것을 막기 위해서이다. signalfd 파일 디스크립터를 통해 `SIGKILL`이나 `SIGSTOP` 시그널을 받는 것은 불가능하다. `mask`에 그 시그널들을 지정하면 조용히 무시된다.

`fd` 인자가 -1이면 호출에서 새 파일 디스크립터를 생성하고 `mask`에 지정한 시그널 집합을 그 파일 디스크립터에 연계한다. `fd`가 -1이 아니면 유효한 기존 signalfd 파일 디스크립터를 지정해야 하며, 그 파일 디스크립터에 연계된 시그널 집합이 `mask`로 교체된다.

리눅스 2.6.27부터 `flags`에 다음 값들을 비트 OR 해서 `signalfd()`의 동작 방식을 바꿀 수 있다.

<dl>
<dt><code>SFD_NONBLOCK</code></dt>
<dd>새 파일 디스크립터가 가리키는 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)에 <code>O_NONBLOCK</code> 파일 상태 플래그를 설정한다. 이 플래그를 사용하면 같은 결과를 얻기 위해 <tt>[[fcntl(2)]]</tt>을 추가로 호출하지 않아도 된다.</dd>

<dt><code>SFD_CLOEXEC</code></dt>
<dd>새 파일 디스크립터에 'exec에서 닫기'(<code>FD_CLOEXEC</code>) 플래그를 설정한다. 이게 유용할 수 있는 이유에 대해선 <tt>[[open(2)]]</tt>의 <code>O_CLOEXEC</code> 플래그 설명을 보라.</dd>
</dl>

리눅스 버전 2.6.26까지는 `flags` 인자를 사용하지 않으며 0으로 지정해야 한다.

`signalfd()`가 반환하는 파일 디스크립터는 다음 작업을 지원한다.

<dl>
<dt><code>read(2)</code></dt>
<dd>

<code>mask</code>에 지정한 시그널들이 하나 이상 프로세스에 미처리 상태이면 <code>read(2)</code>에 준 버퍼를 이용해 시그널을 기술하는 <code>signalfd_siginfo</code> 구조체(아래 참고)를 하나 이상 반환한다. 미처리 시그널에 대한 정보를 버퍼에 가급적 많이 채워서 <code>read(2)</code>가 반환한다. 버퍼가 최소 <code>sizeof(struct signalfd_siginfo)</code> 바이트여야 한다. <code>read(2)</code>의 반환 값은 읽은 바이트 총수이다.

<code>read(2)</code>의 결과로 시그널이 소비되며, 그래서 더 이상 그 프로세스에 미처리인 시그널이 아니게 된다. (즉, 시그널 핸들러에 잡히지 않으며 <tt>[[sigwaitinfo(2)]]</tt>로 받을 수 없다.)

<code>mask</code> 내의 시그널 어느 것도 프로세스에 미처리 상태가 아니면 <code>mask</code> 내의 시그널들 중 하나가 프로세스에게 생성될 때까지 <code>read(2)</code>가 블록 한다. 파일 디스크립터를 논블록으로 만들었으면 <code>EAGAIN</code> 오류로 실패한다.
</dd>

<dt><tt>[[poll(2)]]</tt>, <tt>[[select(2)]]</tt> (기타 유사 함수)</dt>
<dd>

<code>mask</code> 내의 시그널이 하나 이상 프로세스에 미처리 상태인 경우에 파일 디스크립터가 읽기 가능하다. (<tt>[[select(2)]]</tt> <code>readfds</code> 인자, <tt>[[poll(2)]]</tt> <code>POLLIN</code> 플래그.)

signalfd 파일 디스크립터는 <tt>[[pselect(2)]]</tt>, <tt>[[ppoll(2)]]</tt>, <tt>[[epoll(7)]]</tt> 같은 다른 파일 디스크립터 다중화 API도 지원한다.
</dd>

<dt><tt>[[close(2)]]</tt></dt>
<dd>
파일 디스크립터가 더 이상 필요하지 않으면 닫아야 한다. 동일 signalfd 객체에 연계된 모든 파일 디스크립터가 닫혔을 때 커널이 그 객체의 자원을 해제한다.
</dd>
</dl>

### `signalfd_siginfo` 구조체

signalfd 파일 디스크립터에서 `read(2)`가 반환하는 `signalfd_siginfo` 구조체의 형식은 다음과 같다.

```c
struct signalfd_siginfo {
    uint32_t ssi_signo;    /* 시그널 번호 */
    int32_t  ssi_errno;    /* 오류 번호 (사용 안 함) */
    int32_t  ssi_code;     /* 시그널 코드 */
    uint32_t ssi_pid;      /* 송신자의 PID */
    uint32_t ssi_uid;      /* 송신자의 실제 UID */
    int32_t  ssi_fd;       /* 파일 디스크립터 (SIGIO) */
    uint32_t ssi_tid;      /* 커널 타이머 ID (POSIX 타이머) */
    uint32_t ssi_band;     /* 밴드 이벤트 (SIGIO) */
    uint32_t ssi_overrun;  /* POSIX 타이머 초과 횟수 */
    uint32_t ssi_trapno;   /* 시그널을 유발한 트랩 번호 */
    int32_t  ssi_status;   /* 종료 상태 또는 시그널 (SIGCHLD) */
    int32_t  ssi_int;      /* sigqueue(3)로 보낸 정수 */
    uint64_t ssi_ptr;      /* sigqueue(3)로 보낸 포인터 */
    uint64_t ssi_utime;    /* 사용자 CPU 소모 시간 (SIGCHLD) */
    uint64_t ssi_stime;    /* 시스템 CPU 소모 시간 (SIGCHLD) */
    uint64_t ssi_addr;     /* 시그널을 생성한 주소
                              (하드웨어 생성 시그널에서) */
    uint16_t ssi_addr_lsb; /* 주소의 최하위 비트
                              (SIGBUS, 리눅스 2.6.37부터) */
    uint8_t  pad[X];       /* 128바이트로 채우는 패드
                              (향후 필드 추가 가능) */
};
```

이 구조체의 각 필드는 `siginfo_t` 구조체의 비슷한 이름의 필드와 유사하다. `siginfo_t` 구조체는 <tt>[[sigaction(2)]]</tt>에서 설명한다. 특정 시그널에 대해 반환된 `signalfd_siginfo` 구조체의 모든 필드가 유효한 것은 아니다. `ssi_code` 필드에 반환된 값으로부터 유효한 필드들의 집합을 알아낼 수 있다. 그 필드는 `siginfo_t`의 `si_code` 필드에 대응한다. 자세한 내용은 <tt>[[sigaction(2)]]</tt> 참고.

### <tt>[[fork(2)]]</tt> 동작 방식

<tt>[[fork(2)]]</tt> 후에 자식이 signalfd 파일 디스크립터 사본을 물려받는다. 자식에서 그 파일 디스크립터에 `read(2)` 하면 자식의 큐에 있는 시그널에 대한 정보를 반환하게 된다.

### 파일 디스크립터 전달 동작 방식

다른 파일 디스크립터들과 마찬가지로 유닉스 도메인 소켓을 통해 signalfd 파일 디스크립터를 다른 프로세스로 전달할 수 있다. (<tt>[[unix(7)]]</tt> 참고.) 수신 쪽 프로세스에서 수신한 파일 디스크립터에 `read(2)` 하면 그 프로세스의 큐에 있는 시그널에 대한 정보를 반환하게 된다.

### <tt>[[execve(2)]]</tt> 동작 방식

다른 파일 디스크립터들과 마찬가지로 'exec에서 닫기' 플래그(<tt>[[fcntl(2)]]</tt> 참고) 표시를 하지 않았으면 <tt>[[execve(2)]]</tt>를 거치면서 signalfd 파일 디스크립터가 열린 채로 유지된다. <tt>[[execve(2)]]</tt> 전에 읽기가 가능했던 시그널이 있으면 새로 적재된 프로그램에게 읽기 가능하게 유지된다. (미처리인 차단 시그널이 <tt>[[execve(2)]]</tt>를 거치면서 미처리로 유지되는 전통적 시그널 동작 방식과 유사하다.)

### 스레드 동작 방식

다중 스레드 프로그램에서 signalfd 파일 디스크립터의 동작 방식은 시그널의 표준 동작 방식을 반영한다. 다시 말해 스레드가 signalfd 파일 디스크립터에서 읽기를 하면 그 스레드를 향한 시그널과 프로세스(즉 스레드 그룹 전체)를 향한 시그널을 읽게 된다. (스레드가 프로세스 내 다른 스레드를 향한 시그널을 읽을 수는 없다.)

## RETURN VALUE

성공 시 `signalfd()`는 signalfd 파일 디스크립터를 반환한다. `fd`가 -1이었으면 새 파일 디스크립터이고, `fd`가 유효한 파일 디스크립터였으면 `fd`이다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EBADF</code></dt>
<dd><code>fd</code> 파일 디스크립터가 유효한 파일 디스크립터가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>fd</code>가 유효한 signalfd 파일 디스크립터가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>flags</code>가 유효하지 않다. 또는 리눅스 2.6.26 또는 이전에서 <code>flags</code>가 0이 아니다.</dd>
<dt><code>EMFILE</code></dt>
<dd>열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다.</dd>
<dt><code>ENFILE</code></dt>
<dd>열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.</dd>
<dt><code>ENODEV</code></dt>
<dd>(내부적으로 쓰는) 익명 아이노드 장치를 마운트 할 수 없었다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>새 signalfd 파일 디스크립터를 생성하기에 메모리가 충분하지 않았다.</dd>
</dl>

## VERSIONS

리눅스 커널 2.6.22부터 `signalfd()`가 사용 가능하다. glibc 버전 2.8부터 잘 동작하는 지원을 제공한다. 리눅스 커널 2.6.27부터 `signalfd4()` 시스템 호출(NOTES 참고)이 사용 가능하다.

## CONFORMING TO

`signalfd()`와 `signalfd4()`는 리눅스 전용이다.

## NOTES

프로세스에서 signalfd 파일 디스크립터를 여러 개 만들 수 있다. 그래서 각 파일 디스크립터로 다른 시그널을 받는 게 가능하다. (<tt>[[select(2)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>로 파일 디스크립터를 감시하는 경우에 유용할 수 있다. 도착하는 시그널에 따라 다른 파일 디스크립터가 준비 상태가 된다.) 한 시그널이 여러 파일 디스크립터의 `mask`에 등장하는 경우에는 그 파일 디스크립터들 중 어느 것에서도 그 시그널의 발생을 (한 번) 읽을 수 있다.

`mask`에 `SIGKILL`이나 `SIGSTOP`을 포함시키려는 시도는 조용히 무시된다.

프로세스의 `/proc/[pid]/fdinfo` 디렉터리 내의 대응하는 파일 디스크립터 항목을 통해 signalfd 파일 디스크립터에서 쓰는 시그널 마스크를 볼 수 있다. 자세한 내용은 <tt>[[proc(5)]]</tt> 참고.

### 한계

비유효 메모리 주소 접근으로 인한 `SIGSEGV`나 산술 오류로 인한 `SIGFPE`처럼 동기적으로 생성되는 시그널을 받는 데 signalfd 메커니즘을 이용할 수 없다. 그런 시그널은 시그널 핸들러를 통해서만 잡을 수 있다.

앞서 기술한 것처럼 일반적인 사용 방식에서는 `signalfd()`를 통해 받을 시그널들을 막아 둔다. 자식 프로세스를 만들어서 (signalfd 파일 디스크립터를 필요로 하지 않는) 어떤 헬퍼 프로그램을 실행하려는 경우라면 보통은 <tt>[[fork(2)]]</tt>를 호출한 다음 <tt>[[execve(2)]]</tt> 호출 전에 그 시그널들을 풀어서 헬퍼 프로그램이 기대하는 시그널을 볼 수 있도록 하고 싶을 것이다. 하지만 프로그램에서 호출하는 어떤 라이브러리 함수에 의해 무대 뒤에서 헬퍼 프로그램이 생성되는 경우에는 그게 불가능할 것이다. 그런 경우에는 전통적인 시그널 핸들러로 후퇴해야 한다. 핸들러에서 어떤 파일 디스크립터에 쓰기를 하고 그걸 <tt>[[select(2)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>로 감시하면 된다.

### C 라이브러리/커널 차이

기반 리눅스 시스템 호출에서는 `mask` 인자의 크기를 지정하는 `size_t sizemask` 인자를 추가로 요구한다. glibc의 `signalfd()` 래퍼 함수에서 필요한 값을 기반 시스템 호출에 제공하기 때문에 래퍼 함수에는 그 인자가 없다.

기반 시스템 호출이 두 가지 있다. `signalfd()`와 더 최신인 `signalfd4()`이다. 앞쪽 시스템 호출은 `flags` 인자를 구현하지 않는다. 뒤쪽 시스템 호출은 위에 기술한 `flags` 값들을 구현한다. glibc 2.9부터 `signalfd()` 래퍼 함수에서 가능한 경우 `signalfd4()`를 사용한다.

## BUGS

2.6.25 전의 커널에서는 <tt>[[sigqueue(3)]]</tt>로 보낸 시그널 동반 데이터가 `ssi_ptr` 및 `ssi_int` 필드에 채워지지 않는다.

## EXAMPLE

아래 프로그램은 signalfd 파일 디스크립터를 통해 시그널 `SIGINT`와 `SIGQUIT`을 받는다. `SIGQUIT` 시그널을 받은 후에 프로그램이 종료한다. 다음 셸 세션이 프로그램 사용 방식을 보여 준다.

```
$ ./signalfd_demo
^C                   # Control-C로 SIGINT 생성
Got SIGINT
^C
Got SIGINT
^\                   # Control-\로 SIGQUIT 생성
Got SIGQUIT
$
```

### 프로그램 소스

```c
#include <sys/signalfd.h>
#include <signal.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define handle_error(msg) \
    do { perror(msg); exit(EXIT_FAILURE); } while (0)

int
main(int argc, char *argv[])
{
    sigset_t mask;
    int sfd;
    struct signalfd_siginfo fdsi;
    ssize_t s;

    sigemptyset(&mask);
    sigaddset(&mask, SIGINT);
    sigaddset(&mask, SIGQUIT);

    /* 시그널을 막아서 기본 처리 방식에 따라
       처리되지 않도록 함 */

    if (sigprocmask(SIG_BLOCK, &mask, NULL) == -1)
        handle_error("sigprocmask");

    sfd = signalfd(-1, &mask, 0);
    if (sfd == -1)
        handle_error("signalfd");

    for (;;) {
        s = read(sfd, &fdsi, sizeof(struct signalfd_siginfo));
        if (s != sizeof(struct signalfd_siginfo))
            handle_error("read");

        if (fdsi.ssi_signo == SIGINT) {
            printf("Got SIGINT\n");
        } else if (fdsi.ssi_signo == SIGQUIT) {
            printf("Got SIGQUIT\n");
            exit(EXIT_SUCCESS);
        } else {
            printf("Read unexpected signal\n");
        }
    }
}
```

## SEE ALSO

<tt>[[eventfd(2)]]</tt>, <tt>[[poll(2)]]</tt>, `read(2)`, <tt>[[select(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[sigwaitinfo(2)]]</tt>, <tt>[[timerfd_create(2)]]</tt>, <tt>[[sigsetops(3)]]</tt>, <tt>[[sigwait(3)]]</tt>, <tt>[[epoll(7)]]</tt>, <tt>[[signal(7)]]</tt>

----

2019-03-06
