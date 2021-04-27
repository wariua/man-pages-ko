## NAME

statfs, fstatfs - 파일 시스템 통계 얻기

## SYNOPSIS

```c
#include <sys/vfs.h>    /* 또는 <sys/statfs.h> */

int statfs(const char *path, struct statfs *buf);
int fstatfs(int fd, struct statfs *buf);
```

## DESCRIPTION

`statfs()` 시스템 호출은 마운트 된 파일 시스템에 대한 정보를 반환한다. `path`는 마운트 된 파일 시스템 내 임의 파일의 경로명이다. `buf`는 대략 다음처럼 정의돼 있는 `statfs` 구조체의 포인터다.

```c
struct statfs {
    __fsword_t f_type;    /* 파일 시스템 유형 (아래 참고) */
    __fsword_t f_bsize;   /* 최적 전송 블록 크기 */
    fsblkcnt_t f_blocks;  /* 파일 시스템 내 총 데이터 블록 */
    fsblkcnt_t f_bfree;   /* 파일 시스템 내 유휴 블록 */
    fsblkcnt_t f_bavail;  /* 비특권 사용자가 사용 가능한
                             유휴 블록 */
    fsfilcnt_t f_files;   /* 파일 시스템 내 총 아이노드 */
    fsfilcnt_t f_ffree;   /* 파일 시스템 내 유휴 아이노드 */
    fsid_t     f_fsid;    /* 파일 시스템 ID */
    __fsword_t f_namelen; /* 파일명 최대 길이 */
    __fsword_t f_frsize;  /* 단편 크기 (리눅스 2.6부터) */
    __fsword_t f_flags;   /* 파일 시스템 마운트 플래그
                             (리눅스 2.6.36부터) */
    __fsword_t f_spare[xxx];
                    /* 향후 용도를 위해 예비된 패딩 바이트 */
};
```

`f_type`에 다음 파일 시스템 유형이 나올 수 있다.

```c
ADFS_SUPER_MAGIC      0xadf5
AFFS_SUPER_MAGIC      0xadff
AFS_SUPER_MAGIC       0x5346414f
ANON_INODE_FS_MAGIC   0x09041934 /* 익명 아이노드 FS (이름 없는
                                    가상 파일들을 위한 FS.
                                    예: epoll, signalfd, bpf) */
AUTOFS_SUPER_MAGIC    0x0187
BDEVFS_MAGIC          0x62646576
BEFS_SUPER_MAGIC      0x42465331
BFS_MAGIC             0x1badface
BINFMTFS_MAGIC        0x42494e4d
BPF_FS_MAGIC          0xcafe4a11
BTRFS_SUPER_MAGIC     0x9123683e
BTRFS_TEST_MAGIC      0x73727279
CGROUP_SUPER_MAGIC    0x27e0eb   /* Cgroup 가상 FS */
CGROUP2_SUPER_MAGIC   0x63677270 /* Cgroup v2 가상 FS */
CIFS_MAGIC_NUMBER     0xff534d42
CODA_SUPER_MAGIC      0x73757245
COH_SUPER_MAGIC       0x012ff7b7
CRAMFS_MAGIC          0x28cd3d45
DEBUGFS_MAGIC         0x64626720
DEVFS_SUPER_MAGIC     0x1373     /* 리눅스 2.6.17 및 이전 */
DEVPTS_SUPER_MAGIC    0x1cd1
ECRYPTFS_SUPER_MAGIC  0xf15f
EFIVARFS_MAGIC        0xde5e81e4
EFS_SUPER_MAGIC       0x00414a53
EXT_SUPER_MAGIC       0x137d     /* 리눅스 2.0 및 이전 */
EXT2_OLD_SUPER_MAGIC  0xef51
EXT2_SUPER_MAGIC      0xef53
EXT3_SUPER_MAGIC      0xef53
EXT4_SUPER_MAGIC      0xef53
F2FS_SUPER_MAGIC      0xf2f52010
FUSE_SUPER_MAGIC      0x65735546
FUTEXFS_SUPER_MAGIC   0xbad1dea  /* 사용 안 함 */
HFS_SUPER_MAGIC       0x4244
HOSTFS_SUPER_MAGIC    0x00c0ffee
HPFS_SUPER_MAGIC      0xf995e849
HUGETLBFS_MAGIC       0x958458f6
ISOFS_SUPER_MAGIC     0x9660
JFFS2_SUPER_MAGIC     0x72b6
JFS_SUPER_MAGIC       0x3153464a
MINIX_SUPER_MAGIC     0x137f     /* 원판 미닉스 FS */
MINIX_SUPER_MAGIC2    0x138f     /* 30문자 미닉스 FS */
MINIX2_SUPER_MAGIC    0x2468     /* 미닉스 V2 FS */
MINIX2_SUPER_MAGIC2   0x2478     /* 미닉스 V2 FS, 30문자 이름 */
MINIX3_SUPER_MAGIC    0x4d5a     /* 미닉스 V3 FS, 60문자 이름 */
MQUEUE_MAGIC          0x19800202 /* POSIX 메시지 큐 FS */
MSDOS_SUPER_MAGIC     0x4d44
MTD_INODE_FS_MAGIC    0x11307854
NCP_SUPER_MAGIC       0x564c
NFS_SUPER_MAGIC       0x6969
NILFS_SUPER_MAGIC     0x3434
NSFS_MAGIC            0x6e736673
NTFS_SB_MAGIC         0x5346544e
OCFS2_SUPER_MAGIC     0x7461636f
OPENPROM_SUPER_MAGIC  0x9fa1
OVERLAYFS_SUPER_MAGIC 0x794c7630
PIPEFS_MAGIC          0x50495045
PROC_SUPER_MAGIC      0x9fa0     /* /proc FS */
PSTOREFS_MAGIC        0x6165676c
QNX4_SUPER_MAGIC      0x002f
QNX6_SUPER_MAGIC      0x68191122
RAMFS_MAGIC           0x858458f6
REISERFS_SUPER_MAGIC  0x52654973
ROMFS_MAGIC           0x7275
SECURITYFS_MAGIC      0x73636673
SELINUX_MAGIC         0xf97cff8c
SMACK_MAGIC           0x43415d53
SMB_SUPER_MAGIC       0x517b
SMB2_MAGIC_NUMBER     0xfe534d42
SOCKFS_MAGIC          0x534f434b
SQUASHFS_MAGIC        0x73717368
SYSFS_MAGIC           0x62656572
SYSV2_SUPER_MAGIC     0x012ff7b6
SYSV4_SUPER_MAGIC     0x012ff7b5
TMPFS_MAGIC           0x01021994
TRACEFS_MAGIC         0x74726163
UDF_SUPER_MAGIC       0x15013346
UFS_MAGIC             0x00011954
USBDEVICE_SUPER_MAGIC 0x9fa2
V9FS_MAGIC            0x01021997
VXFS_SUPER_MAGIC      0xa501fcf5
XENFS_SUPER_MAGIC     0xabba1974
XENIX_SUPER_MAGIC     0x012ff7b4
XFS_SUPER_MAGIC       0x58465342
_XIAFS_SUPER_MAGIC    0x012fd16d /* 리눅스 2.0 및 이전 */
```

이 MAGIC 상수들 대부분은 `/usr/include/linux/magic.h`에 정의돼 있으며 일부는 커널 소스에 하드코딩 돼 있다.

`f_flags` 필드는 파일 시스템 마운트 옵션을 나타내는 비트 마스크이다. 다음 비트를 0개 이상 담는다.

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

`ST_NOSYMFOLLOW` (리눅스 5.10부터)
:   경로를 해석할 때 심볼릭 링크를 따라가지 않는다. <tt>[[mount(2)]]</tt> 참고.

`f_fsid`에 뭐가 들어가야 하는지는 아무도 모른다. (하지만 아래 참고.)

해당 파일 시스템에서 규정돼 있지 않은 필드는 0으로 설정된다.

`fstatfs()`는 디스크립터 `fd`가 가리키는 열린 파일에 대해서 같은 정보를 반환한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   (`statfs()`) `path`의 경로 선두부의 어느 요소에 대해 탐색 권한이 거부되었다. (<tt>[[path_resolution(7)]]</tt>도 참고.)

`EBADF`
:   (`fstatfs()`) `fd`가 유효한 열린 파일 디스크립터가 아니다.

`EFAULT`
:   `buf`나 `path`가 유효하지 않은 주소를 가리키고 있다.

`EINTR`
:   호출이 시그널에 의해 중단되었다. <tt>[[signal(7)]]</tt> 참고.

`EIO`
:   파일 시스템에서 읽기를 하던 중 I/O 오류가 발생했다.

`ELOOP`
:   (`statfs()`) `path`를 변환하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   (`statfs()`) `path`가 너무 길다.

`ENOENT`
:   (`statfs()`) `path`가 가리키는 파일이 존재하지 않는다.

`ENOMEM`
:   사용 가능한 커널 메모리가 충분하지 않다.

`ENOSYS`
:   파일 시스템에서 이 호출을 지원하지 않는다.

`ENOTDIR`
:   (`statfs()`) `path`의 경로 선두부의 한 요소가 디렉터리가 아니다.

`EOVERFLOW`
:   일부 값이 너무 커서 반환 구조체로 표현할 수 없다.

## CONFORMING TO

리눅스 전용. 리눅스의 `statfs()`는 4.4BSD의 버전에서 영감을 얻었다. (하지만 같은 구조체를 쓰지는 않는다.)

## NOTES

`statfs` 구조체 정의의 여러 필드에서 쓰는 `__fsword_t` 타입은 glibc 내부용 타입이지 외부 사용 용도가 아니다. 이 때문에 프로그램 내에서 그 필드들을 지역 변수로 복사하거나 비교하려 할 때 좀 어려운 문제가 생긴다. 대다수 시스템에서는 그런 변수에 `unsigned int`를 쓰면 충분하다.

리눅스의 원판 `statfs()` 및 `fstatfs()` 시스템 호출은 극히 큰 파일 크기를 염두에 두고 설계되지 않았다. 그래서 리눅스 2.6에서 새로운 구조체 `statfs64`를 이용하는 새 시스템 호출 `statfs64()` 및 `fstatfs64()`가 추가됐다. 새 구조체에는 원래의 `statfs` 구조체와 같은 필드들이 있되 큰 파일 크기를 담을 수 있도록 여러 필드들의 크기가 커졌다. glibc의 `statfs()` 및 `fstatfs()` 래퍼 함수에서 커널 차이를 투명하게 처리해 준다.

어떤 시스템에는 `<sys/vfs.h>`만 있고 다른 시스템에는 `<sys/statfs.h>`가 있으며 전자가 후자를 포함한다. 따라서 전자를 포함하는 게 최선일 것 같다.

LSB에서는 라이브러리 호출 `statfs()` 및 `fstatfs()`를 쓰지 말고 대신 <tt>[[statvfs(3)]]</tt> 및 <tt>[[fstatvfs(3)]]</tt>를 쓰라고 한다.

### `f_fsid` 필드

솔라리스, Irix, POSIX에 있는 시스템 호출 <tt>[[statvfs(2)]]</tt>는 `struct statvfs`(`</sys/statvfs.h>`에 정의돼 있음)를 반환하고 거기에 `unsigned long f_fsid`가 있다. 리눅스, SunOS, HP-UX, 4.4BSD에 있는 시스템 호출 `statfs()`는 `struct statfs`(`<sys/vfs.h>`에 정의돼 있음)를 반환하고 거기에 `fsid_t f_fsid`가 있으며, `fsid_t`는 `struct { int val[2]; }`로 정의돼 있다. FreeBSD도 마찬가지이되 헤더 파일 `<sys/mount.h>`를 쓴다.

기본 개념은 `f_fsid`에 어떤 임의의 내용물이 있어서 (`f_fsid`,`ino`) 쌍이 파일을 유일하게 결정해 준다는 것이다. 어떤 운영 체제에서는 장치 번호를 (또는 그걸 변형해서) 쓰거나 장치 번호에 파일 시스템 타입을 합쳐서 쓴다. 여러 운영 체제에서는 `f_fsid` 필드를 수퍼유저에게만 제공한다. (비특권 사용자에게는 0으로 채운다.) NFS로 내보낼 때 파일 시스템의 파일 핸들에 그 필드를 사용하므로 그 값을 알려 주는 게 보안 우려 사항이기 때문이다.

일부 운영 체제에서는 그 `fsid`를 <tt>[[sysfs(2)]]</tt> 시스템 호출 두 번째 인자로 쓸 수 있다.

## BUGS

리눅스 2.6.38부터 리눅스 3.1까지에서는 <tt>[[pipe(2)]]</tt>로 생성된 파일 디스크립터에 대해 `fstatfs()`가 `ENOSYS` 오류로 실패한다.

## SEE ALSO

<tt>[[stat(2)]]</tt>, <tt>[[statvfs(3)]]</tt>, <tt>[[path_resolution(7)]]</tt>

----

2021-03-22
