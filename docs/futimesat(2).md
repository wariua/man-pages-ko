## NAME

futimesat - 디렉터리 파일 디스크립터 기준으로 파일의 타임스탬프 바꾸기

## SYNOPSIS

```c
#include <fcntl.h> /* AT_* 상수 정의 */
#include <sys/time.h>

int futimesat(int dirfd, const char *pathname,
              const struct timeval times[2]);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`futimesat()`:
:   `_GNU_SOURCE`

## DESCRIPTION

이 시스템 호출은 구식이다. 대신 <tt>[[utimensat(2)]]</tt>을 사용하라.

`futimesat()` 시스템 호출은 이 매뉴얼 페이지에서 기술하는 차이점들을 제외하면 <tt>[[utimes(2)]]</tt>와 정확히 동일하게 동작한다.

`pathname`으로 준 경로명이 상대적인 경우에는 (<tt>[[utimes(2)]]</tt>에서 상대 경로명에 대해 하듯 호출 프로세스의 현재 작업 디렉터리 기준이 아니라) 파일 디스크립터 `dirfd`가 가리키는 디렉터리를 기준으로 해석한다.

`pathname`이 상대적이고 `dirfd`가 특수한 값 `AT_FDCWD`인 경우에는 (<tt>[[utimes(2)]]</tt>처럼) 호출 프로세스의 현재 작업 디렉터리를 기준으로 `pathname`을 해석한다.

`pathname`이 절대적인 경우에는 `dirfd`를 무시한다.

## RETURN VALUE

성공 시 `futimesat()`은 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<tt>[[utimes(2)]]</tt>에 발생할 수 있는 것과 같은 오류들이 `futimesat()`에도 발생할 수 있다. 그리고 `futimesat()`에는 다음 오류들이 추가로 발생할 수 있다.

`EBADF`
:   `dirfd`가 유효한 파일 디스크립터가 아니다.

`ENOTDIR`
:   `pathname`이 상대적인데 `dirfd`가 디렉터리 아닌 파일을 가리키는 파일 디스크립터이다.

## VERSIONS

리눅스 커널 2.6.16에 `futimeat()`이 추가되었다. glibc 버전 2.6에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

이 시스템 호출은 비표준이다. POSIX.1에 제안되었던 명세에 따라 구현이 이뤄졌는데 그 명세가 <tt>[[utimensat(2)]]</tt> 명세로 교체되었다.

솔라리스에 비슷한 시스템 호출이 있다.

## NOTES

### glibc 참고 사항

`pathname`이 NULL인 경우에 glibc의 `futimesat()` 래퍼 함수는 `dirfd`가 가리키는 파일의 시간들을 갱신한다.

## SEE ALSO

<tt>[[stat(2)]]</tt>, <tt>[[utimensat(2)]]</tt>, <tt>[[utimes(2)]]</tt>, <tt>[[futimes(3)]]</tt>, <tt>[[path_resolution(7)]]</tt>

----

2017-09-15
