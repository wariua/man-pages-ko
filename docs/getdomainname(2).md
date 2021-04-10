## NAME

getdomainname, setdomainname - NIS 도메인 이름 얻기/설정하기

## SYNOPSIS

```c
#include <unistd.h>

int getdomainname(char *name, size_t len);
int setdomainname(const char *name, size_t len);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getdomainname()`, `setdomainname()`:
:   glibc 2.21부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 2.20:
    :   `_DEFAULT_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

    glibc 2.19까지:
    :   `_BSD_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

## DESCRIPTION

이 함수들을 이용해 호스트 시스템의 NIS 도메인 이름에 접근하거나 변경한다.

`setdomainname()`은 도메인 이름을 문자 배열 `name`에 준 값으로 설정한다. `len` 인자는 `name`의 바이트 수를 나타낸다. (즉 `name`에 종료용 널 바이트가 필요치 않다.)

`getdomainname()`은 널로 끝나는 도메인 이름을 길이가 `len` 바이트인 문자 배열 `name`으로 반환한다. 그 널 종료 도메인 이름에 `len` 바이트 넘게 필요한 경우 `getdomainname()`은 (glibc에서) 처음 `len` 바이트를 반환하거나 (libc에서) 오류를 내놓는다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`setdomainname()`이 다음 오류로 실패할 수 있다.

`EFAULT`
:   `name`이 사용자 주소 공간 밖을 가리킨다.

`EINVAL`
:   `len`이 음수이거나 너무 크다.

`EPERM`
:   호출자가 자기 UTS 네임스페이스에 연계된 사용자 네임스페이스에서 `CAP_SYS_ADMIN` 역능을 가지고 있지 않다. (<tt>[[namespaces(7)]]</tt> 참고.)

`getdomainname()`이 다음 오류로 실패할 수 있다.

`EINVAL`
:   libc 하의 `getdomainname()`: `name`이 NULL이거나 `name`이 `len` 바이트보다 길다.

## CONFORMING TO

POSIX에 이 호출들이 명세되어 있지 않다.

## NOTES

리눅스 1.0부터는 도메인 이름 길이에 대한 제한값이 종료용 널 바이트를 포함해서 64바이트이다. 그 전 커널에서는 8바이트였다.

대다수 리눅스 아키텍처(x86 포함)에는 `getdomainname()` 시스템 호출이 없다. glibc에서는 대신 <tt>[[uname(2)]]</tt> 호출이 반환한 `domainname` 필드의 사본을 반환하는 라이브러리 함수로 `getdomainname()`을 구현한다.

## SEE ALSO

<tt>[[gethostname(2)]]</tt>, <tt>[[sethostname(2)]]</tt>, <tt>[[uname(2)]]</tt>

----

2017-09-15
