## NAME

CMSG_ALIGN, CMSG_SPACE, CMSG_NXTHDR, CMSG_FIRSTHDR - 보조 데이터에 접근하기

## SYNOPSIS

```c
#include <sys/socket.h>

struct cmsghdr *CMSG_FIRSTHDR(struct msghdr *msgh);
struct cmsghdr *CMSG_NXTHDR(struct msghdr *msgh,
                            struct cmsghdr *cmsg);
size_t CMSG_ALIGN(size_t length);
size_t CMSG_SPACE(size_t length);
size_t CMSG_LEN(size_t length);
unsigned char *CMSG_DATA(struct cmsghdr *cmsg);
```

## DESCRIPTION

이 매크로들을 이용해 소켓 페이로드에 포함되지 않는 제어 메시지(보조 데이터라고도 함)를 생성하고 접근한다. 그런 제어 정보로는 패킷을 받은 인터페이스, 드물게 쓰이는 여러 헤더 필드들, 상세한 오류 설명, 파일 디스크립터, 유닉스 크리덴셜 집합 등이 있을 수 있다. 예를 들어 제어 메시지를 사용해 IP 옵션 같은 추가 헤더 필드를 보낼 수 있다. <tt>[[sendmsg(2)]]</tt>를 호출해서 보조 데이터를 보내고 <tt>[[recvmsg(2)]]</tt>를 호출해서 받는다. 자세한 내용은 각 매뉴얼 페이지를 보라.

보조 데이터는 `cmsghdr` 구조체에 데이터를 덧붙인 것의 배열이다. 사용할 수 있는 제어 메시지 종류에 대해선 프로토콜별 맨 페이지를 보라. 소켓별 보조 버퍼 최대 크기를 `/proc/sys/net/core/optmem_max`로 설정할 수 있다. <tt>[[socket(7)]]</tt> 참고.

`cmsghdr` 구조체는 다음처럼 정의돼 있다.

```c
struct cmsghdr {
    size_t cmsg_len;    /* 데이터 바이트 수. 헤더 포함.
                           (POSIX에서는 타입이 socklen_t) */
    int    cmsg_level;  /* 기원 프로토콜 */
    int    cmsg_type;   /* 프로토콜별 종류 */
/* 이어서:
    unsigned char cmsg_data[]; */
};
```

`cmsghdr` 구조체 열에 절대 직접 접근하지 말아야 한다. 대신 다음 매크로들을 쓰면 된다.

* `CMSG_FIRSTHDR()`는 받은 `msghdr`에 연계된 보조 데이터 버퍼의 첫 번째 `cmsghdr`에 대한 포인터를 반환한다. 버퍼에 `cmsghdr`가 들어갈 공간이 없으면 NULL을 반환한다.

* `CMSG_NXTHDR()`는 받은 `cmsghdr` 다음의 유효한 `cmsghdr`를 반환한다. 버퍼에 충분한 공간이 남아 있지 않으면 NULL을 반환한다.

  (가령 <tt>[[sendmsg(2)]]</tt>로 보낼) 일련의 `cmsghdr` 구조체를 담을 버퍼를 초기화할 때는 `CMSG_NXTHDR()`의 올바른 동작을 위해 먼저 버퍼를 0으로 채우는 게 좋다.

* `CMSG_ALIGN()`은 길이를 받아서 필수 정렬을 포함한 길이를 반환한다. 상수 식이다.

* `CMSG_SPACE()`는 받은 데이터 길이의 페이로드를 가진 보조 항목이 차지하는 바이트 수를 반환한다. 상수 식이다.

* `CMSG_DATA()`는 `cmsghdr`의 데이터 부분에 대한 포인터를 반환한다. 반환되는 포인터가 어떤 페이로드 데이터 타입이든 접근하기 적합하도록 정렬돼 있다고 상정할 수 없다. 응용에서 이를 페이로드에 맞는 포인터 타입으로 캐스팅하지 말아야 한다. 그보단 적절한 객체를 선언해서 `memcpy(3)`로 데이터를 복사하는 게 좋다.

* `CMSG_LEN()`은 `cmsghdr` 구조체의 `cmsg_len` 멤버에 저장할 값을 반환하는데, 필요한 정렬을 포함한 값이다. 데이터 길이를 인자로 받는다. 상수 식이다.

보조 데이터를 만들려면 먼저 `msghdr`의 `msg_controllen` 멤버를 제어 메시지 버퍼의 길이로 설정해야 한다. 그리고 그 `msghdr`에 `CMSG_FIRSTHDR()`를 써서 첫 번째 제어 메시지를 얻고 `CMSG_NXTHDR()`로 이후 메시지들을 얻는다. 각 제어 메시지에서 (`CMSG_LEN()`으로) `cmsg_len`을, 다른 `cmsghdr` 헤더 필드들을, 그리고 `CMSG_DATA()`를 써서 데이터 부분을 채운다. 마지막으로 `msghdr`의 `msg_controllen` 필드를 버퍼 내 모든 제어 메시지 길이의 `CMSG_SPACE()`의 합으로 설정해야 한다. `msghdr`에 대한 더 자세한 내용은 <tt>[[recvmsg(2)]]</tt>를 보라.

## CONFORMING TO

이 보조 데이터 모델은 POSIX.1g 초안, 4.4BSD-Lite, RFC 2292에서 기술하는 IPv6 고급 API, SUSv2를 따른다. `CMSG_FIRSTHDR()`, `CMSG_NXTHDR()`, `CMSG_DATA()`는 POSIX-1.2008에 명세돼 있다. `CMSG_SPACE()`와 `CMSG_LEN()`은 다음 POSIX 릴리스(Issue 8)에 포함될 예정이다.

`CMSG_ALIGN()`은 리눅스 확장이다.

## NOTES

이식성을 위해선 여기서 설명하는 매크로들만 이용해 보조 데이터에 접근해야 한다. `CMSG_ALIGN()`은 리눅스 확장이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

리눅스에서 `CMSG_LEN()`, `CMSG_DATA()`, `CMSG_ALIGN()`은 (인자가 상수라고 하면) 상수 식이고, 따라서 전역 변수 크기를 선언하는 데 그 값을 쓸 수 있다. 하지만 이식성이 없을 수 있다.

## EXAMPLES

다음 코드는 받은 보조 버퍼에서 `IP_TTL` 옵션을 찾는다.

```c
struct msghdr msgh;
struct cmsghdr *cmsg;
int received_ttl;

/* msgh의 보조 데이터 받기 */

for (cmsg = CMSG_FIRSTHDR(&msgh); cmsg != NULL;
        cmsg = CMSG_NXTHDR(&msgh, cmsg)) {
    if (cmsg->cmsg_level == IPPROTO_IP
            && cmsg->cmsg_type == IP_TTL) {
        memcpy(&receive_ttl, CMSG_DATA(cmsg), sizeof(receive_ttl));
        break;
    }
}

if (cmsg == NULL) {
    /* 오류: IP_TTL을 켜지 않았거나 버퍼가 작거나 I/O 오류 */
}
```

아래 코드는 `SCM_RIGHTS`를 이용해 유닉스 도메인 소켓 상으로 파일 디스크립터 배열을 전달한다.

```c
struct msghdr msg = { 0 };
struct cmsghdr *cmsg;
int myfds[NUM_FD];  /* 전달할 파일 디스크립터들을 담음 */
char iobuf[1];
struct iovec io = {
    .iov_base = iobuf,
    .iov_len = sizeof(iobuf)
};
union {         /* 보조 데이터 버퍼. 적절히 정렬되게
                   하기 위해 공용체로 감싼다 */
    char buf[CMSG_SPACE(sizeof(myfds))];
    struct cmsghdr align;
} u;

msg.msg_iov = &io;
msg.msg_iovlen = 1;
msg.msg_control = u.buf;
msg.msg_controllen = sizeof(u.buf);
cmsg = CMSG_FIRSTHDR(&msg);
cmsg->cmsg_level = SOL_SOCKET;
cmsg->cmsg_type = SCM_RIGHTS;
cmsg->cmsg_len = CMSG_LEN(sizeof(myfds));
memcpy(CMSG_DATA(cmsg), myfds, sizeof(myfds));
```

## SEE ALSO

<tt>[[recvmsg(2)]]</tt>, <tt>[[sendmsg(2)]]</tt>

RFC 2292

----

2021-03-22
