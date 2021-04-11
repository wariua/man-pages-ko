## NAME

select, pselect, FD_CLR, FD_ISSET, FD_SET, FD_ZERO - 동기적 I/O 다중화

## SYNOPSIS

```c
/* POSIX.1-2001, POSIX.1-2008에 따르면 */
#include <sys/select.h>

/* 이전 표준들에 따르면 */
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *utimeout);

void FD_CLR(int fd, fd_set *set);
int  FD_ISSET(int fd, fd_set *set);
void FD_SET(int fd, fd_set *set);
void FD_ZERO(fd_set *set);

#include <sys/select.h>

int pselect(int nfds, fd_set *readfds, fd_set *writefds,
            fd_set *exceptfds, const struct timespec *ntimeout,
            const sigset_t *sigmask);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pselect()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`select()`를 (또는 `pselect()`를) 이용해 여러 파일 디스크립터를 효율적으로 감시할 수 있다. 그 중 일부가 "준비" 상태가 됐는지, 즉 I/O가 가능해졌거나 그 중 일부에서 "예외적인 상황"이 발생했는지 알아볼 수 있다.

주된 인자는 세 가지 파일 디스크립터 "집합"인 `readfds`, `writefds`, `exceptfds`이다. 각 집합은 `fd_set` 타입으로 선언하며 매크로 `FD_CLR()`, `FD_ISSET()`, `FD_SET()`, `FD_ZERO()`로 그 내용을 조작할 수 있다. 새로 집합을 선언하면 먼저 `FD_ZERO()`로 비우는 게 좋다. `select()`에서 아래에 기술하는 규칙에 따라 집합의 내용을 변경하며, `select()` 호출 후에 `FD_ISSET()` 매크로를 이용해 집합에 파일 디스크립터가 계속 있는지 확인할 수 있다. 지정한 파일 디스크립터가 집합 안에 있으면 `FD_ISSET()`이 0 아닌 값을 반환하고 없으면 0을 반환한다. `FD_CLR()`는 집합에서 파일 디스크립터를 뺀다.

### 인자

`readfds`
:   파일 디스크립터들 중 하나라도 읽기 가능한 데이터가 있는지 보기 위해 이 집합을 감시한다. `select()` 반환 후에 `readfds`에는 즉시 읽기가 가능하지 않은 파일 디스크립터들이 모두 지워져 있게 된다.

`writefds`
:   파일 디스크립터들 중 하나라도 데이터를 쓸 공간이 있는지 보기 위해 이 집합을 감시한다. `select()` 반환 후에 `writefds`에는 즉시 쓰기가 가능하지 않은 파일 디스크립터들이 모두 지워져 있게 된다.

`exceptfds`
:   "예외적 상황"을 확인하기 위해 이 집합을 감시한다. 실무에서 유일한 그런 예외적 상황은 꽤 흔한데, 바로 TCP 소켓에서 *대역외*(OOB) 데이터를 읽을 수 있는 경우이다. OOB 데이터에 대한 자세한 내용은 <tt>[[recv(2)]]</tt>, <tt>[[send(2)]]</tt>, <tt>[[tcp(7)]]</tt>를 보라. (<tt>[[select(2)]]</tt>에서 예외 상황이라고 나타내는 덜 흔한 또 다른 경우는 패킷 모드 유사 터미널에서 발생한다. `ioctl_tty(2)` 참고.) `select()` 반환 후에 `exceptfds`에는 예외 상황이 발생하지 않은 파일 디스크립터들이 모두 지워져 있게 된다.

`nfds`
:   집합들에 있는 파일 디스크립터 중 가장 큰 값에 1을 더한 정수이다. 달리 말하면 집합 각각에 파일 디스크립터를 추가하면서 그 중 최대인 정수를 계산한 다음 1만큼 올린 값을 `nfds`로 전달하면 된다.

`utimeout`
:   뭔가 특별한 일이 일어나지 않더라도 `select()`가 반환 전에 최대 이 시간만큼 대기할 수 있다. 이 값을 NULL로 전달하면 파일 디스크립터가 준비 상태가 되기를 기다리며 `select()`가 무한정 블록 한다. `utimeout`을 0초로 설정할 수도 있는데, 그러면 `select()`가 호출 시점의 파일 디스크립터 준비 상태 정보를 가지고 즉시 반환한다. `struct timeval` 구조체는 다음과 같이 정의돼 있다.

        struct timeval {
            time_t tv_sec;    /* 초 */
            long tv_usec;     /* 마이크로초 */
        };

`ntimeout`
:   `pselect()`의 이 인자는 `utimeout`과 의미가 같되 다음과 같이 `struct timespec`의 정밀도가 나노초이다.

        struct timespec {
            long tv_sec;    /* 초 */
            long tv_nsec;   /* 나노초 */
        };

`sigmask`
:   이 인자는 호출자가 `pselect()` 호출 내에 블록 되어 있는 동안 커널에서 차단을 풀어야 하는 (즉 호출 스레드의 시그널 마스크에서 빼야 하는) 시그널들의 집합을 담는다. (<tt>[[sigaddset(3)]]</tt> 및 <tt>[[sigprocmask(2)]]</tt> 참고.) NULL일 수 있으며, 그때는 함수에 들어가고 나올 때 시그널 마스크를 변경하지 않는다. 이 경우 `pselect()`는 그냥 `select()`처럼 동작하게 된다.

### 시그널과 데이터 이벤트 병행하기

파일 디스크립터가 I/O 준비가 되는 것뿐 아니라 시그널도 함께 기다릴 때 `pselect()`가 유용하다. 시그널을 받는 프로그램들은 보통 시그널 핸들러에서 전역 플래그에 표시만 해 둔다. 그 전역 플래그는 프로그램 메인 루프에서 이벤트를 처리해야 한다는 표시이다. 시그널이 전달되면 `select()` (또는 `pselect()`) 호출이 `errno`에 `EINTR`를 설정하고 반환하게 된다. 이렇게 동작하지 않는다면 `select()`가 무한정 블록 할 수도 있으므로 이 동작 방식은 프로그램 메인 루프에서 시그널을 처리하는 데 꼭 필요하다. 이제 메인 루프 내의 어딘가에 조건문이 있어서 그 전역 플래그를 확인할 것이다. 그런데 그 조건문 다음에, 그러면서 `select()` 호출 전에 시그널이 도착하면 어떻게 될까? 답은, 처리를 기다리는 이벤트가 분명 있는데도 `select()`가 무한정 블록 하게 된다는 것이다. 이 경쟁 조건을 해결해 주는 것이 `pselect()` 호출이다. 이 호출을 사용하면 `pselect()` 호출 내에서만 수신할 시그널들을 그 시그널 마스크에 설정할 수 있다. 예를 들어 문제의 이벤트가 자식 프로세스 종료라고 하자. 일단 메인 루프 시작 전에 <tt>[[sigprocmask(2)]]</tt>를 이용해 `SIGCHLD`를 막아 둔다. 그리고 빈 시그널 마스크를 사용하면 `pselect()`에서 `SIGCHLD`를 활성화한다. 다음과 같은 프로그램이 될 것이다.

```c
static volatile sig_atomic_t got_SIGCHLD = 0;

static void
child_sig_handler(int sig)
{
    got_SIGCHLD = 1;
}

int
main(int argc, char *argv[])
{
    sigset_t sigmask, empty_mask;
    struct sigaction sa;
    fd_set readfds, writefds, exceptfds;
    int r;

    sigemptyset(&sigmask);
    sigaddset(&sigmask, SIGCHLD);
    if (sigprocmask(SIG_BLOCK, &sigmask, NULL) == -1) {
        perror("sigprocmask");
        exit(EXIT_FAILURE);
    }

    sa.sa_flags = 0;
    sa.sa_handler = child_sig_handler;
    sigemptyset(&sa.sa_mask);
    if (sigaction(SIGCHLD, &sa, NULL) == -1) {
        perror("sigaction");
        exit(EXIT_FAILURE);
    }

    sigemptyset(&empty_mask);

    for (;;) {          /* 메인 루프 */
        /* pselect() 호출 전에 readfds, writefds,
           exceptfds를 설정한다. (코드 생략) */

        r = pselect(nfds, &readfds, &writefds, &exceptfds,
                    NULL, &empty_mask);
        if (r == -1 && errno != EINTR) {
            /* 오류 처리 */
        }

        if (got_SIGCHLD) {
            got_SIGCHLD = 0;

            /* 시그널로 받은 이벤트를 여기서 처리.
               가령 종료한 자식들 wait() 하기. (코드 생략) */
        }

        /* 프로그램 주 작업 */
    }
}
```

### 실용성

근데 `select()`가 왜 있는 걸까? 그냥 하고 싶을 때 파일 디스크립터에서 읽기와 쓰기를 하면 안 될까? `select()`의 핵심은 동시에 여러 파일 디스크립터를 감시하고 아무 활동이 없으면 올바로 프로세스를 잠들게 한다는 점이다. 유닉스 프로그래머들은 종종 데이터 흐름이 간헐적인 여러 파일 디스크립터들에서 I/O 처리를 해야 할 때가 있다. 단순히 `read(2)`와 `write(2)`를 차례로 호출하기만 한다면 어떤 파일 디스크립터에 대기하며 호출이 블록하고 있는 동안 다른 파일 디스크립터가 I/O 준비 상태인데도 사용하지 못하는 경우를 보게 될 것이다. `select()`로 그런 상황에 효율적으로 대처할 수 있다.

### 선택 규칙

`select()`를 써 보려는 많은 이들은 이해하기 어려우며 이식성이 없거나 오락가락하는 결과를 내놓는 동작 방식에 부닥친다. 예를 들어 위 프로그램은 파일 디스크립터를 논블로킹 모드로 설정하지 않았지만 어느 지점에서도 블록 하지 않도록 조심스럽게 작성된 것이다. 자칫하면 `select()`를 쓰는 장점을 없애 버리는 미묘한 오류를 만들게 되기 쉬우므로 `select()` 사용 시 주의해야 할 핵심 사항들을 여기 나열한다.

1. 가급적 `select()`를 타임아웃 없이 쓰는 게 좋다. 가용 데이터가 없으면 프로그램에서 할 일이 없어야 한다. 타임아웃에 의존하는 코드는 항상 이식 가능하지는 않으며 디버그 하기 어렵다.

2. 효율성을 위해 `nfds`의 값을 위에 설명한 것처럼 올바로 계산해야 한다.

3. `select()` 호출 후에 결과를 확인해서 적절히 대응할 게 아니라면 파일 디스크립터를 아무 집합에도 추가하지 마라. 다음 규칙 참고.

4. `select()` 반환 후에 모든 집합의 모든 파일 디스크립터들을 확인하는 게 좋다.

5. `read(2)`, <tt>[[recv(2)]]</tt>, `write(2)`, <tt>[[send(2)]]</tt> 함수가 꼭 요청 데이터 모두를 읽는/쓰는 것은 *아니다*. 전체를 읽는다면/쓴다면 그건 트래픽 부하가 작고 스트림이 빠르기 때문이다. 그런데 항상 그렇지는 않다. 함수에서 한 바이트만 겨우 보내거나 받는 경우에 대처해야 한다.

6. 처리할 데이터가 정말 조금만 있는 경우가 아니면 절대 한번에 한 바이트씩 읽지/쓰지 마라. 버퍼를 최대한 채워서 데이터를 읽지/쓰지 않으면 매우 비효율적으로 된다. 아래 예의 버퍼들은 1024바이트지만 손쉽게 크기를 키울 수 있다.

7. `read(2)`, <tt>[[recv(2)]]</tt>, `write(2)`, <tt>[[send(2)]]</tt>, `select()` 호출이 `EINTR` 오류로 실패할 수 있으며 `read(2)`, <tt>[[recv(2)]]</tt>, `write(2)`, <tt>[[send(2)]]</tt> 호출이 실패해서 `errno`에 `EAGAIN`(`EWOULDBLOCK`)이 설정될 수 있다. 그런 경우들을 제대로 처리해야 한다. (위에서는 그러지 않았다.) 프로그램이 어떤 시그널도 받지 않는다면 아마 `EINTR`가 나오지 않을 것이다. 프로그램에서 논블로킹 I/O를 설정하지 않으면 `EAGAIN`이 나오지 않을 것이다.

8. 절대로 버퍼 길이를 0으로 해서 `read(2)`, <tt>[[recv(2)]]</tt>, `write(2)`, <tt>[[send(2)]]</tt>를 호출하지 마라.

9. 함수 `read(2)`, <tt>[[recv(2)]]</tt>, `write(2)`, <tt>[[send(2)]]</tt>가 7번에 나열한 것 외의 오류로 실패하거나 입력 함수들이 파일 끝을 나타내는 0을 반환하는 경우에는 그 파일 디스크립터를 `select()`로 다시 전달하지 *않아야 한다*. 위 예에서는 파일 디스크립터를 즉시 닫고서 -1로 설정해서 집합에 포함되는 걸 방지한다.

10. `select()`를 호출할 때마다 타임아웃 값을 설정해야 한다. 일부 운영 체제에서 그 구조체를 변경하기 때문이다. 하지만 `pselect()`에서는 타임아웃 구조체를 변경하지 않는다.

11. `select()`에서 파일 디스크립터 집합들을 변경하므로 루프 안에서 호출을 하는 경우라면 호출 전에 매번 집합들을 다시 설정해야 한다.

### usleep 에뮬레이션

<tt>[[usleep(3)]]</tt> 함수가 없는 시스템에서는 다음처럼 파일 디스크립터 없이 정해진 타임아웃으로 `select()`를 호출할 수 있다.

```c
struct timeval tv;
tv.tv_sec = 0;
tv.tv_usec = 200000;  /* 0.2 초 */
select(0, NULL, NULL, NULL, &tv);
```

다만 유닉스 시스템들에서만 동작이 보장된다.

## RETURN VALUE

성공 시 `select()`는 파일 디스크립터 집합들에 아직 있는 파일 디스크립터들의 총개수를 반환한다.

`select()`가 타임아웃 되면 반환 값이 0이 된다. 파일 디스크립터 집합이 모두 비어 있을 것이다. (하지만 어떤 시스템에서는 그렇지 않을 수도 있다.)

반환 값 -1은 오류를 나타내며 `errno`가 적절히 설정된다. 오류 경우에 반환된 집합들의 내용과 `struct timeout`의 내용은 규정돼 있지 않으므로 사용하지 말아야 한다. 다만 `pselect()`에서는 절대 `ntimeout`을 변경하지 않는다.

## NOTES

일반적으로 말해서 소켓을 지원하는 운영 체제들은 모두 `select()`도 지원한다. `select()`를 쓰면 초보 프로그래머가 스레드, 포크, IPC, 시그널, 메모리 공유 등을 이용해 복잡하게 해결하려 하는 여러 문제들을 이식성 있고 효율적인 방식으로 해결할 수 있다.

<tt>[[poll(2)]]</tt> 시스템 호출은 `select()`와 기능성이 같으며 파일 디스크립터들이 드문드문한 집합을 감시할 때 조금 더 효율적이다. 요즘은 널리 사용 가능하지만 역사적으로는 `select()`보다 이식성이 낮았다.

리눅스 전용인 <tt>[[epoll(7)]]</tt> API는 많은 파일 디스크립터들을 감시할 때 <tt>[[select(2)]]</tt> 및 <tt>[[poll(2)]]</tt>보다 효율적인 인터페이스를 제공한다.

## EXAMPLES

다음은 `select()`의 유용성을 더 제대로 보여 주는 예시이다. 한 TCP 포트에서 다른 포트로 전달을 하는 TCP 포워딩 프로그램이다.

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/select.h>
#include <string.h>
#include <signal.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <errno.h>

static int forward_port;

#undef max
#define max(x,y) ((x) > (y) ? (x) : (y))

static int
listen_socket(int listen_port)
{
    struct sockaddr_in addr;
    int lfd;
    int yes;

    lfd = socket(AF_INET, SOCK_STREAM, 0);
    if (lfd == -1) {
        perror("socket");
        return -1;
    }

    yes = 1;
    if (setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR,
            &yes, sizeof(yes)) == -1) {
        perror("setsockopt");
        close(lfd);
        return -1;
    }

    memset(&addr, 0, sizeof(addr));
    addr.sin_port = htons(listen_port);
    addr.sin_family = AF_INET;
    if (bind(lfd, (struct sockaddr *) &addr, sizeof(addr)) == -1) {
        perror("bind");
        close(lfd);
        return -1;
    }

    printf("accepting connections on port %d\n", listen_port);
    listen(lfd, 10);
    return lfd;
}

static int
connect_socket(int connect_port, char *address)
{
    struct sockaddr_in addr;
    int cfd;

    cfd = socket(AF_INET, SOCK_STREAM, 0);
    if (cfd == -1) {
        perror("socket");
        return -1;
    }

    memset(&addr, 0, sizeof(addr));
    addr.sin_port = htons(connect_port);
    addr.sin_family = AF_INET;

    if (!inet_aton(address, (struct in_addr *) &addr.sin_addr.s_addr)) {
        fprintf(stderr, "inet_aton(): bad IP address format\n");
        close(cfd);
        return -1;
    }

    if (connect(cfd, (struct sockaddr *) &addr, sizeof(addr)) == -1) {
        perror("connect()");
        shutdown(cfd, SHUT_RDWR);
        close(cfd);
        return -1;
    }
    return cfd;
}

#define SHUT_FD1 do {                                \
                     if (fd1 >= 0) {                 \
                         shutdown(fd1, SHUT_RDWR);   \
                         close(fd1);                 \
                         fd1 = -1;                   \
                     }                               \
                 } while (0)

#define SHUT_FD2 do {                                \
                     if (fd2 >= 0) {                 \
                         shutdown(fd2, SHUT_RDWR);   \
                         close(fd2);                 \
                         fd2 = -1;                   \
                     }                               \
                 } while (0)

#define BUF_SIZE 1024

int
main(int argc, char *argv[])
{
    int h;
    int fd1 = -1, fd2 = -1;
    char buf1[BUF_SIZE], buf2[BUF_SIZE];
    int buf1_avail = 0, buf1_written = 0;
    int buf2_avail = 0, buf2_written = 0;

    if (argc != 4) {
        fprintf(stderr, "Usage\n\tfwd <listen-port> "
                 "<forward-to-port> <forward-to-ip-address>\n");
        exit(EXIT_FAILURE);
    }

    signal(SIGPIPE, SIG_IGN);

    forward_port = atoi(argv[2]);

    h = listen_socket(atoi(argv[1]));
    if (h == -1)
        exit(EXIT_FAILURE);

    for (;;) {
        int ready, nfds = 0;
        ssize_t nbytes;
        fd_set readfds, writefds, exceptfds;

        FD_ZERO(&readfds);
        FD_ZERO(&writefds);
        FD_ZERO(&exceptfds);
        FD_SET(h, &readfds);
        nfds = max(nfds, h);

        if (fd1 > 0 && buf1_avail < BUF_SIZE)
            FD_SET(fd1, &readfds);
            /* 참고: 아래에서 fd1을 exceptfds에 추가할 때
               nfds를 갱신함. */
        if (fd2 > 0 && buf2_avail < BUF_SIZE)
            FD_SET(fd2, &readfds);

        if (fd1 > 0 && buf2_avail - buf2_written > 0)
            FD_SET(fd1, &writefds);
        if (fd2 > 0 && buf1_avail - buf1_written > 0)
            FD_SET(fd2, &writefds);

        if (fd1 > 0) {
            FD_SET(fd1, &exceptfds);
            nfds = max(nfds, fd1);
        }
        if (fd2 > 0) {
            FD_SET(fd2, &exceptfds);
            nfds = max(nfds, fd2);
        }

        ready = select(nfds + 1, &readfds, &writefds, &exceptfds, NULL);

        if (ready == -1 && errno == EINTR)
            continue;

        if (ready == -1) {
            perror("select()");
            exit(EXIT_FAILURE);
        }

        if (FD_ISSET(h, &readfds)) {
            socklen_t addrlen;
            struct sockaddr_in client_addr;
            int fd;

            addrlen = sizeof(client_addr);
            memset(&client_addr, 0, addrlen);
            fd = accept(h, (struct sockaddr *) &client_addr, &addrlen);
            if (fd == -1) {
                perror("accept()");
            } else {
                SHUT_FD1;
                SHUT_FD2;
                buf1_avail = buf1_written = 0;
                buf2_avail = buf2_written = 0;
                fd1 = fd;
                fd2 = connect_socket(forward_port, argv[3]);
                if (fd2 == -1)
                    SHUT_FD1;
                else
                    printf("connect from %s\n",
                            inet_ntoa(client_addr.sin_addr));

                /* 닫은 이전 파일 디스크립터들의 이벤트 무시. */
                continue;
            }
        }

        /* 주의: OOB 데이터를 일반 데이터 전에 읽을 것. */

        if (fd1 > 0 && FD_ISSET(fd1, &exceptfds)) {
            char c;

            nbytes = recv(fd1, &c, 1, MSG_OOB);
            if (nbytes < 1)
                SHUT_FD1;
            else
                send(fd2, &c, 1, MSG_OOB);
        }
        if (fd2 > 0 && FD_ISSET(fd2, &exceptfds)) {
            char c;

            nbytes = recv(fd2, &c, 1, MSG_OOB);
            if (nbytes < 1)
                SHUT_FD2;
            else
                send(fd1, &c, 1, MSG_OOB);
        }
        if (fd1 > 0 && FD_ISSET(fd1, &readfds)) {
            nbytes = read(fd1, buf1 + buf1_avail,
                      BUF_SIZE - buf1_avail);
            if (nbytes < 1)
                SHUT_FD1;
            else
                buf1_avail += nbytes;
        }
        if (fd2 > 0 && FD_ISSET(fd2, &readfds)) {
            nbytes = read(fd2, buf2 + buf2_avail,
                      BUF_SIZE - buf2_avail);
            if (nbytes < 1)
                SHUT_FD2;
            else
                buf2_avail += nbytes;
        }
        if (fd1 > 0 && FD_ISSET(fd1, &writefds) && buf2_avail > 0) {
            nbytes = write(fd1, buf2 + buf2_written,
                       buf2_avail - buf2_written);
            if (nbytes < 1)
                SHUT_FD1;
            else
                buf2_written += nbytes;
        }
        if (fd2 > 0 && FD_ISSET(fd2, &writefds) && buf1_avail > 0) {
            nbytes = write(fd2, buf1 + buf1_written,
                       buf1_avail - buf1_written);
            if (nbytes < 1)
                SHUT_FD2;
            else
                buf1_written += nbytes;
        }

        /* 데이터 쓰기가 데이터 읽기를 따라잡았는지 확인하자. */

        if (buf1_written == buf1_avail)
            buf1_written = buf1_avail = 0;
        if (buf2_written == buf2_avail)
            buf2_written = buf2_avail = 0;

        /* 한쪽이 연결을 닫았다. 반대쪽에선 완전히
           비울 때까지 쓰기를 계속하자. */

        if (fd1 < 0 && buf1_avail - buf1_written == 0)
            SHUT_FD2;
        if (fd2 < 0 && buf2_avail - buf2_written == 0)
            SHUT_FD1;
    }
    exit(EXIT_SUCCESS);
}
```

위 프로그램은 `telnet` 서버가 전송하는 OOB 신호 데이터를 포함해 대부분의 TCP 연결을 제대로 전달한다. 양방향 모두로 데이터 흐름이 있는 까다로운 문제를 잘 처리한다. <tt>[[fork(2)]]</tt> 호출을 해서 각 스트림에 스레드를 하나씩 쓰는 게 더 효율적이라 생각할 수도 있을 것이다. 하지만 실제 해 보면 생각보다 까다롭다. 또 <tt>[[fcntl(2)]]</tt>을 써서 논블로킹 I/O를 설정하는 걸 생각할 수 있다. 하지만 비효율적인 타임아웃 방식을 쓰게 되므로 역시 문제가 있다.

이 프로그램에서는 동시 연결 한 개만 처리하고 있지만 버퍼들의 연결 리스트를 두고 연결마다 하나씩 쓰게 하면 손쉽게 확장할 수 있다. 현재는 새 연결이 들어오면 현재 연결을 버리게 된다.

## SEE ALSO

`accept(2)`, `connect(2)`, <tt>[[poll(2)]]</tt>, `read(2)`, <tt>[[recv(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[send(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, `write(2)`, <tt>[[epoll(7)]]</tt>

----

2019-03-06
