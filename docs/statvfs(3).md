## NAME

statvfs, fstatvfs - 파일 시스템 통계 얻기

## SYNOPSIS

```c
#include <sys/statvfs.h>

int statvfs(const char *restrict path, struct statvfs *restrict buf);
int fstatvfs(int fd, struct statvfs *buf);
```

## DESCRIPTION

`statvfs()` 함수는 마운트 된 파일 시스템에 대한 정보를 반환한다. `path`는 마운트 된 파일 시스템 내 임의 파일의 경로명이다. `buf`는 대략 다음처럼 정의돼 있는 `statvfs` 구조체에 대한 포인터다.

```c
struct statvfs {
    unsigned long  f_bsize;    /* 파일 시스템 블록 크기 */
    unsigned long  f_frsize;   /* 단편 크기 */
    fsblkcnt_t     f_blocks;   /* f_frsize 단위의 fs 크기 */
    fsblkcnt_t     f_bfree;    /* 유휴 블록 수 */
    fsblkcnt_t     f_bavail;   /* 비특권 사용자를 위한
                                  유휴 블록 수 */
    fsfilcnt_t     f_files;    /* 아이노드 수 */
    fsfilcnt_t     f_ffree;    /* 유휴 아이노드 수 */
    fsfilcnt_t     f_favail;   /* 비특권 사용자를 위한
                                  유휴 아이노드 수 */
    unsigned long  f_fsid;     /* 파일 시스템 ID */
    unsigned long  f_flag;     /* 마운트 플래그 */
    unsigned long  f_namemax;  /* 파일명 최대 길이 */
};
```

타입 `fsblkcnt_t`와 `fsfilcnt_t`는 `<sys/types.h>`에 정의돼 있다. 둘 모두 예전에는 `unsigned long`이었다.

`f_flag` 필드는 이 파일 시스템을 마운트 할 때 사용한 여러 옵션들을 나타내는 비트 마스크이다. 다음 비트를 0개 이상 담는다.

`ST_MANDLOCK`
:   파일 시스템 상에서 강제적 락킹을 허용한다. (<tt>[[fcntl(2)]]</tt> 참고.)

`ST_NOATIME`
:   접근 시간을 갱신하지 않는다. <tt>[[mount(2)]]</tt> 참고.

`ST_NODEV`
:   파일 시스템 상에서 장치 특수 파일에 대한 접근을 불허한다.

`ST_NODIRATIME`
:   디렉터리 접근 시간을 갱신하지 않는다. <tt>[[mount(2)]]</tt> 참고.

`ST_NOEXEC`
:   파일 시스템 상에서 프로그램 실행을 불허한다.

`ST_NOSUID`
:   파일 시스템 상의 실행 파일에 대해 <tt>[[exec(3)]]</tt>에서 set-user-ID 및 set-group-ID 비트를 무시한다.

`ST_RDONLY`
:   파일 시스템이 읽기 전용으로 마운트 돼 있다.

`ST_RELATIME`
:   mtime/ctime에 따라서 atime을 갱신한다. <tt>[[mount(2)]]</tt> 참고.

`ST_SYNCHRONOUS`
:   쓰기를 파일 시스템으로 즉시 동기화한다. (<tt>[[open(2)]]</tt>의 `O_SYNC` 설명 참고.)

모든 파일 시스템에서 반환 구조체의 모든 필드들에 유의미한 값이 있는지 여부는 명세돼 있지 않다.

`fstatvfs()`는 디스크립터 `fd`가 가리키는 열린 파일에 대해서 같은 정보를 반환한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   (`statvfs()`) `path`의 경로 선두부의 어느 요소에 대해 탐색 권한이 거부되었다. (<tt>[[path_resolution(7)]]</tt>도 참고.)

`EBADF`
:   (`fstatvfs()`) `fd`가 유효한 열린 파일 디스크립터가 아니다.

`EFAULT`
:   `buf`나 `path`가 유효하지 않은 주소를 가리키고 있다.

`EINTR`
:   호출이 시그널에 의해 중단되었다. <tt>[[signal(7)]]</tt> 참고.

`EIO`
:   파일 시스템에서 읽기를 하던 중 I/O 오류가 발생했다.

`ELOOP`
:   (`statvfs()`) `path`를 변환하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   (`statvfs()`) `path`가 너무 길다.

`ENOENT`
:   (`statvfs()`) `path`가 가리키는 파일이 존재하지 않는다.

`ENOMEM`
:   사용 가능한 커널 메모리가 충분하지 않다.

`ENOSYS`
:   파일 시스템에서 이 호출을 지원하지 않는다.

`ENOTDIR`
:   (`statvfs()`) `path`의 경로 선두부의 한 요소가 디렉터리가 아니다.

`EOVERFLOW`
:   일부 값이 너무 커서 반환 구조체로 표현할 수 없다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `statvfs()`, `fstatvfs()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

`f_flag` 필드의 `ST_NOSUID` 및 `ST_RDONLY` 플래그만 POSIX.1에 명세돼 있다. 다른 필드들의 정의를 얻으려면 `_GNU_SOURCE`를 정의해야 한다.

## NOTES

리눅스 커널에 있는 시스템 호출 <tt>[[statfs(2)]]</tt> 및 <tt>[[fstatfs(2)]]</tt>가 이 라이브러리 호출을 뒷받침한다.

glibc 버전 2.13 전의 `statvfs()`에서는 `/proc/mounts`에 나오는 마운트 옵션들을 훑어서 `f_flag` 필드의 비트들을 채웠다. 하지만 리눅스 2.6.36부터 기반 시스템 호출 <tt>[[statfs(2)]]</tt>가 `f_flags` 필드를 통해 필요한 정보를 제공해 주며, 그래서 glibc 버전 2.13부터는 `statvfs()` 함수에서 `/proc/mounts`를 훑는 대신 그 필드의 정보를 이용하게 된다.

다음 호출의 glibc 구현에서는 `path`를 인자로 `statvfs()`를 호출해서 반환된 `f_frsize`, `f_frsize`, `f_bsize` 필드를 각각 사용한다.

```c
pathconf(path, _PC_REC_XFER_ALIGN);
pathconf(path, _PC_ALLOC_SIZE_MIN);
pathconf(path, _PC_REC_MIN_XFER_SIZE);
```

## SEE ALSO

<tt>[[statfs(2)]]</tt>

----

2021-03-22
