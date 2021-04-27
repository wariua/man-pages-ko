## NAME

poll, ppoll - 파일 디스크립터에서 어떤 이벤트 기다리기

## SYNOPSIS

```c
#include <poll.h>

int poll(struct pollfd *fds, nfds_t nfds, int timeout);

#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <signal.h>
#include <poll.h>

int ppoll(struct pollfd *fds, nfds_t nfds,
        const struct timespec *tmo_p, const sigset_t *sigmask);
```

## DESCRIPTION

`poll()`은 <tt>[[select(2)]]</tt>와 비슷한 작업을 한다. 즉 한 파일 디스크립터라도 I/O를 수행할 수 있는 상태가 되기를 기다린다. 리눅스 전용인 <tt>[[epoll(7)]]</tt> API도 비슷한 작업을 하며 `poll()`에서 볼 수 있는 것 이상의 기능을 제공한다.

감시할 파일 디스크립터들의 집합을 `fds` 인자에서 지정하는데, 다음과 같은 구조체의 배열이다.

```c
struct pollfd {
    int   fd;         /* 파일 디스크립터 */
    short events;     /* 요청한 이벤트 */
    short revents;    /* 반환된 이벤트 */
};
```

`fds`의 항목 개수를 호출자가 `nfds`에 명시해야 한다.

`fd` 필드는 열린 파일에 대한 파일 디스크립터를 담는다. 이 필드가 음수이면 대응하는 `events` 필드를 무시하며 `revents` 필드에 0을 반환한다. (이 점을 이용하면 어떤 파일 디스크립터를 `poll()` 호출 한 번 동안만 무시하는 걸 간단히 할 수 있다. `fd` 필드 부호만 바꾸면 된다. 다만 파일 디스크립터 0은 이 기법으로 무시할 수 없다.)

`events` 필드는 입력 매개변수이며 파일 디스크립터 `fd`에 대해 응용에서 관심 있는 이벤트들을 나타내는 비트 마스크이다. 이 필드를 0으로 지정할 수도 있으며, 그 경우 `revents`로 반환될 수 이벤트는 `POLLHUP`, `POLLERR`, `POLLNVAL`뿐이다. (아래 참고.)

`revents` 필드는 출력 매개변수이며 실제 발생한 이벤트들을 커널에서 채운다. `revents`로 반환되는 비트들에는 `events`에 지정한 비트들이, 그리고 값 `POLLERR`, `POLLHUP`, `POLLNVAL` 중 하나가 포함될 수 있다. (이 세 비트는 `events` 필드에서는 무의미하며 해당 조건이 참이면 `revents` 필드에 설정된다.)

파일 디스크립터 모두에서 요청 이벤트 어느 것도 (그리고 어떤 오류도) 발생하지 않았으면 그 이벤트들 중 하나가 일어날 때까지 `poll()`이 블록 한다.

`timeout` 인자는 파일 디스크립터가 준비 상태가 되기를 기다리며 `poll()`에서 블록 할 밀리초 수를 나타낸다. 다음 어느 경우든 해당할 때까지 호출이 블록 하게 된다.

* 파일 디스크립터가 준비 상태가 된다.

* 호출이 시그널 핸들러에 의해 중단된다.

* 타임아웃이 만료된다.

참고로 `timeout` 시간을 시스템 클럭 해상도에 따라 올림 하게 되며 커널 스케줄링 지연도 있기 때문에 그 블록 시간을 약간 넘길 수도 있다. `timeout`에 음수 값을 지정하는 건 무한 타임아웃을 뜻한다. `timeout`을 0으로 지정하면 준비 상태인 파일 디스크립터가 없더라도 `poll()`이 즉시 반환하게 된다.

`events`에 설정하고 `revents`로 반환될 수 있는 비트들이 `<poll.h>`에 정의돼 있다.

`POLLIN`
:   읽을 데이터가 있다.

`POLLPRI`
:   파일 디스크립터에 어떤 예외 상황이 있다. 다음 경우들 등이 가능하다.

    * TCP 소켓에 대역외 데이터가 있다. (<tt>[[tcp(7)]]</tt> 참고.)

    * 패킷 모드인 유사 터미널 마스터가 슬레이브에서의 상태 변화를 보았다. (`ioctl_tty(2)` 참고.)

    * `cgroup.events` 파일이 변경되었다. (<tt>[[cgroups(7)]]</tt> 참고.)

`POLLOUT`
:   현재 쓰기가 가능하다. 다만 소켓 내지 파이프의 가용 공간보다 큰 데이터를 써넣으면 (`O_NONBLOCK`이 설정돼 있지 않으면) 여전히 블록 된다.

`POLLRDHUP` (리눅스 2.6.17부터)
:   스트림 소켓의 상대가 연결을 닫았거나 연결의 쓰기 쪽을 닫았다. 이 비트 정의를 쓸 수 있으려면 (*어떤* 헤더 파일도 포함시키기 전에) 기능 확인 매크로 `_GNU_SOURCE`가 정의돼 있어야 한다.

`POLLERR`
:   오류 상황. (`revents`로 반환되기만 하고 `events`에서는 무시.) 파이프의 쓰기 쪽을 가리키는 파일 디스크립터에 대해서 읽기 쪽이 닫혔을 때도 이 비트가 설정된다.

`POLLHUP`
:   연결 끊겼음. (`revents`로 반환되기만 하고 `events`에서는 무시.) 참고로 파이프나 스트림 소켓 같은 채널에서 읽기를 할 때 이 이벤트는 상대가 채널의 그쪽 끝을 닫았다는 표시일 뿐이다. 그 채널의 미처리 데이터를 모두 소비한 후에야 채널 읽기가 0(파일 끝)을 반환하게 된다.

`POLLNVAL`
:   유효하지 않은 요청: `fd`가 열려 있지 않음. (`revents`로 반환되기만 하고 `events`에서는 무시.) 

`_XOPEN_SOURCE`가 정의된 채로 컴파일 하면 다음 비트들도 쓸 수 있게 되는데, 위에 나열한 비트들 이상의 정보를 주지는 않는다.

`POLLRDNORM`
:   `POLLIN`과 동등하다.

`POLLRDBAND`
:   우선 대역 데이터를 읽을 수 있다. (리눅스에서는 보통 쓰지 않음.)

`POLLWRNORM`
:   `POLLOUT`과 동등하다.

`POLLWRBAND`
:   우선 대역 데이터를 쓸 수 있다.

리눅스에는 `POLLMSG`도 있지만 쓰지는 않는다.

### `ppoll()`

`poll()`과 `ppoll()`의 관계는 <tt>[[select(2)]]</tt>와 <tt>[[pselect(2)]]</tt>의 관계와 비슷하다. <tt>[[pselect(2)]]</tt>처럼 응용에서 `ppoll()`을 사용해 파일 디스크립터가 준비 상태가 되거나 시그널을 잡을 때까지 안전하게 대기할 수 있다.

`timeout` 인자의 정밀도 차이를 제외하면 다음 `ppoll()` 호출은

```c
ready = ppoll(&fds, nfds, tmo_p, &sigmask);
```

다음 호출들을 *원자적으로* 실행하는 것과 거의 동등하다.

```c
sigset_t origmask;
int timeout;

timeout = (tmo_p == NULL) ? -1 :
          (tmo_p->tv_sec * 1000 + tmo_p->tv_nsec / 1000000);
pthread_sigmask(SIG_SETMASK, &sigmask, &origmask);
ready = poll(&fds, nfds, timeout);
pthread_sigmask(SIG_SETMASK, &origmask, NULL);
```

위 코드가 *거의* 동등하다고 한 이유는 `poll()`에선 음수 `timeout` 값을 무한대 타임아웃으로 해석하지만 `ppoll()`에선 `*tmo_p`의 값이 음수이면 오류가 나기 때문이다.

`ppoll()`이 필요한 이유는 <tt>[[pselect(2)]]</tt>의 설명을 보라.

`sigmask`를 NULL로 지정한 경우에는 시그널 마스크 조작을 수행하지 않는다. (그러면 `ppoll()`과 `poll()`의 차이는 `timeout` 인자의 정밀도뿐이다.)

`tmo_p` 인자는 `ppoll()`에서 블록 할 시간의 상한을 나타낸다. 이 인자는 다음 구조체의 포인터이다.

```c
struct timespec {
    long    tv_sec;         /* 초 */
    long    tv_nsec;        /* 나노초 */
};
```

`tmo_p`를 NULL로 지정한 경우에는 `ppoll()`에서 무한정 블록 할 수 있다.

## RETURN VALUE

성공 시 `poll()`은 음수 아닌 값을 반환하는데, 이는 `revents` 필드가 (이벤트나 오류를 나타내는) 0 아닌 값으로 설정된 `pollfds` 내 항목 개수다. 반환 값 0은 파일 디스크립터가 준비 상태가 되기 전에 호출이 타임아웃 되었음을 나타낸다.

오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   인자로 준 배열이 호출 프로그램의 주소 공간에 들어 있지 않다.

`EINTR`
:   요청한 이벤트들에 앞서 시그널이 발생했다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `nfds` 값이 `RLIMIT_NOFILE` 값을 초과한다.

`EINVAL`
:   (`ppoll()`) `*ip`에 있는 타임아웃 값이 유효하지 않다 (음수다).

`ENOMEM`
:   커널 자료 구조를 위한 메모리를 할당할 수 없다.

## VERSIONS

리눅스 2.1.23에서 `poll()` 시스템 호출이 도입되었다. 이 시스템 호출이 없는 더 전 커널에서는 glibc의 래퍼 함수에서 <tt>[[select(2)]]</tt>를 써서 에뮬레이션을 제공한다.

리눅스 커널 2.6.16에서 `ppoll()` 시스템 호출이 추가되었다. glibc 2.4에서 `ppoll()` 라이브러리 호출이 추가되었다.

## CONFORMING TO

`poll()`은 POSIX.1-2001 및 POSIX.1-2008을 준수한다. `ppoll()`은 리눅스 전용이다.

## NOTES

`poll()` 및 `ppoll()`의 동작은 `O_NONBLOCK` 플래그에 영향을 받지 않는다.

일부 다른 유닉스 시스템에서는 커널 내부 자원을 할당하지 못했을 때 리눅스의 `ENOMEM`이 아니라 `EAGAIN` 오류로 실패할 수 있다. POSIX에서 그런 동작을 허용한다. 이식 가능한 프로그램에서는 `EAGAIN`을 확인해서 `EINTR` 경우처럼 루프를 계속 도는 게 좋을 수 있다.

어떤 구현들에서는 `poll()`에서 `timeout`에 쓸 수 있는 비표준 상수 `INFTIM`을 -1 값으로 정의하고 있다. glibc에서는 이 상수를 제공하지 않는다.

`poll()`로 감시 중인 파일 디스크립터를 다른 스레드에서 닫으면 어떻게 될 수 있는지에 대한 설명은 <tt>[[select(2)]]</tt>를 보라.

### C 라이브러리/커널 차이

리눅스의 `ppoll()` 시스템 호출에서는 `tmo_p` 인자를 변경한다. 하지만 glibc 래퍼 함수에서 타임아웃 인자에 대한 지역 변수를 쓰고 그 변수를 시스템 호출로 전달하여 그 동작 방식을 감춘다. 그리하여 glibc의 `ppoll()` 함수는 `tmo_p` 인자를 변경하지 않는다.

진짜 `ppoll()` 시스템 호출에는 다섯 번째 인자 `size_t sigsetsize`가 있는데, 이는 `sigmask` 인자의 바이트 단위 크기를 나타낸다. glibc의 `ppoll()` 래퍼 함수에서 이 인자를 고정된 값(`sizeof(kernel_sigset_t)`)으로 지정한다. 커널과 libc에서의 시그널 집합 개념 차이에 대한 설명은 <tt>[[sigprocmask(2)]]</tt>를 보라.

## BUGS

<tt>[[select(2)]]</tt>의 BUGS 절에 있는 준비 상태 거짓 알림 설명을 보라.

## EXAMPLES

아래 프로그램은 명령행 인자로 받은 파일 각각을 열어서 파일 디스크립터가 읽기 준비(`POLLIN`)가 되는지 감시한다. 루프를 돌며 `poll()`을 반복 사용해서 파일 디스크립터들을 감시하고, 준비 상태인 파일 디스크립터 수를 찍는다. 그리고 준비 상태인 파일 디스크립터 각각에 대해 다음을 수행한다.

* 반환된 `revents` 필드를 사람이 읽을 수 있는 형태로 표시.

* 파일 디스크립터가 읽기 가능이면 데이터를 좀 읽어서 표준 출력으로 표시.

* 파일 디스크립터가 읽기 가능이 아니고 어떤 다른 이벤트(아마 `POLLHUP`)가 발생했으면 파일 디스크립터 닫기.

한 터미널에서 프로그램을 실행해서 FIFO를 열게 한다고 하자.

```text
$ mkfifo myfifo
$ ./poll_input myfifo
```

그리고 두 번째 터미널 창에서 그 FIFO를 쓰기용으로 열어서 데이터를 좀 써넣은 다음 닫자.

```text
$ echo aaaaabbbbbccccc > myfifo
```

그러면 프로그램이 도는 터미널에서 다음을 보게 된다.

```text
Opened "myfifo" on fd 3
About to poll()
Ready: 1
  fd=3; events: POLLIN POLLHUP
    read 10 bytes: aaaaabbbbb
About to poll()
Ready: 1
  fd=3; events: POLLIN POLLHUP
    read 6 bytes: ccccc

About to poll()
Ready: 1
  fd=3; events: POLLHUP
    closing fd 3
All file descriptors closed; bye
```

위 출력을 보면 `poll()`이 세 번 반환한 것을 알 수 있다.

* 첫 번째 반환 시 `revents` 필드로 반환된 비트는 파일 디스크립터가 읽기 가능이라는 뜻의 `POLLIN`과 FIFO 맞은편이 닫혔음을 나타내는 `POLLHUP`이었다. 가용 입력 일부를 프로그램에서 읽었다.

* 두 번째 `poll()` 반환에서도 `POLLIN`과 `POLLHUP`이 표시됐다. 가용 입력 나머지를 프로그램에서 읽었다.

* 마지막 반환에서 `poll()`은 FIFO에 `POLLHUP`만 표시했고, 그 시점에서 파일 디스크립터를 닫고 프로그램이 종료했다.

### 프로그램 소스

```c
/* poll_input.c

   Licensed under GNU General Public License v2 or later.
*/
#include <poll.h>
#include <fcntl.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

int
main(int argc, char *argv[])
{
    int nfds, num_open_fds;
    struct pollfd *pfds;

    if (argc < 2) {
       fprintf(stderr, "Usage: %s file...\n", argv[0]);
       exit(EXIT_FAILURE);
    }

    num_open_fds = nfds = argc - 1;
    pfds = calloc(nfds, sizeof(struct pollfd));
    if (pfds == NULL)
        errExit("malloc");

    /* 명령행의 각 파일을 열어서 `pfds` 배열에 추가하기. */

    for (int j = 0; j < nfds; j++) {
        pfds[j].fd = open(argv[j + 1], O_RDONLY);
        if (pfds[j].fd == -1)
            errExit("open");

        printf("Opened \"%s\" on fd %d\n", argv[j + 1], pfds[j].fd);

        pfds[j].events = POLLIN;
    }

    /* 파일 디스크립터가 하나라도 열려 있는 동안 계속 poll()
       호출하기. */

    while (num_open_fds > 0) {
        int ready;

        printf("About to poll()\n");
        ready = poll(pfds, nfds, -1);
        if (ready == -1)
            errExit("poll");

        printf("Ready: %d\n", ready);

        /* poll()이 반환한 배열 다루기. */

        for (int j = 0; j < nfds; j++) {
            char buf[10];

            if (pfds[j].revents != 0) {
                printf("  fd=%d; events: %s%s%s\n", pfds[j].fd,
                        (pfds[j].revents & POLLIN)  ? "POLLIN "  : "",
                        (pfds[j].revents & POLLHUP) ? "POLLHUP " : "",
                        (pfds[j].revents & POLLERR) ? "POLLERR " : "");

                if (pfds[j].revents & POLLIN) {
                    ssize_t s = read(pfds[j].fd, buf, sizeof(buf));
                    if (s == -1)
                        errExit("read");
                    printf("    read %zd bytes: %.*s\n",
                            s, (int) s, buf);
                } else {                /* POLLERR | POLLHUP */
                    printf("    closing fd %d\n", pfds[j].fd);
                    if (close(pfds[j].fd) == -1)
                        errExit("close");
                    num_open_fds--;
                }
            }
        }
    }

    printf("All file descriptors closed; bye\n");
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[restart_syscall(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[select_tut(2)]]</tt>, <tt>[[epoll(7)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
