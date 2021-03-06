## NAME

futimes, lutimes - 파일 타임스탬프 바꾸기

## SYNOPSIS

```c
#include <sys/time.h>

int futimes(int fd, const struct timeval tv[2]);
int lutimes(const char *filename, const struct timeval tv[2]);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`futimes()`, `lutimes()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE`

## DESCRIPTION

`futimes()`는 <tt>[[utimes(2)]]</tt>와 같은 방식으로 파일의 접근 시간과 수정 시간을 바꾼다. 차이는 경로명이 아니라 파일 디스크립터를 통해 타임스탬프를 바꿀 파일을 지정한다는 점이다.

`lutimes()`는 <tt>[[utimes(2)]]</tt>와 같은 방식으로 파일의 접근 시간과 수정 시간을 바꾼다. 차이는 `filename`이 심볼릭 링크를 가리키는 경우 그 링크를 따라가지 않는다는 점이다. 대신 그 심볼릭 링크의 타임스탬프를 바꾼다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

오류는 <tt>[[utimes(2)]]</tt>와 마찬가지이되, `futimes()`에 다음이 추가된다.

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니다.

`ENOSYS`
:   `/proc` 파일 시스템에 접근할 수 없다.

`lutimes()`에서는 다음 오류가 추가로 발생할 수 있다.

`ENOSYS`
:   커널이 이 호출을 지원하지 않는다. 리눅스 2.6.22나 이후 버전이 필요하다.

## VERSIONS

glibc 2.3부터 `futimes()`가 사용 가능하다. glibc 2.6부터 `lutimes()`가 사용 가능한데, 커널 2.6.22부터 지원하는 <tt>[[utimensat(2)]]</tt> 시스템 호출을 이용해 구현되어 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `futimes()`, `lutimes()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 어떤 표준에도 명세되어 있지 않다. 리눅스를 제외하면 BSD 계열에서만 사용 가능하다.

## SEE ALSO

<tt>[[utime(2)]]</tt>, <tt>[[utimensat(2)]]</tt>, <tt>[[symlink(7)]]</tt>

----

2021-03-22
