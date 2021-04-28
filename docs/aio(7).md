## NAME

aio - POSIX 비동기 I/O 소개

## DESCRIPTION

POSIX 비동기 I/O(AIO) 인터페이스를 응용에서 이용하여 비동기적으로 (즉 배경에서) 수행되는 I/O 동작을 한 개 이상 개시할 수 있다. I/O 동작 완료에 대한 알림을 응용에서 원하는 다양한 방식으로 받을 수 있다. 시그널 전달로, 새로운 스레드로, 또는 알림을 받지 않을 수도 있다.

POSIX AIO 인터페이스는 다음 함수들로 이뤄져 있다.

<tt>[[aio_read(3)]]</tt>
:   읽기 요청을 큐에 넣는다. `read(2)`의 비동기 형태다.

<tt>[[aio_write(3)]]</tt>
:   쓰기 요청을 큐에 넣는다. `write(2)`의 비동기 형태다.

<tt>[[aio_fsync(3)]]</tt>
:   파일 디스크립터 상의 I/O 동작들에 대한 동기화 요청을 큐에 넣는다. <tt>[[fsync(2)]]</tt> 및 <tt>[[fdatasync(2)]]</tt>의 비동기 형태다.

<tt>[[aio_error(3)]]</tt>
:   큐에 넣은 I/O 요청의 오류 상태를 얻는다.

<tt>[[aio_return(3)]]</tt>
:   완료된 I/O 요청의 반환 상태를 얻는다.

<tt>[[aio_suspend(3)]]</tt>
:   지정한 I/O 요청이 하나 이상 완료될 때까지 호출자를 멈춘다.

<tt>[[aio_cancel(3)]]</tt>
:   지정한 파일 디스크립터 상의 미처리 I/O 요청들을 취소 시도한다.

<tt>[[lio_listio(3)]]</tt>
:   한 번의 함수 호출로 여러 I/O 요청을 큐에 넣는다.

`aiocb`(비동기 I/O 제어 블록) 구조체가 I/O 동작을 제어하는 매개변수들을 정의한다. 위에 나열한 함수 모두에서 이 타입의 인자를 쓴다. 다음 형태의 구조체다.

```c
#include <aiocb.h>

struct aiocb {
    /* 이 필드들의 순서는 구현에 따라 다름 */

    int             aio_fildes;     /* 파일 디스크립터 */
    off_t           aio_offset;     /* 파일 오프셋 */
    volatile void  *aio_buf;        /* 버퍼 위치 */
    size_t          aio_nbytes;     /* 전송 길이 */
    int             aio_reqprio;    /* 요청 우선순위 */
    struct sigevent aio_sigevent;   /* 알림 방법 */
    int             aio_lio_opcode; /* 수행할 동작.
                                       lio_listio()에서만 */

    /* 여러 구현 내부용 필드 생략 */
};

/* 'aio_lio_opcode'의 동작 코드 */

enum { LIO_READ, LIO_WRITE, LIO_NOP };
```

이 구조체의 필드들은 다음과 같다.

`aio_fildes`
:   I/O 동작을 수행할 파일 디스크립터.

`aio_offset`
:   I/O 동작을 수행할 파일 오프셋.

`aio_buf`
:   읽기 또는 쓰기 동작에서 데이터 이동에 사용하는 버퍼.

`aio_nbytes`
:   `aio_buf`가 가리키는 버퍼의 크기.

`aio_reqprio`
:   이 필드에 지정한 값을 호출 스레드의 실시간 우선순위에서 빼서 I/O 요청의 실행 우선순위를 정한다. (<tt>[[pthread_setschedparam(3)]]</tt> 참고.) 지정하는 값이 0과 `sysconf(_SC_AIO_PRIO_DELTA_MAX)` 반환 값 사이여야 한다. 파일 동기화 동작에서는 이 필드가 무시된다.

`aio_sigevent`
:   비동기 I/O 동작이 완료됐을 때 호출자가 알림을 받는 방법을 지정하는 구조체다. `aio_sigevent.sigev_notify`에 가능한 값은 `SIGEV_NONE`, `SIGEV_SIGNAL`, `SIGEV_THREAD`다. 자세한 내용은 <tt>[[sigevent(7)]]</tt> 참고.

`aio_lio_opcode`
:   수행할 동작 종류. <tt>[[lio_listio(3)]]</tt>에서만 사용.

위에 나열한 표준 함수들에 더해서 GNU C 라이브러리에서는 POSIX AIO API에 대한 다음 확장을 제공한다.

<tt>[[aio_init(3)]]</tt>
:   glibc POSIX AIO 구현의 동작을 조정하는 매개변수들을 설정한다.

## ERRORS

`EINVAL`
:   `aiocb` 구조체의 `aio_reqprio` 필드가 0보다 작거나 `sysconf(_SC_AIO_PRIO_DELTA_MAX)` 호출이 반환한 한계치보다 크다.

## VERSIONS

glibc 버전 2.1부터 POSIX AIO 인터페이스를 제공한다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

제어 블록 버퍼를 사용하기 전에 0으로 채우는 게 좋다. (`memset(3)` 참고.) I/O 동작이 진행 중인 동안 제어 블록 버퍼와 `aio_buf`가 가리키는 버퍼가 변경되어선 안 된다. I/O 동작이 완료될 때까지 그 버퍼들이 유효한 상태로 유지돼야 한다.

같은 `aiocb` 구조체를 써서 비동기 읽기 내지 쓰기 동작들을 동시에 할 때의 결과는 규정돼 있지 않다.

현재 리눅스의 POSIX AIO 구현은 사용자 공간에서 glibc가 제공한다. 그래서 여러 제약이 있는데, 가장 눈에 띄는 건 I/O 동작을 수행하기 위해 여러 스레드를 유지하는 게 비용이 크고 확장성이 떨어진다는 점이다. 얼마 전부터 커널에서 상태 머신 기반으로 비동기 I/O를 구현하는 작업이 진행 중이긴 한데 (<tt>[[io_submit(2)]]</tt>, <tt>[[io_setup(2)]]</tt>, <tt>[[io_cancel(2)]]</tt>, <tt>[[io_destroy(2)]]</tt>, <tt>[[io_getevents(2)]]</tt> 참고) 그 커널 시스템 호출들을 이용해 POSIX AIO 구현을 완전히 재구현할 수 있는 수준까지는 아직 오지 못했다.

## EXAMPLES

아래 프로그램에서는 명령행 인자에 지명한 파일들 각각을 열어서 얻은 파일 디스크립터에 <tt>[[aio_read(3)]]</tt>로 요청을 넣는다. 그러고서 루프를 돌면서 아직 진행 중인 I/O 동작 각각을 <tt>[[aio_error(3)]]</tt>를 이용해 주기적으로 확인한다. I/O 요청 각각은 시그널 전달로 알림을 주도록 설정한다. 모든 I/O 요청이 완료된 후에 <tt>[[aio_return(3)]]</tt>으로 상태를 가져온다.

`SIGQUIT` 시그널(Control-\ 입력으로 생성)을 주면 프로그램에서 <tt>[[aio_cancel(3)]]</tt>로 미처리 요청 각각의 취소를 요청한다.

다음은 이 프로그램을 실행한 결과 예시다. 이 예에서는 표준 입력에 대한 요청을 두 개 하고, "abc"와 "x"를 담은 두 입력 행으로 그 요청을 충족시킨다.

```text
$ ./a.out /dev/stdin /dev/stdin
opened /dev/stdin on descriptor 3
opened /dev/stdin on descriptor 4
aio_error():
    for request 0 (descriptor 3): In progress
    for request 1 (descriptor 4): In progress
abc
I/O completion signal received
aio_error():
    for request 0 (descriptor 3): I/O succeeded
    for request 1 (descriptor 4): In progress
aio_error():
    for request 1 (descriptor 4): In progress
x
I/O completion signal received
aio_error():
    for request 1 (descriptor 4): I/O succeeded
All I/O requests completed
aio_return():
    for request 0 (descriptor 3): 4
    for request 1 (descriptor 4): 2
```

### 프로그램 소스

```c
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>
#include <aio.h>
#include <signal.h>

#define BUF_SIZE 20     /* 읽기 동작을 위한 버퍼 크기 */

#define errExit(msg) do { perror(msg); exit(EXIT_FAILURE); } while (0)

struct ioRequest {      /* I/O 요청 추적을 위해 응용 자체에서
                           정의하는 구조체 */
    int           reqNum;
    int           status;
    struct aiocb *aiocbp;
};

static volatile sig_atomic_t gotSIGQUIT = 0;
                        /* SIGQUIT이 전달되면 미처리 I/O 요청을
                           모두 취소하려고 시도 */

static void             /* SIGQUIT 핸들러 */
quitHandler(int sig)
{
    gotSIGQUIT = 1;
}

#define IO_SIGNAL SIGUSR1   /* I/O 완료를 알리는 데 쓸 시그널 */

static void                 /* I/O 완료 시그널 핸들러 */
aioSigHandler(int sig, siginfo_t *si, void *ucontext)
{
    if (si->si_code == SI_ASYNCIO) {
        write(STDOUT_FILENO, "I/O completion signal received\n", 31);

        /* 대응하는 ioRequest 구조체를 다음과 같이 얻을 수 있음:
               struct ioRequest *ioReq = si->si_value.sival_ptr;
           그리고 파일 디스크립터를 다음을 통해 얻을 수 있음:
               ioReq->aiocbp->aio_fildes */
    }
}

int
main(int argc, char *argv[])
{
    struct sigaction sa;
    int s;
    int numReqs;        /* 큐에 넣은 I/O 요청 총 개수 */
    int openReqs;       /* 아직 진행 중인 I/O 요청 개수 */

    if (argc < 2) {
        fprintf(stderr, "Usage: %s <pathname> <pathname>...\n",
                argv[0]);
        exit(EXIT_FAILURE);
    }

    numReqs = argc - 1;

    /* 배열 할당하기. */

    struct ioRequest *ioList = calloc(numReqs, sizeof(*ioList));
    if (ioList == NULL)
        errExit("calloc");

    struct aiocb *aiocbList = calloc(numReqs, sizeof(*aiocbList));
    if (aiocbList == NULL)
        errExit("calloc");

    /* SIGQUIT 및 I/O 완료 시그널을 위한 핸들러 설정하기. */

    sa.sa_flags = SA_RESTART;
    sigemptyset(&sa.sa_mask);

    sa.sa_handler = quitHandler;
    if (sigaction(SIGQUIT, &sa, NULL) == -1)
        errExit("sigaction");

    sa.sa_flags = SA_RESTART | SA_SIGINFO;
    sa.sa_sigaction = aioSigHandler;
    if (sigaction(IO_SIGNAL, &sa, NULL) == -1)
        errExit("sigaction");

    /* 명령행에 지정한 파일 각각을 열고, 그렇게 얻은 파일
       디스크립터에서 읽기 요청을 큐에 넣기. */

    for (int j = 0; j < numReqs; j++) {
        ioList[j].reqNum = j;
        ioList[j].status = EINPROGRESS;
        ioList[j].aiocbp = &aiocbList[j];

        ioList[j].aiocbp->aio_fildes = open(argv[j + 1], O_RDONLY);
        if (ioList[j].aiocbp->aio_fildes == -1)
            errExit("open");
        printf("opened %s on descriptor %d\n", argv[j + 1],
                ioList[j].aiocbp->aio_fildes);

        ioList[j].aiocbp->aio_buf = malloc(BUF_SIZE);
        if (ioList[j].aiocbp->aio_buf == NULL)
            errExit("malloc");

        ioList[j].aiocbp->aio_nbytes = BUF_SIZE;
        ioList[j].aiocbp->aio_reqprio = 0;
        ioList[j].aiocbp->aio_offset = 0;
        ioList[j].aiocbp->aio_sigevent.sigev_notify = SIGEV_SIGNAL;
        ioList[j].aiocbp->aio_sigevent.sigev_signo = IO_SIGNAL;
        ioList[j].aiocbp->aio_sigevent.sigev_value.sival_ptr =
                                &ioList[j];

        s = aio_read(ioList[j].aiocbp);
        if (s == -1)
            errExit("aio_read");
    }

    openReqs = numReqs;

    /* 루프 돌면서 I/O 요청 상태 확인하기. */

    while (openReqs > 0) {
        sleep(3);       /* 확인 간격 */

        if (gotSIGQUIT) {

            /* SIGQUIT 수신 시 미처리 I/O 요청 각각을 취소 시도하고,
               취소 요청에서 반환 받은 상태를 표시하기. */

            printf("got SIGQUIT; canceling I/O requests: \n");

            for (int j = 0; j < numReqs; j++) {
                if (ioList[j].status == EINPROGRESS) {
                    printf("    Request %d on descriptor %d:", j,
                            ioList[j].aiocbp->aio_fildes);
                    s = aio_cancel(ioList[j].aiocbp->aio_fildes,
                            ioList[j].aiocbp);
                    if (s == AIO_CANCELED)
                        printf("I/O canceled\n");
                    else if (s == AIO_NOTCANCELED)
                        printf("I/O not canceled\n");
                    else if (s == AIO_ALLDONE)
                        printf("I/O all done\n");
                    else
                        perror("aio_cancel");
                }
            }

            gotSIGQUIT = 0;
        }

        /* 아직 진행 중인 I/O 요청 각각의 상태 확인하기. */

        printf("aio_error():\n");
        for (int j = 0; j < numReqs; j++) {
            if (ioList[j].status == EINPROGRESS) {
                printf("    for request %d (descriptor %d): ",
                        j, ioList[j].aiocbp->aio_fildes);
                ioList[j].status = aio_error(ioList[j].aiocbp);

                switch (ioList[j].status) {
                case 0:
                    printf("I/O succeeded\n");
                    break;
                case EINPROGRESS:
                    printf("In progress\n");
                    break;
                case ECANCELED:
                    printf("Canceled\n");
                    break;
                default:
                    perror("aio_error");
                    break;
                }

                if (ioList[j].status != EINPROGRESS)
                    openReqs--;
            }
        }
    }

    printf("All I/O requests completed\n");

    /* 모든 I/O 요청의 상태 반환 값 확인하기. */

    printf("aio_return():\n");
    for (int j = 0; j < numReqs; j++) {
        ssize_t s;

        s = aio_return(ioList[j].aiocbp);
        printf("    for request %d (descriptor %d): %zd\n",
                j, ioList[j].aiocbp->aio_fildes, s);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[io_cancel(2)]]</tt>, <tt>[[io_destroy(2)]]</tt>, <tt>[[io_getevents(2)]]</tt>, <tt>[[io_setup(2)]]</tt>, <tt>[[io_submit(2)]]</tt>, <tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_error(3)]]</tt>, <tt>[[aio_init(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>

"Asynchronous I/O Support in Linux 2.5", Bhattacharya, Pratt, Pulavarty, and Morgan, Proceedings of the Linux SYmposium, 2003, <https://www.kernel.org/doc/ols/2003/ols2003-pages-351-366.pdf>

----

2021-03-22
