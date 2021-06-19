## NAME

sockatmark - 소켓이 대역외 표시에 있는지 알아보기

## SYNOPSIS

```c
#include <sys/socket.h>

int sockatmark(int sockfd);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sockatmark()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`sockatmark()`는 파일 디스크립터 `sockfd`가 가리키는 소켓이 대역외 표시에 위치해 있는지 여부를 나타내는 값을 반환한다. 소켓이 그 표시에 있으면 1을 반환한다. 소켓이 그 표시에 있지 않으면 0을 반환한다. 이 함수가 대역외 표시를 제거하지는 않는다.

## RETURN VALUE

성공적인 `sockatmark()` 호출은 소켓이 대역외 표시에 있으면 1을 반환하고 아니면 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   `sockfd`가 유효한 파일 디스크립터가 아니다.

`EINVAL`
:   `sockfd`가 `sockatmark()`를 적용할 수 있는 파일 디스크립터가 아니다.

## VERSIONS

glibc 버전 2.2.4에서 `sockatmark()`가 추가되었다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `sockatmark()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`sockatmark()`가 1을 반환하는 경우에는 <tt>[[recv(2)]]</tt>의 `MSG_OOB` 플래그를 사용해 대역외 데이터를 읽을 수 있다.

일부 스트림 소켓 프로토콜들에서만 대역외 데이터를 지원한다.

`SIGURG` 시그널 핸들러에서 안전하게 `sockatmark()`를 호출할 수 있다.

`sockatmark()`는 `SIOCATMARK` <tt>[[ioctl(2)]]</tt> 동작을 이용해 구현되어 있다.

## BUGS

glibc 2.4 전에서는 `sockatmark()`가 제대로 동작하지 않았다.

## EXAMPLES

다음 코드를 `SIGURG` 시그널 수신 후에 사용해서 표시까지의 모든 데이터를 읽어들인 (그리고 폐기한) 다음 표시에 있는 데이터 바이트를 읽을 수 있다.

```c
char buf[BUF_LEN];
char oobdata;
int atmark, s;

for (;;) {
    atmark = sockatmark(sockfd);
    if (atmark == -1) {
        perror("sockatmark");
        break;
    }

    if (atmark)
        break;

    s = read(sockfd, buf, BUF_LEN);
    if (s == -1)
        perror("read");
    if (s <= 0)
        break;
}

if (atmark == 1) {
    if (recv(sockfd, &oobdata, 1, MSG_OOB) == -1) {
        perror("recv");
        ...
    }
}
```

## SEE ALSO

<tt>[[fcntl(2)]]</tt>, <tt>[[recv(2)]]</tt>, <tt>[[send(2)]]</tt>, <tt>[[tcp(7)]]</tt>

----

2021-03-22
