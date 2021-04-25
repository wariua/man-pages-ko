## NAME

eventfd - 이벤트 알림을 위한 파일 디스크립터 만들기

## SYNOPSIS

```c
#include <sys/eventfd.h>

int eventfd(unsigned int initval, int flags);
```

## DESCRIPTION

`eventfd()`는 "eventfd 객체"를 생성하며, 이를 사용자 공간 응용에서 이벤트 대기/알림 메커니즘으로 사용하거나 커널에서 사용자 공간 응용에게 이벤트를 알리는 데 쓸 수 있다. 그 객체에는 커널에서 관리하는 부호 없는 64비트 정수 (`uint64_t`) 카운터가 있다. 인자 `initval`에 지정한 값으로 그 카운터를 초기화 한다.

반환 값으로 `eventfd()`는 새 파일 디스크립터를 반환하며 이를 이용해 eventfd 객체를 참조할 수 있다.

`flags`에 다음 값들을 비트 OR 해서 `eventfd()`의 동작 방식을 바꿀 수 있다.

`EFD_CLOEXEC` (리눅스 2.6.27부터)
:   새 파일 디스크립터에 'exec에서 닫기'(`FD_CLOEXEC`) 플래그를 설정한다. 이게 유용할 수 있는 이유에 대해선 <tt>[[open(2)]]</tt>의 `O_CLOEXEC` 플래그 설명을 보라.

`EFD_NONBLOCK` (리눅스 2.6.27부터)
:   새 파일 디스크립터가 가리키는 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)에 `O_NONBLOCK` 파일 상태 플래그를 설정한다. 이 플래그를 사용하면 같은 결과를 얻기 위해 <tt>[[fcntl(2)]]</tt>을 추가로 호출하지 않아도 된다.

`EFD_SEMAPHORE` (리눅스 2.6.30부터)
:   새 파일 디스크립터 읽기에 세마포어 같은 동작 방식을 제공한다. 아래 참고.

리눅스 버전 2.6.26까지는 `flags` 인자를 사용하지 않으며 0으로 지정해야 한다.

`eventfd()`가 반환한 파일 디스크립터에 다음 작업을 수행할 수 있다.

`read(2)`
:   `read(2)`가 성공할 때마다 8바이트 정수를 반환한다. 제공받은 버퍼의 크기가 8바이트보다 작으면 `read(2)`가 `EINVAL` 오류로 실패한다.

    `read(2)`가 반환하는 값은 호스트 바이트 순서, 즉 호스트 머신 원래의 정수 바이트 순서이다.

    `read(2)`의 동작 방식은 eventfd 카운터가 현재 0 아닌 값을 가지고 있는지 여부와 eventfd 파일 디스크립터 생성 시 `EFD_SEMAPHORE` 플래그를 지정했는지 여부에 따라 달라진다.

    * `EFD_SEMAPHORE`를 지정하지 않았고 eventfd 카운터의 값이 0이 아니면 `read(2)`가 그 값을 담은 8바이트를 반환하며 카운터 값이 0으로 재설정된다.

    * `EFD_SEMAPHORE`를 지정했으며 eventfd 카운터의 값이 0이 아니면 `read(2)`가 값 1을 담은 8바이트를 반환하며 카운터 값이 1만큼 줄어든다.

    * `read(2)` 호출 시점에 eventfd 카운터가 0이면 카운터가 0이 아니게 될 때까지 블록 한다. (그때 `read(2)`가 위 설명처럼 진행한다.) 파일 디스크립터를 논블록으로 만들었으면 `EAGAIN` 오류로 실패한다.

`write(2)`
:   `write(2)` 호출은 버퍼에 준 8바이트 정수 값을 카운터에 더한다. 카운터에 저장할 수 있는 최댓값은 가장 큰 부호 없는 64비트 정수에서 1을 뺀 것(즉 0xfffffffffffffffe)이다. 더한 결과가 그 최댓값을 초과하게 될 것 같으면 그 파일 디스크립터에 `read(2)`가 이뤄질 때까지 `write(2)`가 블록 한다. 파일 디스크립터를 논블록으로 만들었으면 `EAGAIN` 오류로 실패한다.

    제공 버퍼의 크기가 8바이트보다 작거나 값 0xffffffffffffffff를 쓰려고 시도하면 `write(2)`가 `EINVAL` 오류로 실패한다.

<tt>[[poll(2)]]</tt>, <tt>[[select(2)]]</tt> (기타 유사 함수)
:   반환되는 파일 디스크립터가 다음과 같이 <tt>[[poll(2)]]</tt> (<tt>[[epoll(7)]]</tt>도 비슷함) 및 <tt>[[select(2)]]</tt>를 지원한다.

    * 카운터 값이 0보다 크면 파일 디스크립터가 읽기 가능하다. (<tt>[[select(2)]]</tt>의 `readfds` 인자, <tt>[[poll(2)]]</tt>의 `POLLIN` 플래그)

    * 블록 되지 않고 적어도 "1" 값을 기록할 수 있으면 파일 디스크립터가 쓰기 가능하다. (<tt>[[select(2)]]</tt>의 `writefds` 인자, <tt>[[poll(2)]]</tt>의 `POLLOUT` 플래그)

    * 카운터 값 오버플로를 감지한 경우 <tt>[[select(2)]]</tt>는 파일 디스크립터가 읽기 가능하면서 동시에 쓰기 가능하다고 표시하며 <tt>[[poll(2)]]</tt>은 `POLLERR` 이벤트를 반환한다. 위에서 언급한 것처럼 `write(2)`는 절대 카운터를 넘치게 할 수 없다. 하지만 KAIO 서브시스템에서 eventfd "알림 발송"을 2^64번 수행한다면 넘침이 발생할 수 있다. (이론적으로 가능한 것이고 실제로는 가능성이 낮다.) 넘침이 발생했으면 `read(2)`가 가장 큰 `uint64_t` 값(즉 0xffffffffffffffff)을 반환하게 된다.

    eventfd 파일 디스크립터는 <tt>[[pselect(2)]]</tt>와 <tt>[[ppoll(2)]]</tt> 같은 다른 파일 디스크립터 다중화 API도 지원한다.

<tt>[[close(2)]]</tt>
:   파일 디스크립터가 더이상 필요하지 않으면 닫아야 한다. 동일 eventfd 객체에 연계된 모든 파일 디스크립터가 닫혔을 때 커널이 그 객체의 자원을 해제한다.

<tt>[[fork(2)]]</tt>로 생성된 자식은 `eventfd()`로 만든 파일 디스크립터의 사본을 물려받는다. 복제된 파일 디스크립터는 동일한 eventfd 객체에 연계되어 있다. 'exec에서 닫기' 플래그를 설정하지 않았으면 <tt>[[execve(2)]]</tt>를 거치면서 `eventfd()`로 만든 파일 디스크립터가 유지된다.

## RETURN VALUE

성공 시 `eventfd()`는 새 eventfd 파일 디스크립터를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   지원하지 않는 값을 `flags`에 지정했다.

`EMFILE`
:   열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다.

`ENFILE`
:   열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.

`ENODEV`
:   (내부적으로 쓰는) 익명 아이노드 장치를 마운트 할 수 없었다.

`ENOMEM`
:   새 eventfd 파일 디스크립터를 생성하기에 메모리가 충분하지 않았다.

## VERSIONS

리눅스 커널 2.6.22부터 `eventfd()`가 사용 가능하다. glibc 버전 2.8부터 잘 동작하는 지원을 제공한다. 리눅스 커널 2.6.27부터 `eventfd2()` 시스템 호출(NOTES 참고)이 사용 가능하다. 버전 2.9부터 glibc의 `eventfd()` 래퍼가 커널 지원 시 `eventfd2()` 시스템 호출을 이용한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `eventfd()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`eventfd()`와 `eventfd2()`는 리눅스 전용이다.

## NOTES

응용에서 파이프(<tt>[[pipe(2)]]</tt> 참고)를 이벤트 전달용으로만 쓰는 모든 경우에서 파이프 대신 eventfd 파일 디스크립터를 사용할 수 있다. eventfd의 커널 오버헤드가 파이프보다 훨씬 낮으며 파일 디스크립터가 한 개만 필요하다. (파이프에서는 두 개가 필요하다.)

커널에서 사용 시 eventfd 파일 디스크립터는 커널에서 사용자 공간으로의 가교가 될 수 있다. 그래서 예를 들어 KAIO(커널 AIO) 같은 기능에서 어떤 동작이 완료되었을 때 파일 디스크립터로 알릴 수 있다.

eventfd 파일 디스크립터의 핵심은 여느 파일 디스크립터들처럼 <tt>[[select(2)]]</tt>나 <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>을 이용해 상태를 확인할 수 있다는 점이다. 그래서 응용에서 "전통적" 파일의 준비 상태와 eventfd 인터페이스를 지원하는 커널 메커니즘의 준비 상태를 동시에 확인할 수 있다. (`eventfd()` 인터페이스가 없으면 그 메커니즘을 <tt>[[select(2)]]</tt>나 <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>을 통해 다중화 할 수 없었을 것이다.)

프로세스의 `/proc/[pid]/fdinfo` 디렉터리 내의 대응하는 파일 디스크립터 항목을 통해 eventfd 카운터의 현재 값을 볼 수 있다. 자세한 내용은 <tt>[[proc(5)]]</tt> 참고.

### C 라이브러리/커널 차이

기반 시스템 호출이 두 가지 있다. `eventfd()`와 더 최신인 `eventfd2()`이다. 앞쪽 시스템 호출은 `flags` 인자를 구현하지 않는다. 뒤쪽 시스템 호출은 위에 기술한 `flags` 값들을 구현한다. glibc 래퍼 함수에서 가능한 경우 `eventfd2()`를 사용한다.

### glibc 추가 기능

GNU C 라이브러리에서 새로운 타입 하나와 함수 두 개를 정의하고 있는데, 이는 eventfd 파일 디스크립터 읽기와 쓰기의 세부 사항을 좀 추상화 해 보려는 것이다.

```c
typedef uint64_t eventfd_t;

int eventfd_read(int fd, eventfd_t *value);
int eventfd_write(int fd, eventfd_t value);
```

이 함수들은 eventfd 파일 디스크립터에 읽기 및 쓰기 동작을 수행하며, 정확한 수의 바이트를 전송했으면 0을 반환하고 아니면 -1을 반환한다.

## EXAMPLES

아래 프로그램은 eventfd 파일 디스크립터를 만들고서 분기하여 자식 프로세스를 만든다. 부모가 잠시 잠드는 동안 자식이 프로그램 명령행 인자로 받은 정수들 각각을 eventfd 파일 디스크립터에 쓴다. 그리고 잠에서 깬 부모가 eventfd 파일 디스크립터에서 읽기를 한다.

다음 셸 세션은 프로그램 실행 예를 보여 준다.

```text
$ ./a.out 1 2 4 7 14
Child writing 1 to efd
Child writing 2 to efd
Child writing 4 to efd
Child writing 7 to efd
Child writing 14 to efd
Child completed write loop
Parent about to read
Parent read 28 (0x1c) from efd
```

### 프로그램 소스

```c
#include <sys/eventfd.h>
#include <unistd.h>
#include <inttypes.h>           /* PRIu64 및 PRIx64 정의 */
#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>             /* uint64_t 정의 */

#define handle_error(msg) \
    do { perror(msg); exit(EXIT_FAILURE); } while (0)

int
main(int argc, char *argv[])
{
    int efd;
    uint64_t u;
    ssize_t s;

    if (argc < 2) {
        fprintf(stderr, "Usage: %s <num>...\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    efd = eventfd(0, 0);
    if (efd == -1)
        handle_error("eventfd");

    switch (fork()) {
    case 0:
        for (int j = 1; j < argc; j++) {
            printf("Child writing %s to efd\n", argv[j]);
            u = strtoull(argv[j], NULL, 0);
                    /* strtoull()에 다양한 기수 가능 */
            s = write(efd, &u, sizeof(uint64_t));
            if (s != sizeof(uint64_t))
                handle_error("write");
        }
        printf("Child completed write loop\n");

        exit(EXIT_SUCCESS);

    default:
        sleep(2);

        printf("Parent about to read\n");
        s = read(efd, &u, sizeof(uint64_t));
        if (s != sizeof(uint64_t))
            handle_error("read");
        printf("Parent read %"PRIu64" (%#"PRIx64") from efd\n", u, u);
        exit(EXIT_SUCCESS);

    case -1:
        handle_error("fork");
    }
}
```

## SEE ALSO

<tt>[[futex(2)]]</tt>, <tt>[[pipe(2)]]</tt>, <tt>[[poll(2)]]</tt>, `read(2)`, <tt>[[select(2)]]</tt>, <tt>[[signalfd(2)]]</tt>, <tt>[[timerfd_create(2)]]</tt>, `write(2)`, <tt>[[epoll(7)]]</tt>, <tt>[[sem_overview(7)]]</tt>

----

2021-03-22
