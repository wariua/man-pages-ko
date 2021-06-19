## NAME

mkfifo, mkfifoat - FIFO 특수 파일 (이름 있는 파이프) 만들기

## SYNOPSIS

```c
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);

#include <fcntl.h>           /* AT_* 상수 정의 */
#include <sys/stat.h>

int mkfifoat(int dirfd, const char *pathname, mode_t mode);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`mkfifoat()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_ATFILE_SOURCE`

## DESCRIPTION

`mkfifo()`는 이름이 `pathname`인 FIFO 특수 파일을 만든다. `mode`는 그 FIFO의 권한을 나타낸다. 프로세스의 `umask`에 의해 일반적 방식으로 변경된다. 즉, 생성 파일의 권한은 `(mode & ~umask)`이다.

FIFO 특수 파일은 파이프와 비슷하되 생성되는 방식이 다르다. 이름 없는 통신 채널 형태가 아니라 `mkfifo()` 호출로 파일 시스템에 FIFO 특수 파일을 만든다.

이렇게 FIFO 특수 파일을 만들고 나면 일반 파일과 같은 방식으로 프로세스에서 열어서 읽고 쓸 수 있다. 하지만 파일에 입력 내지 출력 동작을 진행할 수 있으려면 양쪽이 동시에 열려 있어야 한다. FIFO를 읽기용으로 열면 어떤 다른 프로세스가 같은 FIFO를 쓰기용으로 열기 전까지 블록하게 되며, 반대도 마찬가지다. FIFO 특수 파일을 논블로킹 방식으로 다루는 방식은 <tt>[[fifo(7)]]</tt>를 보라.

### `mkfifoat()`

`mkfifoat()` 함수는 아래 설명하는 차이를 빼면 `mkfifo()`와 똑같은 방식으로 동작한다.

`pathname`의 경로명이 상대적인 경우에는 (`mkfifo()`에서 하듯 호출 프로세스의 현재 작업 디렉터리 기준이 아니라) 파일 디스크립터 `dirfd`가 가리키는 디렉터리를 기준으로 해석한다.

`pathname`이 상대적이고 `dirfd`가 특수한 값 `AT_FDCWD`인 경우에는 (`mkfifo()`처럼) 호출 프로세스의 현재 작업 디렉터리를 기준으로 `pathname`을 해석한다.

`pathname`이 절대적인 경우에는 `dirfd`를 무시한다.

## RETURN VALUE

`mkfifo()`와 `mkfifoat()`은 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   `pathname`의 디렉터리들 중 하나에서 탐색(실행) 권한을 허용하지 않는다.

`EDQUOT`
:   파일 시스템에서 사용자의 디스크 블록 내지 아이노드 쿼터가 고갈되었다.

`EEXIST`
:   `pathname`이 이미 존재한다. `pathname`이 (깨진 것이든 아니든) 심볼릭 링크인 경우도 포함된다.

`ENAMETOOLONG`
:   `pathname`의 총 길이가 `PATH_MAX`보다 크거나 개별 경로 부분의 길이가 `NAME_MAX`보다 크다. GNU 시스템에선 전체 파일명 길이에 어떤 제한도 두지 않지만 일부 파일 시스템에서 경로 부분의 길이에 제한을 둘 수도 있다.

`ENOENT`
:   `pathname`의 어느 부분이 존재하지 않거나 깨진 심볼릭 링크다.

`ENOSPC`
:   디렉터리나 파일 시스템에 새 파일을 위한 여유 공간이 없다.

`ENOTDIR`
:   `pathname`에서 디렉터리로 쓰인 부분이 실제로는 디렉터리가 아니다.

`EROFS`
:   `pathname`이 읽기 전용 파일 시스템을 가리키고 있다.

`mkfifoat()`에서 추가로 다음 오류가 발생할 수 있다.

`EBADF`
:   `dirfd`가 유효한 파일 디스크립터가 아니다.

`ENOTDIR`
:   `pathname`이 상대 경로이고 `dirfd`가 디렉터리 아닌 파일을 가리키는 파일 디스크립터다.

## VERSIONS

glibc 버전 2.4에서 `mkfifoat()`이 추가되었다. 리눅스 커널 2.6.16부터 이용 가능한 <tt>[[mknodat(2)]]</tt>을 이용해 구현돼 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mkfifo()`, `mkfifoat()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`mkfifo()`: POSIX.1-2001, POSIX.1-2008.

`mkfifoat()`: POSIX.1-2008.

## SEE ALSO

`mkfifo(1)`, <tt>[[close(2)]]</tt>, <tt>[[open(2)]]</tt>, <tt>[[read(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[umask(2)]]</tt>, <tt>[[write(2)]]</tt>, <tt>[[fifo(7)]]</tt>

----

2021-03-22
