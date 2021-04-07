## NAME

stat, fstat, lstat, fstatat - 파일 상태 정보 얻기

## SYNOPSIS

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int stat(const char *pathname, struct stat *statbuf);
int fstat(int fd, struct stat *statbuf);
int lstat(const char *pathname, struct stat *statbuf);

#include <fcntl.h>           /* AT_* 상수 정의 */
#include <sys/stat.h>

int fstatat(int dirfd, const char *pathname, struct stat *statbuf,
            int flags);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>lstat()</code>:</dt>
<dd>
<code>/* glibc 2.19 및 이전: */ _BSD_SOURCE</code><br>
<code>    || /* glibc 2.20부터 */ _DEFAULT_SOURCE</code><br>
<code>    || _XOPEN_SOURCE >= 500</code><br>
<code>    || /* glibc 2.10부터: */ _POSIX_C_SOURCE >= 200112L</code>
</dd>
<dt><code>fstatat()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.10부터:</dt>
 <dd><code>_POSIX_C_SOURCE >= 200809L</code></dd>
 <dt>glibc 2.10 전:</dt>
 <dd><code>_ATFILE_SOURCE</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

이 함수들은 파일에 대한 정보를 `statbuf`가 가리키는 버퍼로 반환한다. 그 파일 자체에 대해선 어떤 권한도 필요치 않지만 (`stat()`, `fstatat()`, `lstat()`에서는) `pathname`에서 그 파일까지 이어지는 디렉터리 모두에 대해 실행(탐색) 권한이 필요하다.

`stat()`과 `fstatat()`은 `pathname`이 가리키는 파일에 대한 정보를 가져온다. `fstatat()`의 차이점을 아래에서 설명한다.

`lstat()`은 `stat()`과 동일하되 `pathname`이 심볼릭 링크인 경우에는 대상 파일이 아니라 링크 자체에 대한 정보를 반환한다.

`fstat()`은 `stat()`과 동일하되 어떤 파일에 대한 정보를 가져올지를 파일 디스크립터 `fd`로 지정한다.

### `stat` 구조체

이 시스템 호출들은 모두 `stat` 구조체를 반환하는데, 이는 다음 필드들을 담고 있다.

```c
struct stat {
    dev_t     st_dev;         /* 파일을 담은 장치의 ID */
    ino_t     st_ino;         /* 아이노드 번호 */
    mode_t    st_mode;        /* 파일 종류 및 모드 */
    nlink_t   st_nlink;       /* 하드 링크 수 */
    uid_t     st_uid;         /* 소유자의 사용자 ID */
    gid_t     st_gid;         /* 소유자의 그룹 ID */
    dev_t     st_rdev;        /* 장치 ID (특수 파일인 경우) */
    off_t     st_size;        /* 총 크기, 바이트 단위 */
    blksize_t st_blksize;     /* 파일 시스템 I/O의 블록 크기 */
    blkcnt_t  st_blocks;      /* 할당된 512B 블록 수 */

    /* 리눅스 2.6부터 커널에서 다음 타임스탬프 필드들에
       나노초 정밀도를 지원한다. 리눅스 2.6 전에 대한
       내용은 NOTES 참고. */

    struct timespec st_atim;  /* 최근 접근 시간 */
    struct timespec st_mtim;  /* 최근 수정 시간 */
    struct timespec st_ctim;  /* 최근 상태 변경 시간 */

#define st_atime st_atim.tv_sec      /* 하위 호환성 */
#define st_mtime st_mtim.tv_sec
#define st_ctime st_ctim.tv_sec
};
```

*주의*: `stat` 구조체 내 필드 순서가 아키텍처에 따라 약간 다르다. 더불어 위 정의에는 여러 아키텍처에서 일부 필드들 사이에 있는 패딩 바이트가 나와 있지 않다. 자세한 내용을 알 필요가 있다면 glibc 및 커널 소스 코드를 확인해 보라.

*주의*: 성능과 단순성을 위해 `stat` 구조체의 필드들이 시스템 호출 실행 중의 상이한 시점의 상태 정보를 담을 수 있다. 예를 들어 다른 프로세스에서 <tt>[[chmod(2)]]</tt>나 <tt>[[chown(2)]]</tt>을 호출해서 `st_mode`나 `st_uid`를 바꾸는 경우에 `stat()`이 이전 `st_mode`와 새 `st_uid`를 반환할 수도 있고 이전 `st_uid`와 새 `st_mode`를 반환할 수도 있다.

`stat` 구조체의 필드는 다음과 같다.

<dl>
<dt><code>st_dev</code></dt>
<dd>이 필드는 이 파일이 위치한 장치를 기술한다. (이 필드의 장치 ID를 분해하는 데 <tt>[[major(3)]]</tt> 및 <tt>[[minor(3)]]</tt> 매크로가 유용할 수 있다.)</dd>

<dt><code>st_ino</code></dt>
<dd>이 필드는 파일의 아이노드 번호를 담는다.</dd>

<dt><code>st_mode</code></dt>
<dd>이 필드는 파일 종류와 모드를 담는다. 자세한 내용은 <tt>[[inode(7)]]</tt> 참고.</dd>

<dt><code>st_nlink</code></dt>
<dd>이 필드는 파일에 대한 하드 링크 수를 담는다.</dd>

<dt><code>st_uid</code></dt>
<dd>이 필드는 파일 소유자의 사용자 ID를 담는다.</dd>

<dt><code>st_gid</code></dt>
<dd>이 필드는 파일 그룹 소유자의 ID를 담는다.</dd>

<dt><code>st_rdev</code></dt>
<dd>이 필드는 이 파일이 (아이노드가) 나타내는 장치를 기술한다.</dd>

<dt><code>st_size</code></dt>
<dd>이 필드는 (정규 파일이나 심볼릭 링크인 경우) 바이트 단위 파일 크기를 알려 준다. 심볼릭 링크의 크기란 담고 있는 (종료 널 바이트 없는) 경로명의 길이다.</dd>

<dt><code>st_blksize</code></dt>
<dd>이 필드는 효율적인 파일 시스템 I/O를 위한 "선호" 블록 크기를 알려 준다.</dd>

<dt><code>st_blocks</code></dt>
<dd>이 필드는 파일에 할당된 512바이트 단위 블록 수를 나타낸다. (파일에 구멍이 있을 때는 <code>st_size</code>/512보다 작을 수도 있다.</dd>

<dt><code>st_atime</code></dt>
<dd>파일의 최근 접근 타임스탬프다.</dd>

<dt><code>st_mtime</code></dt>
<dd>파일의 최근 수정 타임스탬프다.</dd>

<dt><code>st_ctime</code></dt>
<dd>파일의 최근 상태 변경 타임스탬프다.</dd>
</dl>

위 필드들에 대한 더 자세한 내용은 <tt>[[inode(7)]]</tt>를 보라.

### `fstatat()`

`fstatat()` 시스템 호출은 파일 정보에 접근하기 위한 범용적 인터페이스로 `stat()`, `lstat()`, `fstat()` 각각의 동작 방식까지 정확하게 제공할 수 있다.

`pathname`에 준 경로명이 상대 경로이면 (상대 경로명에 대해 `stat()` 및 `lstat()`에서 하듯 호출 프로세스의 현재 작업 디렉터리를 기준으로 하는 게 아니라) 파일 디스크립터 `dirfd`가 가리키는 디렉터리를 기준으로 경로명을 해석한다.

`pathname`이 상대 경로이고 `dirfd`가 특수 값 `AT_FDCWD`이면 (`stat()` 및 `lstat()`처럼) 호출 프로세스의 현재 작업 디렉터리를 기준으로 `pathname`을 해석한다.

`pathname`이 절대 경로이면 `dirfd`를 무시한다.

`flags`는 0일 수도 있고 다음 플래그를 1개 이상 OR 해서 담을 수도 있다.

<dl>
<dt><code>AT_EMPTY_PATH</code> (리눅스 2.6.39부터)</dt>
<dd><code>pathname</code>이 빈 문자열이면 (<tt>[[open(2)]]</tt> <code>O_PATH</code> 플래그로 얻은 것일 수도 있는) <code>dirfd</code>가 가리키는 파일에 대해 동작한다. 이 경우에 <code>dirfd</code>는 디렉터리만이 아니라 임의 종류의 파일을 가리킬 수 있으며 <code>fstatat()</code>의 동작 방식은 <code>fstat()</code>과 비슷하다. <code>dirfd</code>가 <code>AT_FDCWD</code>이면 현재 작업 디렉터리에 대해 호출이 동작한다. 이 플래그는 리눅스 전용이다. 이 정의를 얻으려면 <code>_GNU_SOURCE</code>를 정의해야 한다.</dd>

<dt><code>AT_NO_AUTOMOUNT</code> (리눅스 2.6.38부터)</dt>
<dd><code>pathname</code>의 마지막 요소("basename")가 자동 마운트 지점인 디렉터리인 경우에 자동 마운트를 하지 않는다. 이를 통해 (마운트 될 위치가 아니라) 자동 마운트 지점의 속성들을 호출자가 얻을 수 있다. 또한 리눅스 4.14부터는 automounter 간접 맵 등에 쓰이는 on-demand 디렉터리에 실재하지 않는 이름을 만들어 내지 않는다. 디렉터리들을 훑는 도구들에서 이 플래그를 사용해서 자동 마운트 지점인 디렉터리를 잔뜩 자동 마운트 하는 걸 방지할 수 있다. 마운트 지점에 이미 마운트가 됐으면 <code>AT_NO_AUTOMOUNT</code> 플래그에 아무 효력이 없다. 이 플래그는 리눅스 전용이다. 이 정의를 얻으려면 <code>_GNU_SOURCE</code>를 정의해야 한다. <code>stat()</code>과 <code>lstat()</code> 모두 <code>AT_NO_AUTOMOUNT</code>가 설정된 것처럼 동작한다.</dd>

<dt><code>AT_SYMLINK_NOFOLLOW</code></dt>
<dd><code>pathname</code>이 심볼릭 링크인 경우 역참조를 하지 않는다. 대신 <code>lstat()</code>처럼 링크 자체에 대한 정보를 반환한다. (기본적으로 <code>fstatat()</code>은 <code>stat()</code>처럼 심볼릭 링크를 역참조 한다.)</dd>
</dl>

`fstatat()`의 필요성에 대한 설명은 <tt>[[openat(2)]]</tt>을 보라.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EACCES</code></dt>
<dd><code>pathname</code>의 경로 선두부의 한 디렉터리에 대해 탐색 권한이 거부되었다. (<tt>[[path_resolution(7)]]</tt> 참고.)</dd>
<dt><code>EBADF</code></dt>
<dd><code>fd</code>가 유효한 열린 파일 디스크립터가 아니다.</dd>
<dt><code>EFAULT</code></dt>
<dd>잘못된 주소.</dd>
<dt><code>ELOOP</code></dt>
<dd>경로명을 순회하는 동안 너무 많은 심볼릭 링크를 만났다.</dd>
<dt><code>ENAMETOOLONG</code></dt>
<dd><code>pathname</code>이 너무 길다.</dd>
<dt><code>ENOENT</code></dt>
<dd><code>pathname</code>의 어느 요소가 존재하지 않거나 깨진 심볼릭 링크이다.</dd>
<dt><code>ENOENT</code></dt>
<dd><code>pathname</code>이 빈 문자열인데 <code>flags</code>에 <code>AT_EMPTY_PATH</code>를 지정하지 않았다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>메모리 (즉 커널 메모리) 부족.</dd>
<dt><code>ENOTDIR</code></dt>
<dd><code>pathname</code>의 경로 선두부의 어느 요소가 디렉터리가 아니다.</dd>
<dt><code>EOVERFLOW</code></dt>
<dd><code>pathname</code>이나 <code>fd</code>가 그 크기, 아이노드 번호, 블록 수를 각기 <code>off_t</code>, <code>ino_t</code>, <code>blkcnt_t</code> 타입으로 표현할 수 없는 파일을 가리키고 있다. 예를 들어 32비트 플랫폼에서 <code>-D_FILE_OFFSET_BITS=64</code> 없이 컴파일 한 응용이 크기가 <code>(1<<31)-1</code> 바이트를 넘는 파일을 열려고 하는 경우에 이 오류가 발생할 수 있다.</dd>
</dl>

`fstatat()`에서 다음 오류들이 추가로 발생할 수 있다.

<dl>
<dt><code>EBADF</code></dt>
<dd><code>dirfd</code>가 유효한 파일 디스크립터가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>flags</code>에 유효하지 않은 플래그를 지정했다.</dd>
<dt><code>ENOTDIR</code></dt>
<dd><code>pathname</code>이 상대 경로인데 <code>dirfd</code>가 디렉터리 아닌 파일을 가리키는 파일 디스크립터이다.</dd>
</dl>

## VERSIONS

리눅스 커널 2.6.16에서 `fstatat()`이 추가되었다. glibc 버전 2.4에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`stat()`, `fstat()`, `lstat()`: SVr4, 4.3BSD, POSIX.1-2001, POSIX.1-2008.

`fstatat()`: POSIX.1-2008.

POSIX.1-2001에 따르면 심볼릭 링크에 `lstat()`을 하면 `stat` 구조체의 `st_size` 필드와 `st_mode` 필드의 파일 종류에만 유효한 정보를 반환하면 된다. POSIX.1-2008에서는 명세를 더 강화해서 `lstat()`이 `st_mode`의 모드 비트를 제외한 모든 필드들에 유효한 정보를 반환하도록 한다.

`st_blocks` 및 `st_blksize` 필드를 쓰면 이식성이 좀 떨어질 수도 있다. (두 필드는 BSD에서 도입됐다. 시스템에 따라 해석 방식이 다르고, NFS 마운트가 있으면 단일 시스템 내에서도 다를 수 있다.)

## NOTES

### 타임스탬프 필드

구식 커널 및 구식 표준에서는 나노초 타임스탬프 필드를 지원하지 않았다. 대신 `time_t` 타입으로 타임스탬프 필드 세 개(`st_atime`, `st_mtime`, `st_ctime`)가 있어서 초 단위로 타임스탬프를 기록했다.

커널 2.5.48부터 `stat` 구조체의 그 세 가지 타임스탬프 필드에서 나노초 해상도를 지원한다. 적절한 기능 확인 매크로가 정의돼 있으면 각 타임스탬프의 나노초 부분을 `st_atim.tv_nsec` 형태의 이름으로 쓸 수 있다. 나노초 타임스탬프는 POSIX.1-2008에서 표준화됐으며, glibc 버전 2.12부터 `_POSIX_C_SOURCE`가 200809L 이상 값으로 정의돼 있거나 `_XOPEN_SOURCE`가 700 이상 값으로 정의돼 있으면 나노초 부분 이름이 드러난다. glibc 2.19까지에서는 `_BSD_SOURCE`나 `_SVID_SOURCE`가 정의돼 있는 경우에도 나노초 부분의 정의가 드러난다. 앞서 언급한 어떤 매크로도 정의돼 있지 않은 경우에는 `st_atimensec` 형태의 이름으로 나노초 값들을 쓸 수 있다.

### C 라이브러리/커널 차이

시간이 흐르며 `stat` 구조체가 커지면서 세 가지 `stat()` 버전이 생겼다. i386 같은 32비트 플랫폼에서 `sys_stat()` (슬롯 `__NR_oldstat`), `sys_newstat()` (슬롯 `__NR_stat`), 그리고 `sys_stat64()` (슬롯 `__NR_stat64`)이다. 처음 두 버전은 (다른 이름이기는 했지만) 리눅스 1.0에도 있었으며, 마지막 버전은 리눅스 2.4에서 추가됐다. 비슷한 내용이 `fstat()`과 `lstat()`에도 적용된다.

각 버전에서 다루는 커널 내부 `stat` 구조체 버전은 다음과 같다.

<dl>
<dt><code>__old_kernel_stat</code></dt>
<dd>원래 구조체. 필드들이 좀 작고 패딩 없음.</dd>

<dt><code>stat</code></dt>
<dd><code>st_ino</code> 필드가 커지고 향후 확장을 위해 구조체 여러 부분에 패딩이 추가됨.</dd>

<dt><code>stat64</code></dt>
<dd><code>st_ino</code> 필드가 더 커지고, 리눅스 2.4에서 UID 및 GID를 32비트로 확장한 것에 맞춰 <code>st_uid</code> 및 <code>st_gid</code> 필드가 커지고, 여러 다른 필드들이 더 커지고 구조체에 패딩들이 추가됨. (리눅스 2.6에서 32비트 장치 ID와 타임스탬프 나노초 부분이 등장하면서 여러 패딩 바이트가 사라졌다.)</dd>
</dl>

glibc의 `stat()` 래퍼 함수에서 이런 세부 사항을 응용에게 감춰 주고 커널이 제공하는 가장 최신 버전의 시스템 호출을 부르며 구식 바이너리를 위해 필요한 경우 반환된 정보를 다시 포장해 준다.

최신 64비트 시스템에서는 간단하다. `stat()` 시스템 호출이 하나만 있고 커널에서는 필드 크기가 충분히 큰 `stat` 구조체를 사용한다.

glibc의 `fstatat()` 래퍼 함수에서 이용하는 기반 시스템 호출의 실제 이름이 `fstatat64()`이고 일부 아키텍처에서는 `newfstatat()`이다.

## EXAMPLE

다음 프로그램에서는 `lstat()`을 호출해서 반환된 `stat` 구조체의 주요 필드들을 표시한다.

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <time.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/sysmacros.h>

int
main(int argc, char *argv[])
{
    struct stat sb;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s <pathname>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    if (lstat(argv[1], &sb) == -1) {
        perror("lstat");
        exit(EXIT_FAILURE);
    }

    printf("ID of containing device:  [%lx,%lx]\n",
         (long) major(sb.st_dev), (long) minor(sb.st_dev));

    printf("File type:                ");

    switch (sb.st_mode & S_IFMT) {
    case S_IFBLK:  printf("block device\n");            break;
    case S_IFCHR:  printf("character device\n");        break;
    case S_IFDIR:  printf("directory\n");               break;
    case S_IFIFO:  printf("FIFO/pipe\n");               break;
    case S_IFLNK:  printf("symlink\n");                 break;
    case S_IFREG:  printf("regular file\n");            break;
    case S_IFSOCK: printf("socket\n");                  break;
    default:       printf("unknown?\n");                break;
    }

    printf("I-node number:            %ld\n", (long) sb.st_ino);

    printf("Mode:                     %lo (octal)\n",
            (unsigned long) sb.st_mode);

    printf("Link count:               %ld\n", (long) sb.st_nlink);
    printf("Ownership:                UID=%ld   GID=%ld\n",
            (long) sb.st_uid, (long) sb.st_gid);

    printf("Preferred I/O block size: %ld bytes\n",
            (long) sb.st_blksize);
    printf("File size:                %lld bytes\n",
            (long long) sb.st_size);
    printf("Blocks allocated:         %lld\n",
            (long long) sb.st_blocks);

    printf("Last status change:       %s", ctime(&sb.st_ctime));
    printf("Last file access:         %s", ctime(&sb.st_atime));
    printf("Last file modification:   %s", ctime(&sb.st_mtime));

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`ls(1)`, `stat(1)`, <tt>[[access(2)]]</tt>, <tt>[[chmod(2)]]</tt>, <tt>[[chown(2)]]</tt>, <tt>[[readlink(2)]]</tt>, <tt>[[statx(2)]]</tt>, <tt>[[utime(2)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[inode(7)]]</tt>, <tt>[[symlink(7)]]</tt>

----

2019-03-06
