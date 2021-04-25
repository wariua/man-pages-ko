## NAME

killpg - 프로세스 그룹에게 시그널 보내기

## SYNOPSIS

```c
#include <signal.h>

int killpg(int pgrp, int sig);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`killpg()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19: */ _BSD_SOURCE`

## DESCRIPTION

`killpg()`는 프로세스 그룹 `pgrp`에게 시그널 `sig`를 보낸다. 시그널 목록은 <tt>[[signal(7)]]</tt>을 보라.

`pgrp`이 0인 경우 `killpg()`는 호출 프로세스의 프로세스 그룹에게 시그널을 보낸다. (POSIX: `pgrp`이 1 이하인 경우 동작 방식은 규정되어 있지 않다.)

다른 프로세스에게 시그널을 보내는 데 필요한 권한에 대해선 <tt>[[kill(2)]]</tt>을 보라.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `sig`가 유효한 시그널 번호가 아니다.

`EPERM`
:   프로세스에게 대상 프로세스 어느 것에도 시그널을 보낼 권한이 없다. 필요한 권한에 대해선 <tt>[[kill(2)]]</tt>을 보라.

`ESRCH`
:   `pgrp`으로 지정한 프로세스 그룹에서 프로세스를 찾을 수 없다.

`ESRCH`
:   프로세스 그룹을 0으로 주었는데 보내는 프로세스에게 프로세스 그룹이 없다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.4BSD (4BSD에서 `killpg()`가 처음 등장).

## NOTES

BSD 계열 시스템과 시스템 V 계열 시스템 간에는 권한 검사에 다양한 차이점이 있다. POSIX의 `kill(3p)`에 대한 rationale를 보라. POSIX에서 언급하지 않은 차이점은 반환 값 `EPERM`에 대한 것이다. BSD에서는 최소 한 개의 대상 프로세스에 대해 권한 검사가 실패했을 때 시그널을 보내지 않고 `EPERM`을 반환한다고 적고 있다. 반면 POSIX에서는 모든 대상 프로세스들에 대해 권한 검사가 실패했을 때만 `EPERM`이라고 적고 있다.

### C 라이브러리/커널 차이

리눅스에서 `killpg()`는 `kill(-pgrp, sig)` 호출을 하는 라이브러리 함수로 구현되어 있다.

## SEE ALSO

`getpgrp(2)`, <tt>[[kill(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[credentials(7)]]</tt>

----

2021-03-22
