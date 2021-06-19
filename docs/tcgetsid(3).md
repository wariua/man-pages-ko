## NAME

tcgetsid - 세션 ID 얻기

## SYNOPSIS

```c
#define _XOPEN_SOURCE 500        /* feature_test_macros(7) */
#include <termios.h>

pid_t tcgetsid(int fd);
```

## DESCRIPTION

`tcgetsid()` 함수는 `fd`에 연계된 터미널이 제어 터미널인 현재 세션의 세션 ID를 반환한다. 그 터미널이 호출 프로세스의 제어 터미널이어야 한다.

## RETURN VALUE

`fd`가 우리 세션의 제어 터미널을 가리키는 경우 `tcgetsid()` 함수는 그 세션의 세션 ID를 반환한다. 아니면 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니다.

`ENOTTY`
:   호출 프로세스에 제어 터미널이 없거나, `fd`가 나타내는 터미널이 아니다.

## VERSIONS

glibc 버전 2.1부터 `tcgetsid()`를 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `tcgetsid()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`TIOCGSID` <tt>[[ioctl(2)]]</tt>을 통해 이 함수가 구현돼 있으며 리눅스 2.1.71부터 존재한다.

## SEE ALSO

<tt>[[getsid(2)]]</tt>

----

2021-03-22
