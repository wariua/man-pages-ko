## NAME

nice - 프로세스 우선순위 바꾸기

## SYNOPSIS

```c
#include <unistd.h>

int nice(int inc);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`nice()`:
:   `_XOPEN_SOURCE`<br>
    `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19: */ _BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

`nice()`는 호출 스레드의 나이스 값에 `inc`를 더한다. (높은 나이스 값이 낮은 우선순위를 뜻한다.)

나이스 값의 범위는 +19(낮은 우선순위)에서 -20(높은 우선순위)까지이다. 그 범위 밖으로 나이스 값을 설정하려고 하면 범위에 맞게 잘린다.

전통적으로는 특권 프로세스만 나이스 값을 낮출 수 (즉 우선순위를 높게 설정할 수) 있었다. 하지만 리눅스 2.6.12부터는 비특권 프로세스가 적절한 `RLIMIT_NICE` 연성 제한을 가진 대상 프로세스의 나이스 값을 낮출 수 있다. 자세한 내용은 <tt>[[getrlimit(2)]]</tt>을 보라.

## RETURN VALUE

성공 시 새 나이스 값을 반환한다. (하지만 아래 NOTES를 보라.) 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

성공한 호출이 적법하게 -1을 반환할 수 있다. 오류를 탐지하려면 호출 전에 `errno`를 0으로 설정했다가 `nice()`가 -1을 반환한 후에 0이 아닌지 확인하면 된다.

## ERRORS

`EPERM`
:   호출 프로세스가 음수 `inc`를 제공하여 자기 우선순위를 높이려고 시도했지만 충분한 특권을 가지고 있지 않다. 리눅스에서는 `CAP_SYS_NICE` 역능이 필요하다. (하지만 <tt>[[setrlimit(2)]]</tt>에 있는 `RLIMIT_NICE` 자원 제한 설명을 보라.)

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD. 하지만 시스템 호출과 (glibc 2.2.4 전의) (g)libc의 반환 값이 비표준이다. 아래 참고.

## NOTES

나이스 값에 대한 더 자세한 내용은 <tt>[[sched(7)]]</tt>를 보라.

*주의*: 리눅스 2.6.38에 "autogroup" 기능이 추가되면서 많은 경우에서 나이스 값이 더이상 전통적 효과를 주지 않게 되었다. 자세한 내용은 <tt>[[sched(7)]]</tt>를 보라.

### C 라이브러리/커널 차이

POSIX.1에서는 `nice()`가 새 나이스 값을 반환해야 한다고 명세한다. 하지만 리눅스 시스템 호출은 성공 시 0을 반환한다. 마찬가지로 glibc 2.2.3 이전에서 제공하는 `nice()` 래퍼 함수는 성공 시 0을 반환한다.

glibc 2.2.4부터는 glibc 제공 래퍼 함수에서 <tt>[[getpriority(2)]]</tt> 호출로 새 나이스 값을 얻어서 호출자에게 반환함으로써 POSIX.1을 준수한다.

## SEE ALSO

`nice(1)`, `renice(1)`, <tt>[[fork(2)]]</tt>, <tt>[[getpriority(2)]]</tt>, <tt>[[getrlimit(2)]]</tt>, <tt>[[setpriority(2)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2021-03-22
