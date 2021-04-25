## NAME

epoll_ctl - epoll 파일 디스크립터 제어 인터페이스

## SYNOPSIS

```c
#include <sys/epoll.h>

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

## DESCRIPTION

이 시스템 호출을 사용해 파일 디스크립터 `epfd`가 가리키는 <tt>[[epoll(7)]]</tt> 인스턴스의 관심 목록에서 항목을 추가, 변경, 제거한다. 대상 파일 디스크립터 `fd`에 대해 동작 `op`를 수행하기를 요청한다.

`op` 인자에 유효한 값은 다음과 같다.

`EPOLL_CTL_ADD`
:   epoll 파일 디스크립터 `epfd`의 관심 목록에 항목을 추가한다. 항목은 열린 파일 기술 항목(<tt>[[epoll(7)]]</tt> 및 <tt>[[open(2)]]</tt> 참고)에 대응하는 파일 디스크립터 `fd`와 `event`에 지정한 설정으로 이뤄진다.

`EPOLL_CTL_MOD`
:   관심 목록에서 `fd`에 연계된 설정을 `event`의 새 설정으로 바꾼다.

`EPOLL_CTL_DEL`
:   관심 목록에서 대상 파일 디스크립터 `fd`를 제거(등록 해제)한다. `event`는 무시되며 NULL일 수 있다. (하지만 아래 BUGS 참고.)

`event` 인자는 파일 디스크립터 `fd`에 연결할 객체를 기술한다. `struct epoll_event`는 다음처럼 정의돼 있다.

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

`epoll_event` 구조체의 `data` 멤버는 커널에서 저장하고 있다가 이 파일 디스크립터가 준비 상태가 되었을 때 (<tt>[[epoll_wait(2)]]</tt>을 통해) 반환되는 데이터를 나타낸다.

`epoll_event` 구조체의 `events` 멤버는 사용 가능한 다음 이벤트 종류들을 0개 이상 OR 해서 구성한 비트 마스크이다.

`EPOLLIN`
:   연계 파일이 `read(2)`에 사용 가능하다.

`EPOLLOUT`
:   연계 파일이 `write(2)`에 사용 가능하다.

`EPOLLRDHUP` (리눅스 2.6.17부터)
:   스트림 소켓의 상대가 연결을 닫았거나 연결의 쓰기 쪽을 닫았다. (이 플래그는 특히 에지 트리거 사용 시 간단한 코드 작성으로 상대의 shutdown을 탐지하는 데 유용하다.)

`EPOLLPRI`
:   파일 디스크립터에 어떤 예외 상황이 있다. <tt>[[poll(2)]]</tt>의 `POLLPRI` 설명 참고.

`EPOLLERR`
:   연계 파일 디스크립터에서 오류 상황이 발생했다. 또 파이프의 읽기 쪽이 닫혔을 때 쓰기 쪽에서 이 이벤트가 보고된다.

    <tt>[[epoll_wait(2)]]</tt>은 항상 이 이벤트를 보고하므로 `epoll_ctl()` 호출 시 `events`에 설정하지 않아도 된다.

`EPOLLHUP`
:   연계 파일 디스크립터에서 연결이 끊겼다.

    <tt>[[epoll_wait(2)]]</tt>은 항상 이 이벤트를 보고하므로 `epoll_ctl()` 호출 시 `events`에 설정하지 않아도 된다.

    참고로 파이프나 스트림 소켓 같은 채널에서 읽기를 할 때 이 이벤트는 상대가 채널의 그쪽 끝을 닫았다는 표시일 뿐이다. 그 채널의 미처리 데이터를 모두 소비한 후에야 채널 읽기가 0(파일 끝)을 반환하게 된다.

`EPOLLET`
:   연계 파일 디스크립터에 에지 트리거 알림을 요청한다. **epoll**의 기본 동작 방식은 레벨 트리거다. 에지 트리거 및 레벨 트리거 알림에 대한 자세한 정보는 <tt>[[epoll(7)]]</tt>을 보라.

    이 플래그는 `epoll_ctl()` 호출 시 `event.events` 필드에 입력하는 플래그다. 절대 <tt>[[epoll_wait(2)]]</tt>에서 반환되지 않는다.

`EPOLLONESHOT` (리눅스 2.6.2부터)
:   연계 파일 디스크립터에 단발 알림을 요청한다. 즉 <tt>[[epoll_wait(2)]]</tt>으로 파일 디스크립터에 대한 이벤트 알림을 받고 나면 관심 목록에서 연계 파일 디스크립터가 비활성화되어 **epoll** 인터페이스가 다른 이벤트를 보고하지 않게 된다. 그 파일 디스크립터를 재활성화하려면 사용자가 새 이벤트 마스크로 `epoll_ctl()` `EPOLL_CTL_MOD`를 호출해야 한다.

    이 플래그는 `epoll_ctl()` 호출 시 `event.events` 필드에 입력하는 플래그다. 절대 <tt>[[epoll_wait(2)]]</tt>에서 반환되지 않는다.

`EPOLLWAKEUP` (리눅스 3.5부터)
:   `EPOLLONESHOT`과 `EPOLLET`가 설정돼 있지 않고 프로세스에게 `CAP_BLOCK_SUSPEND` 역능이 있으면 이 이벤트가 대기 중이거나 처리 중인 동안 시스템이 "대기"나 "하이버네이션"으로 들어가지 않게 한다. <tt>[[epoll_wait(2)]]</tt> 호출이 이벤트를 반환한 시점부터 시작해서 같은 <tt>[[epoll(7)]]</tt> 파일 디스크립터에 다시 <tt>[[epoll_wait(2)]]</tt> 호출하기, 그 파일 디스크립터 닫기, `EPOLL_CTL_DEL`로 이벤트 파일 디스크립터 제거하기, `EPOLL_CTL_MOD`로 이벤트 파일 디스크립터에서 `EPOLLWAKEUP` 없애기까지를 "처리 중"이라고 본다. BUGS도 참고.

    이 플래그는 `epoll_ctl()` 호출 시 `event.events` 필드에 입력하는 플래그다. 절대 <tt>[[epoll_wait(2)]]</tt>에서 반환되지 않는다.

`EPOLLEXCLUSIVE` (리눅스 4.5부터)
:   대상 파일 디스크립터 `fd`에 붙는 epoll 파일 디스크립터에 배타적 깨우기 모드를 설정한다. 깨우는 이벤트가 발생했는데 같은 대상 파일에 여러 개의 epoll 파일 디스크립터가 `EPOLLEXCLUSIVE`로 붙어 있으면 그 epoll 파일 디스크립터들 중 한 개 또는 그 이상이 <tt>[[epoll_wait(2)]]</tt>로 이벤트를 수신하게 된다. 이런 경우에 (`EPOLLEXCLUSIVE`가 설정돼 있지 않을 때의) 기본 동작은 모든 epoll 파일 디스크립터들이 이벤트를 수신하는 것이다. 따라서 특정 상황에서 단체로 깨어나기(thundering herd) 문제를 피하는 데 `EPOLLEXCLUSIVE`가 쓸모가 있다.

    여러 epoll 인스턴스에 같은 파일 디스크립터가 있는데 일부에는 `EPOLLEXCLUSIVE` 플래그를 썼고 나머지에는 그러지 않았다면 `EPOLLEXCLUSIVE`를 지정하지 않은 epoll 인스턴스는 모두에 이벤트가 제공되고 `EPOLLEXCLUSIVE`를 지정한 epoll 인스턴스는 그 중 최소 하나에 이벤트가 제공된다.

    `EPOLLEXCLUSIVE`와 함께 지정할 수 있는 값들은 `EPOLLIN`, `EPOLLOUT`, `EPOLLWAKEUP`, `EPOLLET`이다. `EPOLLHUP`과 `EPOLLERR`도 지정할 수는 있지만 그럴 필요가 없다. 언제나처럼 그 이벤트들은 `events`에 지정했는지 여부와 상관없이 발생하면 항상 보고된다. `events`에 그 외의 값을 지정하려고 하면 `EINVAL` 오류가 난다.

    `EPOLLEXCLUSIVE`는 `EPOLL_CTL_ADD` 동작에만 쓸 수 있다. `EPOLL_CTL_MOD`에 쓰려고 하면 오류가 난다. `epoll_ctl()`로 `EPOLLEXCLUSIVE`를 설정해 뒀으면 이후 같은 `epfd`, `fd` 짝에 `EPOLL_CTL_MOD`를 하면 오류가 난다. `events`에 `EPOLLEXCLUSIVE`를 지정하고 대상 파일 디스크립터 `fd`에 epoll 인스턴스를 지정해서 `epoll_ctl()`을 호출하는 것 역시 실패하게 된다. 이 경우 모두에서의 오류는 `EINVAL`이다.

    `EPOLLEXCLUSIVE` 플래그는 `epoll_ctl()` 호출 시 `event.events` 필드에 입력하는 플래그다. 절대 <tt>[[epoll_wait(2)]]</tt>에서 반환되지 않는다.

## RETURN VALUE

성공 시 `epoll_ctl()`은 0을 반환한다. 오류 발생 시 `epoll_ctl()`은 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   `epfd`나 `fd`가 유효한 파일 디스크립터가 아니다.

`EEXIST`
:   `op`가 `EPOLL_CTL_ADD`인데 제공된 파일 디스크립터 `fd`가 이미 그 epoll 인스턴스에 등록돼 있다.

`EINVAL`
:   `epfd`가 **epoll** 파일 디스크립터가 아니거나, `fd`가 `epfd`와 같거나, 요청한 동작 `op`를 이 인터페이스에서 지원하지 않는다.

`EINVAL`
:   `events`에 `EPOLLEXCLUSIVE`와 더불어 유효하지 않은 이벤트 종류를 지정했다.

`EINVAL`
:   `op`가 `EPOLL_CTL_MOD`인데 `events`에 `EPOLLEXCLUSIVE`가 포함돼 있다.

`EINVAL`
:   `op`가 `EPOLL_CTL_MOD`인데 앞서 그 `epfd`, `fd` 짝에 `EPOLLEXCLUSIVE` 플래그가 적용되었다.

`EINVAL`
:   `event`에 `EPOLLEXCLUSIVE`를 지정했는데 `fd`가 epoll 인스턴스를 가리킨다.

`ELOOP`
:   `fd`가 epoll 인스턴스를 가리키는데 이 `EPOLL_CTL_ADD` 동작으로 인해 epoll 인스턴스들이 서로를 감시하는 루프가 생기게 되거나 epoll 인스턴스 중첩 단계가 5보다 커진다.

`ENOENT`
:   `op`가 `EPOLL_CTL_MOD`나 `EPOLL_CTL_DEL`인데 `fd`가 그 epoll 인스턴스에 등록돼 있지 않다.

`ENOMEM`
:   요청한 `op` 제어 동작을 처리하기에 충분한 메모리가 없다.

`ENOSPC`
:   epoll 인스턴스에 새 파일 디스크립터를 등록(`EPOLL_CTL_ADD`)하려 하는 중에 `/proc/sys/fs/epoll/max_user_watches`에 따른 제한에 걸렸다. 자세한 내용은 <tt>[[epoll(7)]]</tt> 참고.

`EPERM`
:   대상 파일 `fd`가 **epoll**을 지원하지 않는다. 예를 들어 `fd`가 정규 파일이나 디렉터리를 가리키면 이 오류가 발생할 수 있다.

## VERSIONS

커널 버전 2.6에서 `epoll_ctl()`이 추가되었다. glibc 버전 2.3.2부터 라이브러리 지원을 제공한다.

## CONFORMING TO

`epoll_ctl()`은 리눅스 전용이다.

## NOTES

**epoll** 인터페이스는 <tt>[[poll(2)]]</tt>을 지원하는 파일 디스크립터들을 모두 지원한다.

## BUGS

커널 버전 2.6.9 전에서는 `EPOLL_CTL_DEL` 동작에서 `event`를 무시하는데도 그 인자에 널 아닌 포인터가 있어야 했다. 리눅스 2.6.9부터는 `EPOLL_CTL_DEL` 이용 시 `event`에 NULL을 지정할 수 있다. 2.6.9 전 커널로 이식 가능해야 하는 응용에서는 `event`에 널 아닌 포인터를 지정해 주는 게 좋다.

`flags`에 `EPOLLWAKEUP`을 지정했지만 호출자에게 `CAP_BLOCK_SUSPEND` 역능이 없는 경우에는 `EPOLLWAKEUP` 플래그가 *조용히 무시된다*. 이런 유감스런 동작 방식이 필요한 이유는 최초 구현에서 `flags` 인자의 유효성 검사를 수행하지 않았던 데다가, 검사를 해서 호출자가 `CAP_BLOCK_SUSPEND` 역능을 가지고 있지 않으면 호출이 실패하도록 `EPOLLWAKEUP`을 추가하면 하필 임의적으로 (그리고 불필요하게) 그 비트를 설정했기 때문에 동작에 문제가 생길 기존 사용자 공간 응용이 적어도 한 가지 있었기 때문이다. 따라서 견고한 응용이라면 `EPOLLWAKEUP` 플래그를 쓰려 할 때 `CAP_BLOCK_SUSPEND` 역능을 가지고 있는지 확실히 확인하는 게 좋다.

## SEE ALSO

<tt>[[epoll_create(2)]]</tt>, <tt>[[epoll_wait(2)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>

----

2021-03-22
