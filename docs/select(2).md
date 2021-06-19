## NAME

select, pselect, FD_CLR, FD_ISSET, FD_SET, FD_ZERO - 동기적 I/O 다중화

## SYNOPSIS

```c
#include <sys/select.h>

int select(int nfds, fd_set *restrict readfds,
           fd_set *restrict writefds, fd_set *restrict exceptfds,
           struct timeval *restrict timeout);

void FD_CLR(int fd, fd_set *set);
int  FD_ISSET(int fd, fd_set *set);
void FD_SET(int fd, fd_set *set);
void FD_ZERO(fd_set *set);

int pselect(int nfds, fd_set *restrict readfds,
            fd_set *restrict writefds, fd_set *restrict exceptfds,
            const struct timespec *restrict timeout,
            const sigset_t *restrict sigmask);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pselect()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`select()`를 이용해 프로그램에서 여러 파일 디스크립터들을 감시하고 그 중 하나 이상이 어떤 유형의 I/O 동작에 "준비" 상태(가령, 입력 가능)가 될 때까지 대기할 수 있다. 해당 I/O 동작(가령 <tt>[[read(2)]]</tt>나 충분히 작은 <tt>[[write(2)]]</tt>)을 블록킹 없이 수행하는 게 가능하면 파일 디스크립터가 준비 상태라고 본다.

`select()`에서는 `FD_SETSIZE`보다 작은 파일 디스크립터 번호만 감시할 수 있다. <tt>[[poll(2)]]</tt>과 <tt>[[epoll(7)]]</tt>에는 그런 제한이 없다. BUGS 참고.

### 파일 디스크립터 집합

`select()`의 핵심 인자는 (`fd_set` 타입으로 선언된) 파일 디스크립터 "집합" 세 가지다. 이를 이용해 지정한 파일 디스크립터 집합에 대해 세 가지 종류의 이벤트를 기다릴 수 있다. 해당 이벤트 종류에 대해 감시할 파일 디스크립터가 없으면 각 `fd_set` 인자를 NULL로 지정할 수도 있다.

**주의**: 반환 시 각 파일 디스크립터 집합은 현재 "준비" 상태인 파일 디스크립터를 나타내도록 변경되어 있다. 따라서 루프에서 `select()`를 쓰는 경우 호출 전마다 집합들을 *다시 초기화해 줘야 한다*. `fd_set` 인자를 값-결과 인자로 구현한 것은 설계 오류이며 <tt>[[poll(2)]]</tt>과 <tt>[[epoll(7)]]</tt>에서는 그 방식을 피하고 있다.

다음 매크로들을 써서 파일 디스크립터 집합의 내용물을 조작할 수 있다.

`FD_ZERO()`
:   이 매크로는 `set`을 비운다 (모든 파일 디스크립터를 제거한다). 파일 디스크립터 집합을 초기화하는 첫 단계로 사용해야 한다.

`FD_SET()`
:   이 매크로는 `set`에 파일 디스크립터 `fd`를 추가한다. 이미 집합에 있는 파일 디스크립터를 추가하는 동작은 no-op이며 오류가 발생하지 않는다.

`FD_CLR()`
:   이 매크로는 `set`에서 파일 디스크립터 `fd`를 제거한다. 집합에 없는 파일 디스크립터를 제거하는 것은 no-op이며 오류가 발생하지 않는다.

`FD_ISSET()`
:   `select()`는 아래 설명하는 규칙에 따라 집합들의 내용물을 변경한다. `select()` 호출 다음에 `FD_ISSET()` 매크로를 써서 집합에 여전히 파일 디스크립터가 있는지 확인할 수 있다. `set`에 파일 디스크립터 `fd`가 있으면 `FD_ISSET()`이 0 아닌 값을 반환하고, 없으면 0을 반환한다.

### 인자

`select()`의 인자는 다음과 같다.

`readfds`
:   이 집합의 파일 디스크립터들이 읽기 준비 상태인지 살펴본다. 파일 디스크립터가 읽기 준비 상태라는 것은 읽기 동작이 블록 하지 않게 된다는 것이다. 특히 파일 끝에서도 파일 디스크립터가 준비 상태이다.

    `select()` 반환 후 `readfds`에선 읽기 준비 상태가 아닌 모든 파일 디스크립터가 비워져 있게 된다.

`writefds`
:   이 집합의 파일 디스크립터들이 쓰기 준비 상태인지 살펴본다. 파일 디스크립터가 쓰기 준비 상태라는 것은 쓰기 동작이 블록 하지 않게 된다는 것이다. 하지만 파일 디스크립터가 쓰기 가능으로 표시된 경우에도 쓰기를 크게 하면 여전히 블록 될 수 있다.

    `select()` 반환 후 `writefds`에선 쓰기 준비 상태가 아닌 모든 파일 디스크립터가 비워져 있게 된다.

`exceptfds`
:   이 집합의 파일 디스크립터들이 "예외 상황"인지 살펴본다. 예외 상황에 대한 예는 <tt>[[poll(2)]]</tt>의 `POLLPRI` 설명을 보라.

    `select()` 반환 후 `exceptfds`에선 예외 상황이 발생하지 않은 모든 파일 디스크립터가 비워져 있게 된다.

`nfds`
:   이 인자는 세 집합에서 번호가 가장 높은 파일 디스크립터에 1을 더한 값으로 설정해야 한다. 각 집합에서 파일 디스크립터가 표시돼 있는지를 그 제한치까지 확인한다. (하지만 BUGS 참고.)

`timeout`
:   `timeout` 인자는 (아래에 있는) `timeval` 구조체이며, 파일 디스크립터가 준비 상태가 되기까지 `select()`가 블록 상태로 기다려야 하는 시간을 지정한다. 다음 어느 경우라도 해당할 때까지 호출이 블록 하게 된다.

    * 파일 디스크립터가 준비 상태가 된다.

    * 호출이 시그널 핸들러에 의해 중단된다.

    * 타임아웃이 만료된다.

    참고로 `timeout` 시간을 시스템 클럭 해상도에 따라 올림 하게 되며 커널 스케줄링 지연도 있기 때문에 지정한 블록 시간을 약간 넘길 수도 있다.

    `timeval` 구조체의 두 필드가 모두 0이면 `select()`가 즉시 반환한다. (폴링에 유용하다.)

    `timeout`을 NULL로 지정하면 파일 디스크립터가 준비 상태가 되길 기다리며 `select()`가 무한정 블록 한다.

### `pselect()`

`pselect()` 시스템 호출을 이용해 파일 디스크립터가 준비 상태가 되거나 시그널을 잡을 때까지 안전하게 대기할 수 있다.

`select()`와 `pselect()`의 동작은 다음 세 가지 차이를 빼면 동일하다.

* `select()`에서 쓰는 타임아웃은 `struct timeval`(초와 마이크로초)이지만 `pselect()`에서 쓰는 건 `struct timespec`(초와 나노초)이다.

* `select()`에서는 남은 시간이 얼마인지 나타내기 위해 `timeout` 인자를 갱신할 수도 있다. `pselect()`에서는 그 인자를 바꾸지 않는다.

* `select()`에는 `sigmask` 인자가 없고, 그래서 `sigmask`를 NULL로 해서 호출한 `pselect()`처럼 동작한다.

`sigmask`는 시그널 마스크(<tt>[[sigprocmask(2)]]</tt> 참고)에 대한 포인터다. NULL이 아닌 경우 `pselect()`에서는 먼저 현재 시그널 마스크를 `sigmask`가 가리키는 마스크로 교체하고, "select" 동작을 하고서, 원래 시그널 마스크를 복원한다. (`sigmask`가 NULL이면 `pselect()` 호출 동안 시그널 마스크가 변경되지 않는다.)

`timeout` 인자의 정밀도 차이를 제외하면 다음 `pselect()` 호출은

```c
ready = pselect(nfds, &readfds, &writefds, &exceptfds,
                timeout, &sigmask);
```

다음 호출들을 *원자적으로* 실행하는 것과 동등하다.

```c
sigset_t origmask;

pthread_sigmask(SIG_SETMASK, &sigmask, &origmask);
ready = select(nfds, &readfds, &writefds, &exceptfds, timeout);
pthread_sigmask(SIG_SETMASK, &origmask, NULL);
```

`pselect()`가 필요한 이유는 시그널과 파일 디스크립터 상태 변화를 함께 기다리려 할 때 경쟁 조건을 막으려면 원자적 검사가 필요하기 때문이다. (시그널 핸들러에서 전역 플래그를 설정하고 반환한다고 하자. 그 전역 플래그를 검사한 다음에 `select()` 호출을 한다고 할 때, 검사 후이자 호출 전에 시그널이 도착한다면 호출이 무한정 멈춰 있을 수도 있다. 반면 `pselect()`를 쓰는 경우에는 일단 시그널들을 막아 두고, 들어온 시그널들을 처리한 다음에 원하는 `sigmask`로 `pselect()`를 호출하여 경쟁을 피할 수 있다.)

### 타임아웃

`select()`의 `timeout` 인자는 다음 타입의 구조체다.

```c
struct timeval {
    time_t      tv_sec;         /* 초 */
    suseconds_t tv_usec;        /* 마이크로초 */
};
```

대응하는 `pselect()` 인자는 다음 타입이다.

```c
struct timespec {
    time_t      tv_sec;         /* 초 */
    long        tv_nsec;        /* 나노초 */
};
```

리눅스에서는 잠자지 않은 시간의 양을 반영하도록 `timeout`을 변경한다. 하지만 대부분의 다른 구현들에서는 그러지 않는다. (POSIX.1에서는 어느 쪽 동작이든 허용한다.) 이 때문에 `timeout`을 읽는 리눅스 코드를 다른 운영 체제로 이식할 때, 그리고 루프에서 `struct timeval`을 재설정하지 않고 `select()` 여러 번에 재사용하는 코드를 리눅스로 이식할 때 문제가 생긴다. `select()` 반환 후에 `timeout`의 값은 규정돼 있지 않다고 보면 된다.

## RETURN VALUE

성공 시 `select()`와 `pselect()`는 반환되는 세 디스크립터 집합에 담긴 파일 디스크립터들의 수를 (즉 `readfds`, `writefds`, `exceptfds`에 설정된 비트들의 총개수를) 반환한다. 파일 디스크립터가 준비 상태가 되기 전에 타임아웃이 만료됐다면 반환 값이 0일 수 있다.

오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다. 그 경우 파일 디스크립터 집합들은 변경되지 않으며 `timeout`은 규정되지 않은 상태가 된다.

## ERRORS

`EBADF`
:   한 집합에 유효하지 않은 파일 디스크립터가 있다. (아마 이미 닫혔거나 오류가 발생했던 파일 디스크립터일 것이다.) 하지만 BUGS 참고.

`EINTR`
:   시그널을 잡았다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `nfds`가 음수이거나 `RLIMIT_NOFILE` 자원 제한값(<tt>[[getrlimit(2)]]</tt> 참고)을 초과한다.

`EINVAL`
:   `timeout`에 담긴 값이 유효하지 않다.

`ENOMEM`
:   내부 테이블을 위한 메모리를 할당할 수 없다.

## VERSIONS

리눅스 커널 2.6.16에서 `pselect()`가 추가되었다. 그 전에는 glibc에서 `pselect()`를 에뮬레이션 했다. (하지만 BUGS 참고.)

## CONFORMING TO

`select()`는 POSIX.1-2001, POSIX.1-2008, 4.4BSD를 (4.2BSD에서 `select()`가 처음 등장) 준수한다. BSD 소켓 계층 복제 형태를 지원하는 BSD 외 시스템들(시스템 V 계열 포함)과의 사이에서 일반적으로 서로 이식 가능하다. 하지만 시스템 V 계열에서는 보통 반환하기 전에 타임아웃 변수를 설정하는 반면 BSD 계열에서는 그러지 않는다.

`pselect()`는 POSIX.1g에, 그리고 POSIX.1-2001 및 POSIX.1-2008에 규정돼 있다.

## NOTES

`fd_set`은 고정 크기 버퍼이다. 음수이거나 `FD_SETSIZE`와 같거나 더 큰 `fd` 값으로 `FD_CLR()`이나 `FD_SET()`을 실행할 때의 동작 방식은 규정돼 있지 않다. 또한 POSIX에서는 `fd`가 유효한 파일 디스크립터여야 한다고 요구한다.

`select()` 및 `pselect()`의 동작은 `O_NONBLOCK` 플래그에 영향을 받지 않는다.

일부 다른 유닉스 시스템에서는 커널 내부 자원을 할당하지 못했을 때 리눅스의 `ENOMEM`이 아니라 `EAGAIN` 오류로 실패할 수 있다. POSIX에서 <tt>[[poll(2)]]</tt>에 이 오류를 명세하고 있지만 `select()`에 대해선 아니다. 이식 가능한 프로그램에서는 `EAGAIN`을 확인해서 `EINTR` 경우처럼 루프를 계속 도는 게 좋을 수 있다.

### 자가 파이프 기법

`pselect()`가 없는 시스템에서는 자가 파이프 기법을 써서 신뢰성 있게 (그리고 더 이식성 있게) 시그널 잡기를 할 수 있다. 이 기법은 시그널 핸들러에서 파이프로 한 바이트를 써넣고 그 반대쪽을 주 프로그램의 `select()`로 감시하는 것이다. (가득 찬 파이프에 써넣거나 빈 파이프에서 읽을 때 블록 될 가능성을 피하기 위해 파이프에 읽고 쓸 때 논블로킹 I/O를 쓴다.)

### <tt>[[usleep(3)]]</tt> 흉내내기

<tt>[[usleep(3)]]</tt>이 출현하기 전에 어떤 코드에서는 꽤 이식성 있게 초 단위 이하 정밀로도 잠드는 방법으로 세 집합을 모두 비우고 `nfds`를 0으로 하고 NULL 아닌 `timeout`으로 `select()`를 호출하는 방법을 이용했다.

### `select()`와 `poll()` 알림의 대응 관계

리눅스 커널 소스에서 찾을 수 있는 다음 정의들이 `select()`의 읽기 가능, 쓰기 가능, 예외 상황 알림과 <tt>[[poll(2)]]</tt> 및 <tt>[[epoll(7)]]</tt>에서 제공하는 이벤트 알림 사이의 대응 관계를 보여 준다.

```c
#define POLLIN_SET  (POLLRDNORM | POLLRDBAND | POLLIN |
                     POLLHUP | POLLERR)
                   /* 읽기 준비됨 */
#define POLLOUT_SET (POLLWRBAND | POLLWRNORM | POLLOUT |
                     POLLERR)
                   /* 쓰기 준비됨 */
#define POLLEX_SET  (POLLPRI)
                   /* 예외 상황 */
```

### 다중 스레드 응용

`select()`로 감시 중인 파일 디스크립터를 다른 스레드에서 닫는 경우의 결과는 명세돼 있지 않다. 어떤 유닉스 시스템에서는 `select()`가 블록을 멈추고 반환하며 그 파일 디스크립터가 준비 상태라고 표시한다. (그러면 이어지는 I/O 동작이 오류로 실패하게 된다. 단 `select()` 반환 시점과 I/O 동작 수행 시점 사이에 다른 프로세스에서 그 파일 디스크립터를 다시 열지 않아야 한다.) 리눅스에서는 (그리고 어떤 다른 시스템에서는) 다른 스레드에서 파일 디스크립터를 닫는 게 `select()`에 영향을 주지 않는다. 요컨대 이 상황에서 특정 동작 방식에 의존하는 응용은 버그가 있는 것으로 봐야 한다.

### C 라이브러리/커널 차이

리눅스 커널에서는 임의 크기의 파일 디스크립터 집합이 가능하고 `nfds`의 값으로 검사할 집합의 길이를 알아낸다. 하지만 glibc 구현에서는 `fd_set` 타입의 길이가 고정돼 있다. BUGS도 참고.

이 페이지에서 기술하는 `pselect()` 인터페이스는 glibc에서 구현하고 있다. 기반 리눅스 시스템 호출의 이름은 `pselect6()`이다. 이 시스템 호출은 glibc 래퍼 함수와 동작 방식이 좀 다르다.

리눅스의 `pselect6()` 시스템 호출에서는 `timeout` 인자를 변경한다. 하지만 glibc 래퍼 함수에서 타임아웃 인자에 대한 지역 변수를 쓰고 그 변수를 시스템 호출로 전달하여 그 동작 방식을 감춘다. 그리하여 glibc의 `pselect()` 함수는 `timeout` 인자를 변경하지 않는다. 이는 POSIX.1-2001에서 요구하는 동작 방식이다.

`pselect6()` 시스템 호출의 마지막 인자는 `sigset_t *` 포인터가 아니라 다음 형태의 구조체이다.

```c
struct {
    const kernel_sigset_t *ss;   /* 시그널 집합에 대한 포인터 */
    size_t ss_len;               /* 'ss'가 가리키는 객체의 크기
                                    (바이트 단위) */
};
```

그래서 여러 아키텍처에서 시스템 호출에 최대 6개 인자만 지원한다는 점을 감안하면서 시스템 호출에서 시그널 집합 포인터와 그 크기를 모두 받을 수 있다. 커널과 libc에서의 시그널 집합 개념 차이에 대한 설명은 <tt>[[sigprocmask(2)]]</tt>를 보라.

### glibc 세부 이력

glibc 2.0에서는 `sigmask` 인자를 받지 않는 잘못된 `pselect()` 버전을 제공했다.

glibc 버전 2.1에서 2.2.1까지는 `<sys/select.h>`에서 `pselect()` 정의를 얻으려면 `_GNU_SOURCE`를 정의해야 한다.

## BUGS

POSIX에서는 파일 디스크립터 집합에 지정할 수 있는 파일 디스크립터들의 범위에 대해 구현에서 상한을 정해서 상수 `FD_SETSIZE`를 통해 알리는 것을 허용한다. 리눅스 커널에서는 어떤 고정 제한도 두지 않지만 glibc 구현에서는 `FD_SETSIZE`를 1024로 해서 `fd_set`을 고정 크기 타입으로 만들고 `FD_*()` 매크로가 그 제한에 따라 동작하게 한다. 1023보다 큰 파일 디스크립터를 감시하려면 <tt>[[poll(2)]]</tt>이나 <tt>[[epoll(7)]]</tt>을 써야 한다.

POSIX에 따르면 `select()`에서는 세 파일 디스크립터 집합에 지정된 모든 파일 디스크립터들을 상한 `nfds-1`까지 확인해야 한다. 하지만 현재 구현에서는 프로세스가 현재 열어 둔 파일 디스크립터 번호의 최댓값보다 큰 파일 디스크립터를 무시한다. POSIX에 따르면 한 집합에 그런 파일 디스크립터가 지정돼 있으면 `EBADF` 오류가 발생해야 한다.

glibc 버전 2.1부터 <tt>[[sigprocmask(2)]]</tt>와 `select()`를 이용해 구현한 `pselect()` 에뮬레이션을 제공했다. 그 구현은 `pselect()`가 방지해야 하는 바로 그 경쟁 조건에 여전히 취약했다. 최근의 glibc 버전들에서는 커널에서 (경쟁 없는) `pselect()` 시스템 호출을 제공하면 그걸 쓴다.

리눅스에서는 `select()`에서 어떤 소켓 파일 디스크립터를 "읽기 준비됨"으로 보고했는데 이어지는 읽기가 블록 될 수도 있다. 예를 들어 데이터가 도착했지만 조사해 보니 체크섬이 틀려서 폐기할 때 그럴 수 있다. 그 외에도 파일 디스크립터가 준비 상태라고 잘못 보고하는 다른 경우들이 있을 수 있다. 따라서 블록 해서는 안 되는 소켓에서는 `O_NONBLOCK`을 쓰는 게 안전할 수 있다.

리눅스에서는 시그널 핸들러에 의해 호출이 중단된 경우(즉 `EINTR` 오류 반환)에도 `select()`에서 `timeout`을 변경한다. 이는 POSIX.1에서 허용하지 않는 동작이다. 리눅스의 `pselect()` 시스템 호출도 동일하게 동작하지만 glibc 래퍼에서 내부적으로 `timeout`을 지역 변수로 복사하고 그 변수를 시스템 호출에 전달함으로써 그 동작 방식을 감춘다.

## EXAMPLES

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/select.h>

int
main(void)
{
    fd_set rfds;
    struct timeval tv;
    int retval;

    /* stdin(fd 0)에 입력이 있는지 감시. */

    FD_ZERO(&rfds);
    FD_SET(0, &rfds);

    /* 5초까지 기다림. */

    tv.tv_sec = 5;
    tv.tv_usec = 0;

    retval = select(1, &rfds, NULL, NULL, &tv);
    /* tv의 값에 의존하지 말 것! */

    if (retval == -1)
        perror("select()");
    else if (retval)
        printf("Data is available now.\n");
        /* FD_ISSET(0, &rfds)가 참임. */
    else
        printf("No data within five seconds.\n");

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`accept(2)`, `connect(2)`, <tt>[[poll(2)]]</tt>, <tt>[[read(2)]]</tt>, <tt>[[recv(2)]]</tt>, <tt>[[restart_syscall(2)]]</tt>, <tt>[[send(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[write(2)]]</tt>, <tt>[[epoll(7)]]</tt>, <tt>[[time(7)]]</tt>

설명과 예시가 있는 자습서인 <tt>[[select_tut(2)]]</tt> 참고.

----

2021-03-22
