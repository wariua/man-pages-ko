## NAME

shutdown - 전이중 연결의 일부 정지시키기

## SYNOPSIS

```c
#include <sys/socket.h>

int shutdown(int sockfd, int how);
```

## DESCRIPTION

`shutdown()` 호출은 `sockfd`에 연계된 소켓 상의 전이중 연결 전체 또는 일부를 정지시킨다. `how`가 `SHUT_RD`이면 더는 수신이 허용되지 않게 된다. `how`가 `SHUT_WR`이면 더는 송신이 허용되지 않게 된다. `how`가 `SHUT_RDWR`이면 더는 수신과 송신이 허용되지 않게 된다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EBADF</code></dt>
<dd><code>sockfd</code>가 유효한 파일 디스크립터가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>how</code>로 유효하지 않은 값을 지정하였다. (하지만 BUGS 참고.)</dd>
<dt><code>ENOTCONN</code></dt>
<dd>지정한 소켓이 연결되어 있지 않다.</dd>
<dt><code>ENOTSOCK</code></dt>
<dd>파일 디스크립터 <code>sockfd</code>가 소켓을 가리키고 있지 않다.</dd>
</dl>

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, 4.4BSD (4.2BSD에서 `shutdown()`이 처음 등장).

## NOTES

상수 `SHUT_RD`, `SHUT_WR`, `SHUT_RDWR`은 각각 0, 1, 2 값을 가지며 glibc-2.1.91부터 `<sys/socket.h>`에 정의되어 있다.

## BUGS

`how`의 유효성 검사는 도메인별 코드에서 이뤄지는데 리눅스 3.7 전에는 모든 도메인에서 이 검사를 수행하지는 않았다. 특히 유닉스 도메인 소켓에서 유효하지 않은 값을 그냥 무시했다. 유닉스 도메인 소켓에 대해선 리눅스 3.7에서 이 문제가 수정되었다.

## SEE ALSO

<tt>[[close(2)]]</tt>, `connect(2)`, <tt>[[socket(2)]]</tt>, <tt>[[socket(7)]]</tt>

----

2018-04-30
