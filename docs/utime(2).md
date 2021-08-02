## NAME

utime, utimes - 파일 최근 접근 시간 및 수정 시간 바꾸기

## SYNOPSIS

```c
#include <sys/types.h>
#include <utime.h>

int utime(const char *filename, const struct utimbuf *times);

#include <sys/time.h>

int utimes(const char *filename, const struct timeval times[2]);
```

## DESCRIPTION

**참고**: 최신 응용이라면 <tt>[[utimensat(2)]]</tt>에서 기술하는 인터페이스 사용을 선호할 것이다.

`utime()` 시스템 호출은 `filename`이 나타내는 아이노드의 접근 시간(access time)과 수정 시간(modification time)을 각각 `times`의 `actime` 필드와 `modtime` 필드로 바꾼다.

`times`가 NULL이면 파일의 접근 시간과 수정 시간을 현재 시간으로 설정한다.

타임스탬프 변경이 허용되는 때는 프로세스에게 적절한 특권이 있거나, 실효 사용자 ID가 파일의 사용자 ID와 같거나, `times`가 NULL이고 프로세스에게 파일에 대한 쓰기 권한이 있을 때이다.

`utimbuf` 구조체는 다음과 같다.

```c
struct utimbuf {
    time_t actime;       /* 접근 시간 */
    time_t modtime;      /* 수정 시간 */
};
```

`utime()` 시스템 호출에서는 1초 해상도로 타임스탬프를 지정할 수 있다.

`utimes()` 시스템 호출도 비숫하지만 `times` 인자가 구조체가 아닌 배열을 가리킨다. 이 배열의 항목들은 `timeval` 구조체이며, 그래서 1마이크로초 정밀도로 타임스탬프를 지정할 수 있다. `timeval` 구조체는 다음과 같다.

```c
struct timeval {
    long tv_sec;        /* 초 */
    long tv_usec;       /* 마이크로초 */
};
```

`times[0]`이 새 접근 시간을 나타내고 `times[1]`이 새 수정 시간을 나타낸다. `times`가 NULL이면 `utime()`에서와 마찬가지로 파일의 접근 시간과 수정 시간을 현재 시간으로 설정한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   `path`의 경로 선두부의 한 디렉터리에 대해 탐색 권한이 거부되었다. (<tt>[[path_resolution(7)]]</tt>도 참고.)

`EACCES`
:   `times`가 NULL이고, 호출자의 실효 사용자 ID가 파일의 소유자와 일치하지 않고, 호출자가 파일에 쓰기 접근권을 가지고 있지 않고, 호출자에게 특권이 없다 (리눅스: `CAP_DAC_OVERRIDE` 역능과 `CAP_FOWNER` 역능 어느 쪽도 없다).

`ENOENT`
:   `filename`이 존재하지 않는다.

`EPERM`
:   `times`가 NULL이 아니고, 호출자의 실효 UID가 파일의 소유자와 일치하지 않으며, 호출자에게 특권이 없다 (리눅스: `CAP_FOWNER` 역능이 없다).

`EROFS`
:   `path`가 읽기 전용 파일 시스템에 위치해 있다.

## CONFORMING TO

`utime()`: SVr4, POSIX.1-2001. POSIX.1-2008에서 `utime()`을 구식으로 표시하였다.

`utimes()`: 4.3BSD, POSIX.1-2001.

## NOTES

리눅스에서는 불변(immutable) 파일의 타임스탬프를 바꾸는 것이나 덧붙임 전용(append-only) 파일의 타임스탬프를 현재 시간 아닌 값으로 설정하는 것을 허용하지 않는다.

## SEE ALSO

<tt>[[chattr(1)]]</tt>, `touch(1)`, <tt>[[futimesat(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[utimensat(2)]]</tt>, <tt>[[futimens(3)]]</tt>, <tt>[[futimes(3)]]</tt>, <tt>[[inode(7)]]</tt>

----

2021-03-22
