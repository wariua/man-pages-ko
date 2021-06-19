## NAME

getsid - 세션 ID 얻기

## SYNOPSIS

```c
#include <unistd.h>

pid_t getsid(pid_t pid);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getsid()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200809L`

## DESCRIPTION

`getsid(0)`는 호출 프로세스의 세션 ID를 반환한다. `getsid()`는 프로세스 ID가 `pid`인 프로세스의 세션 ID를 반환한다. `pid`가 0이면 호출 프로세스의 세션 ID를 반환한다.

## RETURN VALUE

성공 시 세션 ID를 반환한다. 오류 시 `(pid_t) -1`을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EPERM`
:   프로세스 ID가 `pid`인 프로세스가 존재하지만 호출 프로세스와 같은 세션에 있지 않으며, 구현에서 이를 오류로 본다.

`ESRCH`
:   프로세스 ID가 `pid`인 프로세스를 찾을 수 없다.

## VERSIONS

리눅스 버전 2.0부터 이 시스템 호출을 이용할 수 있다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4.

## NOTES

리눅스에선 `EPERM`을 반환하지 않는다.

세션과 세션 ID에 대한 설명은 <tt>[[credentials(7)]]</tt>를 보라.

## SEE ALSO

<tt>[[getpgid(2)]]</tt>, <tt>[[setsid(2)]]</tt>, <tt>[[credentials(7)]]</tt>

----

2021-03-22
