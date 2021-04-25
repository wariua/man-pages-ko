## NAME

epoll_wait, epoll_pwait, epoll_pwait2 - epoll 파일 디스크립터에서 I/O 이벤트 기다리기

## SYNOPSIS

```c
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *events,
               int maxevents, int timeout);
int epoll_pwait(int epfd, struct epoll_event *events,
               int maxevents, int timeout,
               const sigset_t *sigmask);
int epoll_pwait2(int epfd, struct epoll_event *events,
               int maxevents, const struct timespec *timeout,
               const sigset_t *sigmask);
```

## DESCRIPTION

`epoll_wait()` 시스템 호출은 파일 디스크립터 `epfd`가 가리키는 <tt>[[epoll(7)]]</tt> 인스턴스에서 이벤트를 기다린다. `events`가 가리키는 버퍼를 이용해서 뭔가 이벤트가 있는 관심 목록 내 파일 디스크립터들에 대한 준비 목록 정보를 반환한다. 최대 `maxevents` 개 이벤트를 반환한다. `maxevents` 인자는 0보다 커야 한다.

`timeout` 인자는 `epoll_wait()`에서 블록 할 밀리초 수를 나타낸다. `CLOCK_MONOTONIC` 클럭으로 시간을 측정한다.

다음 어느 경우든 해당할 때까지 `epoll_wait()` 호출이 블록 하게 된다.

* 파일 디스크립터가 이벤트를 내놓는다.

* 호출이 시그널 핸들러에 의해 중단된다.

* 타임아웃이 만료된다.

참고로 `timeout` 시간을 시스템 클럭 해상도에 따라 올림 하게 되며 커널 스케줄링 지연도 있기 때문에 그 블록 시간을 약간 넘길 수도 있다. `timeout`을 -1로 지정하면 `epoll_wait()`이 무한정 블록 하게 되며, `timeout`을 0으로 지정하면 가용 이벤트가 없더라도 `epoll_wait()`이 즉시 반환하게 된다.

`struct epoll_event`는 다음으로 정의돼 있다.

```c
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* epoll 이벤트 */
    epoll_data_t data;        /* 사용자 데이터 변수 */
};
```

반환된 `epoll_event` 구조체 각각의 `data` 필드에는 해당 열린 파일 디스크립터에 대해 가장 최근 <tt>[[epoll_ctl(2)]]</tt>(`EPOLL_CTL_ADD`, `EPOLL_CTL_MOD`)에서 지정한 것과 동일한 데이터가 담겨 있다.

`events` 필드는 해당 열린 파일 기술 항목에 발생한 이벤트들을 나타내는 비트 마스크다. 이 마스크에 등장할 수 있는 비트들의 목록은 <tt>[[epoll_ctl(2)]]</tt>을 보라.

### `epoll_pwait()`

`epoll_wait()`과 `epoll_pwait()`의 관계는 <tt>[[select(2)]]</tt>와 <tt>[[pselect(2)]]</tt>의 관계와 비슷하다. <tt>[[pselect(2)]]</tt>처럼 응용에서 `epoll_pwait()`을 사용해 파일 디스크립터가 준비 상태가 되거나 시그널을 잡을 때까지 안전하게 대기할 수 있다.

다음 `epoll_pwait()` 호출은

```c
ready = epoll_pwait(epfd, &events, maxevents, timeout, &sigmask);
```

다음 호출들을 *원자적으로* 실행하는 것과 동등하다.

```c
sigset_t origmask;

pthread_sigmask(SIG_SETMASK, &sigmask, &origmask);
ready = epoll_wait(epfd, &events, maxevents, timeout);
pthread_sigmask(SIG_SETMASK, &origmask, NULL);
```

`sigmask` 인자를 NULL로 지정할 수 있으며, 그 경우 `epoll_pwait()`은 `epoll_wait()`과 동등하다.

### `epoll_pwait2()`

`epoll_pwait2()`는 `timeout` 인자를 빼면 `epoll_pwait()`과 동등하다. `timespect` 타입으로 인자를 받으므로 나노초 해상도로 타임아웃을 지정할 수 있다. 이 인자는 <tt>[[pselect(2)]]</tt> 및 <tt>[[ppoll(2)]]</tt>에서과 똑같이 동작한다. `timeout`이 NULL이면 `epoll_pwait2()`가 무한정 블록 할 수 있다.

## RETURN VALUE

성공 시 `epoll_wait()`은 요청 I/O에 준비된 파일 디스크립터 수를 반환하며, 지정한 `timeout` 밀리초 동안 준비 상태가 된 파일 디스크립터가 없으면 0을 반환한다. 오류 시 `epoll_wait()`은 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   `epfd`가 유요한 파일 디스크립터가 아니다.

`EFAULT`
:   `events`가 가리키는 메모리 영역이 쓰기 권한으로 접근 가능하지 않다.

`EINTR`
:   (1) 요청한 한 이벤트가 발생하거나 (2) `timeout`이 만료하기 전에 호출이 시그널 핸들러에 의해 중단되었다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `epfd`가 **epoll** 파일 디스크립터가 아니거나, `maxevents`가 0보다 작거나 같다.

## VERSIONS

커널 버전 2.6에서 `epoll_wait()`이 추가되었다. glibc 버전 2.3.2부터 라이브러리 지원을 제공한다.

리눅스 커널 2.6.19에서 `epoll_pwait()`이 추가되었다. glibc 버전 2.6부터 라이브러리 지원을 제공한다.

리눅스 커널 5.11에서 `epoll_pwait2()`가 추가되었다.

## CONFORMING TO

`epoll_wait()`, `epoll_pwait()`, `epoll_pwait2()`는 리눅스 전용이다.

## NOTES

한 스레드가 `epoll_wait()` 호출 내에서 블록 돼 있는 동안 다른 스레드에서 그 **epoll** 인스턴스에 파일 디스크립터를 추가하는 게 가능하다. 새 파일 디스크립터가 준비 상태가 되면 `epoll_wait()` 호출의 블록이 풀리게 된다.

`epoll_wait()` 호출 시 `maxevents` 개를 넘는 파일 디스크립터가 준비 상태이면 이어지는 `epoll_wait()` 호출에서 준비 상태 파일 디스크립터들을 라운드 로빈으로 처리하게 된다. 이 동작은 프로세스에서 준비 상태라고 알고 있는 파일 디스크립터들에 집중하느라 준비 상태인 파일 디스크립터가 더 있다는 걸 알아채지 못해서 발생하는 기아 상황을 피하는 데 도움이 된다.

참고로 관심 목록이 현재 비어 있는 (또는 다른 스레드에서 파일 디스크립터를 닫거나 관심 목록에서 제거해서 관심 목록이 비게 되는) **epoll** 인스턴스에도 `epoll_wait()` 호출이 가능하다. 이후 (다른 스레드에서) 관심 목록에 어떤 파일 디스크립터를 추가하고 그 파일 디스크립터가 준비 상태가 될 때까지 호출이 블록 하게 된다.

## BUGS

커널 2.6.37 전에서는 `timeout` 값이 약 `LONG_MAX / HZ` 밀리초보다 크면 -1으로 (즉 무한으로) 처리한다. 그래서 가령 `sizeof(long)`이 4이고 커널 `HZ` 값이 1000인 시스템이라면 35.79 분보다 큰 타임아웃을 무한으로 처리한다.

### C 라이브러리/커널 차이

진짜 `epoll_pwait()` 및 `epoll_pwait2()` 시스템 호출에는 여섯 번째 인자 `size_t sigsetsize`가 있는데, 이는 `sigmask` 인자의 바이트 단위 크기를 나타낸다. glibc의 `epoll_pwait()` 래퍼 함수에서 이 인자를 고정된 값(`sizeof(sigset_t)`)으로 지정한다.

## SEE ALSO

<tt>[[epoll_create(2)]]</tt>, <tt>[[epoll_ctl(2)]]</tt>, <tt>[[epoll(7)]]</tt>

----

2021-03-22
