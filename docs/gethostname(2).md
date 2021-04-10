## NAME

gethostname, sethostname - 호스트명 얻기/설정하기

## SYNOPSIS

```c
#include <unistd.h>

int gethostname(char *name, size_t len);
int sethostname(const char *name, size_t len);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`gethostname()`:
:   glibc 2.12부터:
    :   `_BSD_SOURCE || _XOPEN_SOURCE >= 500`<br>
        `|| /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200112L`

`sethostname()`:
:   glibc 2.21부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 2.20:
    :   `_DEFAULT_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

    glibc 2.19까지:
    :   `_BSD_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

## DESCRIPTION

이 시스템 호출들을 이용해 현재 프로세서의 호스트명에 접근하거나 변경한다.

`sethostname()`은 호스트명을 문자 배열 `name`에 준 값으로 설정한다. `len` 인자는 `name`의 바이트 수를 나타낸다. (즉 `name`에 종료용 널 바이트가 필요치 않다.)

`gethostname()`은 널로 끝나는 호스트명을 길이가 `len` 바이트인 문자 배열 `name`으로 반환한다. 그 널 종료 호스트명이 너무 긴 경우에는 이름이 잘리며 어떤 오류도 반환하지 않는다. (단 아래 NOTES 참고.) POSIX.1에서는 그런 절단이 발생하는 경우에 반환 버퍼에 종료용 널 바이트가 포함되는지 여부가 명세되어 있지 않다고 한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`EFAULT`
:   `name`이 유효하지 않은 주소이다.

`EINVAL`
:   `len`이 음수이거나, `sethostname()`에서 `len`이 허용 최대 길이보다 크다.

`ENAMETOOLONG`
:   (glibc `gethostname()`) `len`이 실제 크기보다 작다. (버전 2.1 전에서 glibc는 이 경우에 `EINVAL`을 사용한다.)

`EPERM`
:   `sethostname()`에서 호출자가 자기 UTS 네임스페이스에 연계된 사용자 네임스페이스에서 `CAP_SYS_ADMIN` 역능을 가지고 있지 않다. (<tt>[[namespaces(7)]]</tt> 참고.)

## CONFORMING TO

SVr4, 4.4BSD (4.2BSD에서 이 인터페이스들이 처음 등장). POSIX.1-2001과 POSIX.1-2008에서 `gethostname()`은 명세하지만 `sethostname()`은 명세하지 않는다.

## NOTES

SUSv2에서는 "호스트 이름이 255바이트로 제한된다"고 보장한다. POSIX.1에서는 "호스트 이름이 (종료용 널 바이트 포함하지 않음) `HOST_NAME_MAX` 바이트로 제한된다"고 보장한다. 리눅스에서 `HOST_NAME_MAX`는 64 값으로 정의돼 있으며 리눅스 1.0부터 그랬다. (그 전 커널에서는 제한값이 8바이트였다.)

### C 라이브러리/커널 차이

GNU C 라이브러리에서는 `gethostname()` 시스템 호출을 이용하지 않는다. 대신 <tt>[[uname(2)]]</tt>을 호출해서 반환된 `nodename` 필드를 `name`으로 `len` 바이트까지 복사하는 라이브러리 함수로 `gethostname()`을 구현한다. 복사를 수행한 다음 그 함수에서는 `nodename`의 길이가 `len`과 같거나 그보다 큰지 확인하고, 그런 경우에는 `errno`를 `ENAMETOOLONG`으로 설정하고 -1을 반환한다. 이 경우 반환되는 `name`에 종료용 널 바이트가 포함되어 있지 않다.

glibc 버전 2.2 전에서는 `nodename` 길이가 `len`과 같거나 그보다 큰 경우를 다르게 처리한다. 함수에서 `name`으로 아무것도 복사하지 않고서 `errno`를 `ENAMETOOLONG`으로 설정하고 -1을 반환한다.

## SEE ALSO

`hostname(1)`, <tt>[[getdomainname(2)]]</tt>, <tt>[[uname(2)]]</tt>

----

2017-09-15
