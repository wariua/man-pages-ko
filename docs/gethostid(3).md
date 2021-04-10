## NAME

gethostid, sethostid - 현재 호스트의 고유 식별자를 얻거나 설정하기

## SYNOPSIS

```c
#include <unistd.h>

long gethostid(void);
int sethostid(long hostid);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`gethostid()`:
:   `_BSD_SOURCE || _XOPEN_SOURCE >= 500`

`sethostid()`:
:   glibc 2.21부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 2.20:
    :   `_DEFAULT_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

    glibc 2.19까지:
    :   `_BSD_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

## DESCRIPTION

`gethostid()`와 `sethostid()`는 현재 머신의 고유한 32비트 식별자를 얻거나 설정한다. 그 32비트 식별자는 존재하는 모든 유닉스 시스템 중에서 유일하게 되어 있다. 보통 로컬 머신의 인터넷 주소를 따라가며, 그래서 대개는 설정해 줄 필요가 전혀 없다.

`sethostid()` 호출은 수퍼유저에게로 제한돼 있다.

## RETURN VALUE

`gethostid()`는 `sethostid()`로 설정된 현재 호스트의 32비트 식별자를 반환한다.

`sethostid()`는 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`sethostid()`가 다음 오류로 실패할 수 있다.

`EACCES`
:   호출자가 호스트 ID를 저장하는 데 사용하는 파일에 쓰기 권한을 가지고 있지 않다.

`EPERM`
:   호출 프로세스의 실효 사용자 ID나 그룹 ID가 대응하는 실제 ID와 같지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `gethostid()` | 스레드 안전성 | MT-Safe hostid env locale |
| `sethostid()` | 스레드 안전성 | MT-Unsafe const:hostid |

## CONFORMING TO

4.2BSD. 4.4BSD에서 이 함수들이 빠졌다. SVr4에 `gethostid()`는 포함되지만 `sethostid()`는 아니다.

POSIX.1-2001과 POSIX.1-2008에서 `gethostid()`는 명세하지만 `sethostid()`는 명세하지 않는다.

## NOTES

glibc 구현에서는 파일 `/etc/hostid`에 `hostid`를 저장한다. (glibc 버전 2.2 전에서는 파일 `/var/adm/hostid`를 사용했다.)

glibc 구현에서 `gethostid()`가 호스트 ID를 담은 파일을 열 수 없는 경우에는 <tt>[[gethostname(2)]]</tt>으로 호스트명을 얻고 그 호스트명을 <tt>[[gethostbyname_r(3)]]</tt>에 줘서 호스트의 IPv4 주소를 얻고 그 IPv4 주소의 비트들을 좀 꼬아서 얻은 값을 반환한다. (이 값은 유일하지 않을 수도 있다.)

## BUGS

식별자가 전세계적으로 유일하도록 하는 게 불가능하다.

## SEE ALSO

`hostid(1)`, <tt>[[gethostbyname(3)]]</tt>

----

2017-09-15
