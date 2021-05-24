## NAME

mknod, mknodat - 특수 파일이나 일반 파일 만들기

## SYNOPSIS

```c
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int mknod(const char *pathname, mode_t mode, dev_t dev);

#include <fcntl.h>           /* AT_* 상수 정의 */
#include <sys/stat.h>

int mknodat(int dirfd, const char *pathname, mode_t mode, dev_t dev);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`mknod()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.19부터 */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19 */ _BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

`mknod()` 시스템 호출은 이름이 `pathname`이고 `mode`와 `dev`가 나타내는 속성의 파일 시스템 노드(파일, 장치 특수 파일, 이름 있는 파이프)를 만든다.

`mode` 인자는 사용할 파일 모드와 생성할 노드 종류 모두를 나타낸다. 아래 나열하는 파일 종류 중 하나와 <tt>[[inode(7)]]</tt>에 나열된 파일 모드 비트 0개 이상을 (비트 OR로) 조합한 것이어야 한다.

파일 모드가 프로세스의 `umask`에 의해 일반적 방식으로 변경된다. 즉, 기본 ACL이 없다면 생성되는 노드의 권한은 `(mode & ~umask)`다.

파일 종류는 `S_IFREG`, `S_IFCHR`, `S_IFBLK`, `S_IFIFO`, `S_IFSOCK` 중 하나여야 하며, 각각 (빈 채로 생성되는) 정규 파일, 문자 특수 파일, 블록 특수 파일, FIFO (이름 있는 파이프), 유닉스 도메인 소켓이다. (파일 종류가 0인 것은 `S_IFREG`와 동등하다.)

파일 종류가 `S_IFCHR`나 `S_IFBLK`인 경우 `dev`는 새로 생성되는 장치 특수 파일의 주번호와 부번호를 나타낸다. (`dev` 값을 만드는 데 <tt>[[makedev(3)]]</tt>가 유용할 수 있다.) 그 외 경우에는 무시된다.

`pathname`이 이미 존재하거나 심볼릭 링크면 호출이 `EEXIST` 오류로 실패한다.

새로 생성되는 노드는 프로세스의 실효 사용자 ID가 소유하게 된다. 노드를 담은 디렉터리에 set-group-ID 비트가 설정돼 있거나 파일 시스템이 BSD 그룹 동작 방식으로 마운트돼 있다면 새 노드가 부모 디렉터리의 그룹 소유자를 물려받게 된다. 아니라면 프로세스의 실효 그룹 ID가 소유하게 된다.

### `mknodat()`

`mknodat()` 시스템 호출은 아래 설명하는 차이를 빼면 `mknod()`와 똑같은 방식으로 동작한다.

`pathname`의 경로명이 상대적인 경우에는 (`mknod()`에서 하듯 호출 프로세스의 현재 작업 디렉터리 기준이 아니라) 파일 디스크립터 `dirfd`가 가리키는 디렉터리를 기준으로 해석한다.

`pathname`이 상대적이고 `dirfd`가 특수한 값 `AT_FDCWD`인 경우에는 (`mknod()`처럼) 호출 프로세스의 현재 작업 디렉터리를 기준으로 `pathname`을 해석한다.

`pathname`이 절대적인 경우에는 `dirfd`를 무시한다.

`mknodat()`의 필요성에 대한 설명은 <tt>[[openat(2)]]</tt>을 보라.

## RETURN VALUE

`mknod()`와 `mknodat()`은 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   부모 디렉터리에서 프로세스에 쓰기 권한을 허용하지 않거나, `pathname`의 경로 선두부의 한 디렉터리에서 탐색 권한을 허용하지 않았다. (<tt>[[path_resolution(7)]]</tt>도 참고.)

`EDQUOT`
:   파일 시스템에서 사용자의 디스크 블록 내지 아이노드 쿼터가 고갈되었다.

`EEXIST`
:   `pathname`이 이미 존재한다. `pathname`이 (깨진 것이든 아니든) 심볼릭 링크인 경우도 포함된다.

`EFAULT`
:   `pathname`이 접근 가능한 주소 공간 밖을 가리킨다.

`EINVAL`
:   `mode`로 정규 파일, 장치 특수 파일, FIFO, 소켓 외의 뭔가를 생성하라고 요청했다.

`ELOOP`
:   `pathname`을 해석하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   `pathname`이 너무 길다.

`ENOENT`
:   `pathname`의 어느 부분이 존재하지 않거나 깨진 심볼릭 링크다.

`ENOMEM`
:   사용 가능한 커널 메모리가 충분하지 않다.

`ENOSPC`
:   `pathname`을 담을 장치에 새 노드를 위한 여유 공간이 없다.

`ENOTDIR`
:   `pathname`에서 디렉터리로 쓰인 부분이 실제로는 디렉터리가 아니다.

`EPERM`
:   `mode`에서 정규 파일, FIFO (이름 있는 파이프), 유닉스 도메인 소켓 아닌 뭔가를 생성하라고 요청했으며 호출자에게 특권이 없다. (리눅스: `CAP_MKNOD` 역능이 없다.) 요청한 노드 종류를 `pathname`을 담을 파일 시스템에서 지원하지 않는 경우에도 반환된다.

`EROFS`
:   `pathname`이 가리키는 파일이 읽기 전용 파일 시스템에 있다.

`mknodat()`에서 추가로 다음 오류가 발생할 수 있다.

`EBADF`
:   `dirfd`가 유효한 파일 디스크립터가 아니다.

`ENOTDIR`
:   `pathname`이 상대 경로이고 `dirfd`가 디렉터리 아닌 파일을 가리키는 파일 디스크립터다.

## VERSIONS

리눅스 커널 2.6.16에서 `mknodat()`이 추가되었다. glibc 버전 2.4에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`mknod()`: SVr4, 4.4BSD, POSIX.1-2001 (하지만 아래 참고), POSIX.1-2008.

`mknodat()`: POSIX.1-2008.

## NOTES

POSIX.1-2001에 따르면: "유일하게 이식성 있는 `mknod()` 사용 방식은 FIFO 특수 파일을 만드는 것이다. `mode`가 `S_IFIFO`가 아니거나 `dev`가 0이 아닐 때 `mknod()`의 동작은 명세돼 있지 않다." 하지만 요즘은 그런 용도에 `mknod()`를 절대 쓰지 않는 게 좋다. 대신 특별히 그 용도로 규정된 함수인 <tt>[[mkfifo(3)]]</tt>를 쓰는 게 좋다.

리눅스에서는 디렉터리를 만드는 데 `mknod()`를 쓸 수 없다. <tt>[[mkdir(2)]]</tt>로 디렉터리를 만드는 게 좋다.

NFS의 기반 프로토콜에 여러 부적합한 점이 있어서 그 중 일부가 `mknod()`와 `mknodat()`에 영향을 끼친다.

## SEE ALSO

`mknod(1)`, <tt>[[chmod(2)]]</tt>, <tt>[[chown(2)]]</tt>, <tt>[[fcntl(2)]]</tt>, <tt>[[mkdir(2)]]</tt>, <tt>[[mount(2)]]</tt>, <tt>[[socket(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[umask(2)]]</tt>, <tt>[[unlink(2)]]</tt>, <tt>[[makedev(3)]]</tt>, <tt>[[mkfifo(3)]]</tt>, `acl(5)`, <tt>[[path_resolution(7)]]</tt>

----

2021-03-22
