## NAME

ulimit - 사용자 제한을 얻고 설정하기

## SYNOPSIS

```c
#include <ulimit.h>

long ulimit(int cmd, long newlimit);
```

## DESCRIPTION

경고: 이 루틴은 구식이다. 대신 <tt>[[getrlimit(2)]]</tt>, <tt>[[setrlimit(2)]]</tt>, <tt>[[sysconf(3)]]</tt>를 사용하라. 셸 명령 `ulimit`에 대해선 `bash(1)`를 보라.

`ulimit()` 호출은 호출 프로세스에 대한 어떤 제한값을 얻거나 설정한다. `cmd` 인자는 다음 값들 중 하나일 수 있다.

`UL_GETFSIZE`
:   파일 크기에 대한 제한을 반환한다. 512바이트 단위이다.

`UL_SETFSIZE`
:   파일 크기에 대한 제한을 설정한다.

`3`
:   (리눅스에 구현돼 있지 않음.) 데이터 세그먼트 주소로 가능한 최댓값을 반환한다.

`4`
:   (구현돼 있지만 심볼 상수 제공하지 않음.) 호출 프로세스가 최대로 열 수 있는 파일 개수를 반환한다.

## RETURN VALUE

성공 시 `ulimit()`는 음수 아닌 값을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`EPERM`
:   비특권 프로세스가 제한값을 올리려고 했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `ulimit()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

SVr4, POSIX.1-2001. POSIX.1-2008에서 `ulimit()`를 구식으로 표시하였다.

## SEE ALSO

`bash(1)`, <tt>[[getrlimit(2)]]</tt>, <tt>[[setrlimit(2)]]</tt>, <tt>[[sysconf(3)]]</tt>

----

2017-09-15
