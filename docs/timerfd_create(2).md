## NAME

timerfd_create, timerfd_settime, timerfd_gettime - 파일 디스크립터를 통해 알려 주는 타이머

## SYNOPSIS

```c
#include <sys/timerfd.h>

int timerfd_create(int clockid, int flags);

int timerfd_settime(int fd, int flags,
                    const struct itimerspec *new_value,
                    struct itimerspec *old_value);
int timerfd_gettime(int fd, struct itimerspec *curr_value);
```

## DESCRIPTION

이 시스템 호출들은 파일 디스크립터를 통해 타이머 만료 알림을 전달하는 타이머를 생성하고 조작한다. <tt>[[setitimer(2)]]</tt>나 <tt>[[timer_create(2)]]</tt>의 대안이 되어 주는데 <tt>[[select(2)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>로 그 파일 디스크립터를 감시할 수 있다는 장점이 있다.

이 세 가지 시스템 호출 사용법은 <tt>[[timer_create(2)]]</tt>, <tt>[[timer_settime(2)]]</tt>, <tt>[[timer_gettime(2)]]</tt>과 비슷하다. (<tt>[[timer_getoverrun(2)]]</tt>에 대응하는 것은 없다. 아래에서 설명하듯 `read(2)`로 그 기능성을 제공하기 때문이다.)

### `timerfd_create()`

`timerfd_create()`는 새 타이머 객체를 생성하고 그 타이머를 가리키는 파일 디스크립터를 반환한다. `clockid` 인자는 타이머 진행을 특징 짓는 클럭을 나타내며 다음 중 한 가지여야 한다.

`CLOCK_REALTIME`
:   설정 가능하며 시스템 전역인 실제 시간 클럭.

`CLOCK_MONOTONIC`
:   설정 불가능하며 시스템 구동 후 바뀌지 않는 과거 불특정 시점으로부터의 시간을 측정하는 단조 증가 클럭.

`CLOCK_BOOTTIME` (리눅스 3.15부터)
:   `CLOCK_MONOTONIC`처럼 단조 증가하는 클럭이다. 하지만 `CLOCK_MONOTONIC` 클럭에서 시스템이 절전 대기 상태인 시간을 측정하지 않는 반면 `CLOCK_BOOTTIME` 클럭에서는 시스템이 절전 대기 상태인 시간을 포함한다. 절전 대기를 인식할 필요가 있는 응용들에 유용하다. 그런 응용들에 `CLOCK_REALTIME`은 적합하지 않은데, 그 클럭은 시스템 클럭의 불연속적 변화에 영향을 받기 때문이다.

`CLOCK_REALTIME_ALARM` (리눅스 3.11부터)
:   이 클럭은 `CLOCK_REALTIME`과 비슷하되 시스템이 절전 대기 상태이면 깨우게 된다. 이 클럭에 대해 타이머를 설정하기 위해선 호출자가 `CAP_WAKE_ALARM` 역능을 가지고 있어야 한다.

`CLOCK_BOOTTIME_ALARM` (리눅스 3.11부터)
:   이 클럭은 `CLOCK_BOOTTIME`과 비슷하되 시스템이 절전 대기 상태이면 깨우게 된다. 이 클럭에 대해 타이머를 설정하기 위해선 호출자가 `CAP_WAKE_ALARM` 역능을 가지고 있어야 한다.

위 클럭들에 대한 좀 더 자세한 내용은 <tt>[[clock_getres(2)]]</tt>를 보라.

이 클럭들 각각의 현재 값은 <tt>[[clock_gettime(2)]]</tt>을 이용해 얻어 올 수 있다.

리눅스 2.6.27부터 `flags`에 다음 값들을 비트 OR 해서 `timerfd_create()`의 동작 방식을 바꿀 수 있다.

`TFD_NONBLOCK`
:   새 파일 디스크립터가 가리키는 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)에 `O_NONBLOCK` 파일 상태 플래그를 설정한다. 이 플래그를 사용하면 같은 결과를 얻기 위해 <tt>[[fcntl(2)]]</tt>을 추가로 호출하지 않아도 된다.

`TFD_CLOEXEC`
:   새 파일 디스크립터에 'exec에서 닫기'(`FD_CLOEXEC`) 플래그를 설정한다. 이게 유용할 수 있는 이유에 대해선 <tt>[[open(2)]]</tt>의 `O_CLOEXEC` 플래그 설명을 보라.

리눅스 버전 2.6.26까지에서는 `flags`를 0으로 지정해야 한다.

### `timerfd_settime()`

`timerfd_settime()`은 파일 디스크립터 `fd`가 가리키는 타이머를 장전(시작)하거나 해제(정지)한다.

`new_value` 인자는 타이머의 최초 만료 시간과 간격을 지정한다. 이 인자에 쓰는 `itimerspec` 구조체에는 두 필드가 있고 각각이 다시 `timespec` 타입 구조체이다.

```c
struct timespec {
    time_t tv_sec;                /* 초 */
    long   tv_nsec;               /* 나노초 */
};

struct itimerspec {
    struct timespec it_interval;  /* 주기 타이머의 간격 */
    struct timespec it_value;     /* 최초 만료 */
};
```

`new_value.it_value`는 타이머의 최초 만료 시간을 초와 나노초로 지정한다. `new_value.it_value`의 한 필드라도 0 아닌 값으로 설정하면 타이머가 장전된다. `new_value.it_value`의 두 필드를 모두 0으로 설정하면 타이머를 해제한다.

`new_value.it_interval`의 필드 한 개나 두 개를 0 아닌 값으로 설정하면 최초 만료 후 타이머의 반복 만료 주기를 초와 나노초로 지정한다. `new_value.it_interval`의 두 필드가 모두 0이면 `new_value.it_value`로 지정한 시간에 한 번만 타이머가 만료된다.

기본적으로 `new_value`로 지정하는 최초 만료 시간은 호출 시점에 타이머 클럭의 현재 시간을 기준으로 상대적으로 해석한다. (즉, `new_value.it_value`가 지정하는 것이 `clockid`로 지정한 클럭의 현재 값에 대한 상대적 시간이다.) `flags` 인자를 통해 절대 시간 타임아웃을 선택할 수 있다.

`flags` 인자는 비트 마스크이며 다음 같들을 포함할 수 있다.

`TFD_TIMER_ABSTIME`
:   `new_value.it_value`를 타이머 클럭에서의 절댓값으로 해석한다. 타이머의 클럭이 `new_value.it_value`로 지정한 값에 도달할 때 타이머가 만료된다.

`TFD_TIMER_CANCEL_ON_SET`
:   `TFD_TIMER_ABSTIME`과 함께 이 플래그를 지정하고 이 타이머의 클럭이 `CLOCK_REALTIME`이나 `CLOCK_REALTIME_ALARM`이면 실제 시간 클럭이 불연속적 변경(<tt>[[settimeofday(2)]]</tt>나 <tt>[[clock_settime(2)]]</tt> 같은 것)을 겪을 때 이 타이머를 취소할 수 있다고 표시한다. 그런 변경이 일어날 때 파일 디스크립터에 대한 현재나 미래의 `read(2)`가 `ECANCELED` 오류로 실패하게 된다.

`old_value` 인자가 NULL이 아니면 가리키는 `itimerspec` 구조체를 이용해 호출 시점에 적용 중이던 타이머 설정을 반환한다. 이어지는 `timerfd_gettime()` 설명을 참고하라.

### `timerfd_gettime()`

`timerfd_gettime()`은 파일 디스크립터 `fd`가 가리키는 타이머의 현재 설정을 담은 `itimerspec` 구조체를 `curr_value`로 반환한다.

`it_value` 필드는 타이머가 다음 만료될 때까지 남은 시간을 반환한다. 이 구조체의 두 필드가 모두 0이라면 타이머가 현재 해제되어 있는 것이다. 타이머를 설정할 때 `TFD_TIMER_ABSTIME` 플래그를 지정했는지와 무관하게 이 필드는 항상 상댓값을 담고 있다.

`it_interval` 필드는 타이머의 간격을 반환한다. 이 구조체의 두 필드가 모두 0이라면 `curr_value.it_value`로 지정한 시간에 한 번만 만료되도록 타이머가 설정되어 있는 것이다.

### 타이머 파일 디스크립터 조작

`timerfd_create()`가 반환하는 파일 디스크립터는 다음 추가 작업을 지원한다.

`read(2)`
:   `timerfd_settime()`을 이용해 마지막으로 설정을 변경한 이후로, 또는 마지막 `read(2)` 성공 이후로 타이머가 한 번 이상 만료되었으면 발생한 만료 횟수를 담은 부호 없는 8바이트 정수(`uint64_t`)를 `read(2)`에 준 버퍼로 반환한다. (반환되는 값은 호스트 바이트 순서, 즉 호스트 머신 자체의 정수 바이트 순서로 되어 있다.)

    `read(2)` 시점까지 타이머 만료가 발생하지 않았으면 다음 타이머 만료까지 호출이 블록 한다. 또는 (<tt>[[fcntl(2)]]</tt> `F_SETFL` 동작으로 `O_NONBLOCK` 플래그를 설정해서) 파일 디스크립터를 비블로킹으로 만들었다면 `EAGAIN` 오류로 실패한다.

    제공한 버퍼의 크기가 8바이트보다 작으면 `read(2)`가 `EINVAL` 오류로 실패한다.

    연계된 클럭이 `CLOCK_REALTIME`이나 `CLOCK_REALTIME_ALARM`이고, 타이머가 절대이고 (`TFD_TIMER_ABSTIME`), `timerfd_settime()` 호출 때 `TFD_TIMER_CANCEL_ON_SET` 플래그를 지정했다면 실제 시간 클럭이 불연속적 변경을 거치는 경우 `read(2)`가 `ECANCELED` 오류로 실패한다. (이를 통해 읽기를 하는 응용에서 클럭에 불연속적 변경이 일어났음을 알아챌 수 있다.)

    연계된 클럭이 `CLOCK_REALTIME`이나 `CLOCK_REALTIME_ALARM`이고, 타이머가 절대이고 (`TFD_TIMER_ABSTIME`), `timerfd_settime()` 호출 때 `TFD_TIMER_CANCEL_ON_SET` 플래그를 지정하지 *않았다면* 시간 만료 후 파일 디스크립터에 대한 `read(2)` 전에 클럭에 (가령 <tt>[[clock_settime(2)]]</tt>으로) 불연속적 역방향 변경이 일어나면 `read(2)`가 블록 하지 않고 0값을 반환할 (즉 읽은 바이트가 없을) 수도 있다.

<tt>[[poll(2)]]</tt>, <tt>[[select(2)]]</tt> (기타 유사 함수)
:   타이머 만료가 한 번 이상 일어났을 때 파일 디스크립터가 읽기 가능하다. (<tt>[[select(2)]]</tt> `readfds` 인자, <tt>[[poll(2)]]</tt> `POLLIN` 플래그.)

    파일 디스크립터가 <tt>[[pselect(2)]]</tt>, <tt>[[ppoll(2)]]</tt>, <tt>[[epoll(7)]]</tt> 같은 다른 파일 디스크립터 다중화 API도 지원한다.

`ioctl(2)`
:   다음의 timerfd 한정 명령을 지원한다.

    `TFD_IOC_SET_TICKS` (리눅스 3.17부터)
    :   발생한 타이머 만료 횟수를 조정한다. 인자는 새 만료 횟수를 담은 0 아닌 8바이트 정수에 대한 포인터(`uint64_t*`)이다. 횟수를 설정하고 나서 그 타이머에 대기 중인 프로세스가 있으면 모두 깨운다. 이 명령의 유일한 용도는 체크포인트/복원 목적으로 만료 횟수를 복원하는 것이다. 커널을 `CONFIG_CHECKPOINT_RESTORE` 옵션으로 구성한 경우에만 이 동작이 사용 가능하다.

<tt>[[close(2)]]</tt>
:   파일 디스크립터가 더이상 필요하지 않으면 닫아야 한다. 동일 타이머 객체에 연계된 모든 파일 디스크립터가 닫혔을 때 커널이 타이머를 해제하고 그 자원을 해제한다.

### <tt>[[fork(2)]]</tt> 동작 방식

`timerfd_create()`으로 생성한 파일 디스크립터의 사본을 <tt>[[fork(2)]]</tt> 후에 자식이 물려받는다. 그 파일 디스크립터는 부모에서의 대응하는 파일 디스크립터와 같은 기반 타이머 객체를 가리키며, 자식에서 `read(2)` 하면 그 타이머의 만료에 대한 정보를 반환하게 된다.

### <tt>[[execve(2)]]</tt> 동작 방식

`timerfd_create()`으로 생성한 파일 디스크립터가 <tt>[[execve(2)]]</tt>를 거치면서 보존되며, 타이머가 장전되어 있었다면 계속 타이머 만료가 발생한다.

## RETURN VALUE

성공 시 `timerfd_create()`은 새 파일 디스크립터를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

`timerfd_settime()`과 `timerfd_gettime()`은 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`timerfd_create()`이 다음 오류로 실패할 수 있다.

`EINVAL`
:   `clockid`가 유효하지 않다.

`EINVAL`
:   `flags`가 유효하지 않다. 또는 리눅스 2.6.26 또는 이전에서 `flags`가 0이 아니다.

`EMFILE`
:   열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다.

`ENFILE`
:   열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.

`ENODEV`
:   (내부적으로 쓰는) 익명 아이노드 장치를 마운트 할 수 없었다.

`ENOMEM`
:   타이머를 생성하기에 커널 메모리가 충분하지 않았다.

`EPERM`
:   `clockid`가 `CLOCK_REALTIME_ALARM` 또는 `CLOCK_BOOTTIME_ALARM`이었지만 호출자가 `CAP_WAKE_ALARM` 역능을 가지고 있지 않다.

`timerfd_settime()`과 `timerfd_gettime()`이 다음 오류로 실패할 수 있다.

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니다.

`EFAULT`
:   `new_value`나 `old_value`, `curr_value`가 유효한 포인터가 아니다.

`EINVAL`
:   `fd`가 유효한 timerfd 파일 디스크립터가 아니다.

`timerfd_settime()`이 다음 오류로 실패할 수도 있다.

`ECANCELED`
:   NOTES 절을 보라.

`EINVAL`
:   `new_value`가 올바로 초기화 되어 있지 않다. (한 `tv_nsec` 필드가 0에서 999,999,999까지 범위 밖에 있다.)

`EINVAL`
:   `flags`가 유효하지 않다.

## VERSIONS

리눅스 커널 2.6.25부터 이 시스템 호출들이 사용 가능하다. glibc 버전 2.8부터 라이브러리 지원을 제공한다.

## CONFORMING TO

이 시스템 호출들은 리눅스 전용이다.

## NOTES

`timerfd_create()`로 생성한 `CLOCK_REALTIME` 내지 `CLOCK_REALTIME_ALARM` 타이머에 대한 다음 시나리오를 생각해 보자.

(a) `TFD_TIMER_ABSTIME` 및 `TFD_TIMER_CANCEL_ON_SET` 플래그를 써서 타이머가 시작(`timerfd_settime()`)되었다.

(b) 그 후에 `CLOCK_REALTIME` 클럭에 불연속적 변경(가령 <tt>[[settimeofday(2)]]</tt>)이 일어난다.

(c) 호출자가 (파일 디스크립터를 먼저 `read(2)` 하지 않고) 한 번 더 `timerfd_settime()`을 호출해서 타이머를 재장전한다.

이 경우에 다음처럼 된다.

* `timerfd_settime()`이 `errno`를 `ECANCELED`로 해서 -1을 반환한다. (이를 통해 호출자는 앞선 타이머가 클럭의 불연속적 변경에 영향을 받았음을 알게 된다.)

* 두 번째 `timerfd_settime()` 호출에서 준 설정대로 타이머가 *성공적으로 재장전된다*. (이는 아마 우발적인 구현이었을 것이다. 하지만 이 동작 방식에 의존하는 응용들이 있을 수 있으므로 현재 수정 계획이 없다.)

## BUGS

현재 `timerfd_create()`가 지원하는 클럭 ID 종류가 <tt>[[timer_create(2)]]</tt>보다 적다.

## EXAMPLES

다음 프로그램은 타이머를 생성하고서 그 진행을 지켜본다. 프로그램은 세 개까지의 명령행 인자를 받는다. 첫 번째 인자는 타이머의 최초 만료까지의 초 수를 나타낸다. 두 번째 인자는 타이머의 초 단위 간격을 나타낸다. 세 번째 인자는 종료 전까지 프로그램에서 허용해야 하는 타이머 만료 횟수를 나타낸다. 두 번째와 세 번째 명령행 인자는 선택적이다.

```text
$ a.out 3 1 100
0.000: timer started
3.000: read: 1; total=1
4.000: read: 1; total=2
^Z                  # control+Z 입력해서 프로그램 정지
[1]+  Stopped                 ./timerfd3_demo 3 1 100
$ fg                # 몇 초 후에 실행 재개
a.out 3 1 100
9.660: read: 5; total=7
10.000: read: 1; total=8
11.000: read: 1; total=9
^C                  # control+C 입력해서 프로그램 중단
```

### 프로그램 소스

```c
#include <sys/timerfd.h>
#include <time.h>
#include <unistd.h>
#include <inttypes.h>      /* PRIu64 정의 */
#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>        /* uint64_t 정의 */

#define handle_error(msg) \
        do { perror(msg); exit(EXIT_FAILURE); } while (0)

static void
print_elapsed_time(void)
{
    static struct timespec start;
    struct timespec curr;
    static int first_call = 1;
    int secs, nsecs;

    if (first_call) {
        first_call = 0;
        if (clock_gettime(CLOCK_MONOTONIC, &start) == -1)
            handle_error("clock_gettime");
    }

    if (clock_gettime(CLOCK_MONOTONIC, &curr) == -1)
        handle_error("clock_gettime");

    secs = curr.tv_sec - start.tv_sec;
    nsecs = curr.tv_nsec - start.tv_nsec;
    if (nsecs < 0) {
        secs--;
        nsecs += 1000000000;
    }
    printf("%d.%03d: ", secs, (nsecs + 500000) / 1000000);
}

int
main(int argc, char *argv[])
{
    struct itimerspec new_value;
    int max_exp, fd;
    struct timespec now;
    uint64_t exp, tot_exp;
    ssize_t s;

    if ((argc != 2) && (argc != 4)) {
        fprintf(stderr, "%s init-secs [interval-secs max-exp]\n",
                argv[0]);
        exit(EXIT_FAILURE);
    }

    if (clock_gettime(CLOCK_REALTIME, &now) == -1)
        handle_error("clock_gettime");

    /* 명령행에서 지정한 최초 만료 시간와 간격으로
       CLOCK_REALTIME 절대 시간 타이머 생성하기. */

    new_value.it_value.tv_sec = now.tv_sec + atoi(argv[1]);
    new_value.it_value.tv_nsec = now.tv_nsec;
    if (argc == 2) {
        new_value.it_interval.tv_sec = 0;
        max_exp = 1;
    } else {
        new_value.it_interval.tv_sec = atoi(argv[2]);
        max_exp = atoi(argv[3]);
    }
    new_value.it_interval.tv_nsec = 0;

    fd = timerfd_create(CLOCK_REALTIME, 0);
    if (fd == -1)
        handle_error("timerfd_create");

    if (timerfd_settime(fd, TFD_TIMER_ABSTIME, &new_value, NULL) == -1)
        handle_error("timerfd_settime");

    print_elapsed_time();
    printf("timer started\n");

    for (tot_exp = 0; tot_exp < max_exp;) {
        s = read(fd, &exp, sizeof(uint64_t));
        if (s != sizeof(uint64_t))
            handle_error("read");

        tot_exp += exp;
        print_elapsed_time();
        printf("read: %" PRIu64 "; total=%" PRIu64 "\n", exp, tot_exp);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[eventfd(2)]]</tt>, <tt>[[poll(2)]]</tt>, `read(2)`, <tt>[[select(2)]]</tt>, <tt>[[setitimer(2)]]</tt>, <tt>[[signalfd(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[timer_gettime(2)]]</tt>, <tt>[[timer_settime(2)]]</tt>, <tt>[[epoll(7)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
