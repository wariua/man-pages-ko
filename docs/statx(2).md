## NAME

statx - 파일 상태 정보 얻기 (확장)

## SYNOPSIS

```c
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>           /* AT_* 상수 정의 */

int statx(int dirfd, const char *restrict pathname, int flags,
          unsigned int mask, struct statx *restrict statxbuf);
```

## DESCRIPTION

이 함수는 파일에 대한 정보를 `statxbuf`가 가리키는 버퍼에 저장해서 반환한다. 반환 버퍼의 구조는 다음과 같다.

```c
struct statx {
    __u32 stx_mask;        /* 채워진 필드들을 나타내는
                              비트 마스크 */
    __u32 stx_blksize;     /* 파일 시스템 I/O의 블록 크기 */
    __u64 stx_attributes;  /* 추가 파일 속성 표시 */
    __u32 stx_nlink;       /* 하드 링크 수 */
    __u32 stx_uid;         /* 소유자의 사용자 ID */
    __u32 stx_gid;         /* 소유자의 그룹 ID */
    __u16 stx_mode;        /* 파일 종류 및 모드 */
    __u64 stx_ino;         /* 아이노드 번호 */
    __u64 stx_size;        /* 총 크기, 바이트 단위 */
    __u64 stx_blocks;      /* 할당된 512B 블록 수 */
    __u64 stx_attributes_mask;
                           /* stx_attributes에 무엇을
                              기대하는지 나타내는 마스크 */

    /* 다음은 파일 타임스탬프 필드들 */
    struct statx_timestamp stx_atime;  /* 최근 접근 */
    struct statx_timestamp stx_btime;  /* 생성 */
    struct statx_timestamp stx_ctime;  /* 최근 상태 변경 */
    struct statx_timestamp stx_mtime;  /* 최근 수정 */

    /* 이 파일이 장치를 나타내는 경우 다음 두 필드가
       그 장치의 ID를 담는다 */
    __u32 stx_rdev_major;  /* 주 ID */
    __u32 stx_rdev_minor;  /* 부 ID */

    /* 다음 두 필드는 파일이 위치한 파일 시스템이
       들어 있는 장치의 ID를 담는다 */
    __u32 stx_dev_major;   /* 주 ID */
    __u32 stx_dev_minor;   /* 부 ID */
};
```

파일 타임스탬프의 구조는 다음과 같다.

```c
struct statx_timestamp {
    __s64 tv_sec;    /* 에포크 이후 초 (유닉스 시간) */
    __u32 tv_nsec;   /* tv_sec 이후 나노초 */
};
```

(참고로 예비 공간과 패딩은 생략돼 있다.)

### `statx()` 호출하기

파일 상태 정보 접근 시 그 파일 자체에 대해선 어떤 권한도 필요치 않지만 경로명이 있는 `statx()`의 경우에는 `pathname`에서 그 파일까지 이어지는 디렉터리 모두에 대해 실행(탐색) 권한이 필요하다.

`statx()`에서는 `pathname`, `dirfd`, `flags`를 사용해 다음 방식 중 하나로 대상 파일을 식별한다.

절대 경로명
:   `pathname`이 슬래시로 시작하면 대상 파일을 나타내는 절대 경로명이다. 이 경우 `dirfd`는 무시한다.

상대 경로명
:   `pathname`이 슬래시 아닌 문자로 시작하는 문자열이고 `dirfd`가 `AT_FDCWD`이면 `pathname`이 상대 경로명이고 프로세스의 현재 작업 디렉터리를 기준으로 해석한다.

디렉터리 기준 상태 경로명
:   `pathname`이 슬래시 아닌 문자로 시작하는 문자열이고 `dirfd`가 디렉터리를 가리키는 파일 디스크립터이면 `pathname`이 상대 경로명이고 `dirfd`가 가리키는 디렉터리를 기준으로 해석한다.

파일 디스크립터
:   `pathname`이 빈 문자열이고 `flags`에 `AT_EMPTY_PATH` 플래그(아래 참고)가 지정돼 있으면 파일 디스크립터 `dirfd`가 가리키는 파일이 대상 파일이다.

`flags`를 이용해 경로명 기반 탐색 동작에 영향을 줄 수 있다. `flags` 값은 다음 상수를 0개 이상 OR 해서 구성한다.

`AT_EMPTY_PATH`
:   `pathname`이 빈 문자열이면 (<tt>[[open(2)]]</tt> `O_PATH` 플래그로 얻은 것일 수도 있는) `dirfd`가 가리키는 파일에 대해 동작한다. 이 경우에 `dirfd`는 디렉터리만이 아니라 임의 종류의 파일을 가리킬 수 있다.

    `dirfd`가 `AT_FDCWD`이면 현재 작업 디렉터리에 대해 호출이 동작한다.

    이 플래그는 리눅스 전용이다. 이 정의를 얻으려면 `_GNU_SOURCE`를 정의해야 한다.

`AT_NO_AUTOMOUNT`
:   `pathname`의 마지막 요소("basename")가 자동 마운트 지점인 디렉터리인 경우에 자동 마운트를 하지 않는다. 이를 통해 (마운트 될 위치가 아니라) 자동 마운트 지점의 속성들을 호출자가 얻을 수 있다. 디렉터리들을 훑는 도구들에서 이 플래그를 사용해서 자동 마운트 지점인 디렉터리를 잔뜩 자동 마운트 하는 걸 방지할 수 있다. 마운트 지점에 이미 마운트가 됐으면 `AT_NO_AUTOMOUNT` 플래그에 아무 효력이 없다. 이 플래그는 리눅스 전용이다. 이 정의를 얻으려면 `_GNU_SOURCE`를 정의해야 한다.

`AT_SYMLINK_NOFOLLOW`
:   `pathname`이 심볼릭 링크인 경우 따라가지 않는다. 대신 `lstat()`처럼 링크 자체에 대한 정보를 반환한다.

또한 `flags`를 이용해 원격 파일 시스템의 파일을 질의할 때 커널에서 하게 되는 동기화 방식을 제어할 수 있다. 다음 값들 중 하나를 OR 한다.

`AT_STATX_SYNC_AS_STAT`
:   <tt>[[stat(2)]]</tt>과 똑같이 한다. 기본 방식이며 파일 시스템에 따라 달라진다.

`AT_STATX_FORCE_SYNC`
:   속성들이 서버와 동기화되도록 한다. 이를 위해 네트워크 파일 시스템에서 타임스탬프를 올바로 처리하기 위해 데이터 전송을 수행해야 할 수도 있다.

`AT_STATX_DONT_SYNC`
:   아무것도 동기화하지 말고 가능하면 시스템에 캐시된 걸 써먹는다. 반환되는 정보가 근사치라는 의미일 수도 있지만 네트워크 파일 시스템에서는 (잡은 리스가 없는 경우에도) 서버로의 왕복이 필요치 않을 수도 있다.

`statx()`의 `mask` 인자를 이용해 호출자가 관심 있는 필드가 뭔지 커널에게 알려 줄 수 있다. `mask`는 다음 상수들을 OR로 조합한 것이다.

| | |
| --- | --- |
| `STATX_TYPE`        | `stx_mode & S_IFMT` 원함 |
| `STATX_MODE`        | `stx_mode & ~S_IFMT` 원함 |
| `STATX_NLINK`       | `stx_nlink` 원함 |
| `STATX_UID`         | `stx_uid` 원함 |
| `STATX_GID`         | `stx_gid` 원함 |
| `STATX_ATIME`       | `stx_atime` 원함 |
| `STATX_MTIME`       | `stx_mtime` 원함 |
| `STATX_CTIME`       | `stx_ctime` 원함 |
| `STATX_INO`         | `stx_ino` 원함 |
| `STATX_SIZE`        | `stx_size` 원함 |
| `STATX_BLOCKS`      | `stx_blocks` 원함 |
| `STATX_BASIC_STATS` | [위 필드들 모두] |
| `STATX_BTIME`       | `stx_btime` 원함 |
| `STATX_ALL`         | [현재 가용 필드들 모두] |

참고로 일반적으로 `mask`에 위와 다른 값이 있어도 커널이 거부하지 *않는다*. (예외는 오류 설명의 `EINVAL`을 보라.) 대신 `statx.stx_mask` 필드를 통해 커널과 파일 시스템에서 지원하는 값들을 호출자에게 알려준다. 따라서 `mask`를 그냥 `UINT_MAX`로 설정(모든 비트 설정)해선 *안 된다*. 어떤 비트가 향후에 버퍼에 대한 확장을 나타내는 데 쓰일 수도 있기 때문이다.

### 반환되는 정보

`statxbuf`가 가리키는 `statx` 구조체로 대상 파일의 상태 정보가 반환된다. 그 구조체에 `stx_mask`가 있어서 어떤 정보들이 반환되는지 나타낸다. `stx_mask`의 형식은 `mask` 인자와 같으며 거기 설정된 비트들이 어떤 필드가 채워졌는지를 나타낸다.

기반 파일 시스템의 지원에 따라서 요청하지 않은 필드를 커널이 반환할 수도 있고 요청한 필드를 반환하지 못할 수도 있음에 유의해야 한다. (질의하지 않았는데 값을 받은 필드는 그냥 무시하면 된다.) 어느 경우든 `stx_mask`가 `mask`와 다르게 된다.

파일 시스템에서 어떤 필드를 지원하지 않거나 그 값이 표현 불가능한 (예를 들어 특이한 타입의 파일) 경우에는 사용자가 요청했더라도 `stx_mask`에서 그 필드에 대응하는 마스크 비트가 비워지게 되며 가능한 경우에는 호환을 위해 더미 값을 채운다. (가령 어떤 경우에 마운트를 위해 더미 UID 및 GID를 지정했을 수 있다.)

호출자가 요청하지 않았더라도 추가 비용 없이 값을 얻을 수 있는 경우에는 파일 시스템에서 필드를 채울 수도 있다. 이 경우 `stx_mask`에서 대응하는 비트가 설정된다.

*주의*: 성능과 단순성을 위해 `statx` 구조체의 필드들이 시스템 호출 실행 중의 상이한 시점의 상태 정보를 담을 수 있다. 예를 들어 다른 프로세스에서 <tt>[[chmod(2)]]</tt>나 <tt>[[chown(2)]]</tt>을 호출해서 `stx_mode`나 `stx_uid`를 바꾸는 경우에 `statx()`가 이전 `stx_mode`와 새 `stx_uid`를 반환할 수도 있고 이전 `stx_uid`와 새 `stx_mode`를 반환할 수도 있다.

(앞서 설명한) `stx_mask`를 빼고 `statx` 구조체의 필드들은 다음과 같다.

`stx_blksize`
:   효율적인 파일 시스템 I/O를 위한 "선호" 블록 크기.

`stx_attributes`
:   파일에 대한 추가 상태 정보. (자세한 내용은 아래 참고.)

`stx_nlink`
:   파일에 대한 하드 링크 수.

`stx_uid`
:   이 필드는 파일 소유자의 사용자 ID를 담는다.

`stx_gid`
:   이 필드는 파일 그룹 소유자의 ID를 담는다.

`stx_mode`
:   파일 종류와 모드. 자세한 내용은 <tt>[[inode(7)]]</tt> 참고.

`stx_ino`
:   파일의 아이노드 번호.

`stx_size`
:   (정규 파일이나 심볼릭 링크인 경우) 바이트 단위 파일 크기. 심볼릭 링크의 크기란 담고 있는 (종료 널 바이트 없는) 경로명의 길이다.

`stx_blocks`
:   매체 상에서 파일에 할당된 512바이트 단위 블록 수. (파일에 구멍이 있을 때는 `stx_size`/512보다 작을 수도 있다.)

`stx_attributes_mask`
:   `stx_attributes`의 어떤 비트들을 VFS와 파일 시스템에서 지원하는지 나타내는 마스크.

`stx_atime`
:   파일의 최근 접근 타임스탬프.

`stx_btime`
:   파일의 생성 타임스탬프.

`stx_ctime`
:   파일의 최근 상태 변경 타임스탬프.

`stx_mtime`
:   파일의 최근 수정 타임스탬프.

`stx_dev_major` 및 `stx_dev_minor`
:   이 파일이 (아이노드가) 위치한 장치.

`stx_rdev_major` 및 `stx_rdev_minor`
:   파일 종류가 블록 장치나 문자 장치인 경우 이 파일이 (아이노드가) 나타내는 장치.

위 필드들에 대한 더 자세한 내용은 <tt>[[inode(7)]]</tt>를 보라.

### 파일 속성

`stx_attributes` 필드는 파일의 추가 속성을 나타내는 플래그들의 OR 집합을 담는다. 참고로 `stx_attributes_mask`에 지원되는 걸로 표시돼 있지 않은 속성은 여기 플래그를 사용할 수 없다. `stx_attributes_mask`의 비트들은 `stx_attributes`와 비트 대 비트로 대응한다.

플래그들은 다음과 같다.

`STATX_ATTR_COMPRESSED`
:   파일 시스템에서 파일을 압축했으며 접근에 자원이 추가로 필요할 수 있다.

`STATX_ATTR_IMMUTABLE`
:   파일을 수정할 수 없다. 즉 삭제하거나 이름을 바꿀 수 없고, 이 파일에 대한 하드 링크를 만들 수 없고, 파일에 데이터를 기록할 수 없다. <tt>[[chattr(1)]]</tt> 참고.

`STATX_ATTR_APPEND`
:   파일을 덧붙이기 모드로 쓰기용으로만 열 수 있다. 임의 접근 쓰기가 허용되지 않는다. <tt>[[chattr(1)]]</tt> 참고.

`STATX_ATTR_NODUMP`
:   `dump(8)` 같은 백업 프로그램이 돌 때 파일이 백업 대상이 아니다. <tt>[[chattr(1)]]</tt> 참고.

`STATX_ATTR_ENCRYPTED`
:   파일 시스템에서 파일을 암호화하기 위해선 키가 필요하다.

`STATX_ATTR_VERITY` (리눅스 5.5부터)
:   파일에 fs-verity가 켜져 있다. 쓰기를 할 수 없으며 모든 읽기를 전체 파일에 대한 (가령 머클 트리를 통한) 암호학적 해시에 대해 검증하게 된다.

`STATX_ATTR_DAX` (리눅스 5.8부터)
:   파일이 DAX(CPU 직접 접근) 상태이다. DAX 상태에선 파일의 I/O와 메모리 매핑 모두에서 소프트웨어 캐시 효과를 최소하려고 한다. DAX를 지원하게 구성된 파일 시스템이 필요하다.

    DAX에선 일반적으로 모든 접근이 CPU 적재 / 저장 인스트럭션을 통해 이뤄진다고 상정하는데, 이는 소규모 접근에서 오버헤드를 최소화할 수 있지만 대규모 전송에서는 CPU 이용률에 부정적 영향을 줄 수 있다.

    사용자 공간 버퍼와 직접 파일 I/O가 이뤄지며 커널 페이지 캐시를 우회하는 직접 메모리 매핑으로 메모리 맵 I/O가 수행될 수도 있다.

    DAX 특성으로 인해 보통 데이터가 동기적으로 전송되긴 하지만, 데이터와 필요한 메타데이터가 함께 전송되는 `O_SYNC` 플래그(<tt>[[open(2)]]</tt> 참고)와 같은 보장까지 해 주진 않는다.

    DAX 파일을 `MAP_SYNC` 플래그로 맵 하는 게 지원될 수도 있는데, 그러면 프로그램에서 명시적 <tt>[[fsync(2)]]</tt> 없이 CPU 캐시 플러시 인스트럭션을 써서 CPU 저장 동작들을 영속화하는 게 가능해진다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   `pathname`의 경로 선두부의 한 디렉터리에 대해 탐색 권한이 거부되었다. (<tt>[[path_resolution(7)]]</tt>도 참고.)

`EBADF`
:   `dirfd`가 유효한 열린 파일 디스크립터가 아니다.

`EFAULT`
:   `pathname`이나 `statxbuf`가 NULL이거나 프로세스의 접근 가능 주소 공간 밖의 위치를 가리키고 있다.

`EINVAL`
:   `flags`에 유효하지 않은 플래그를 지정했다.

`EINVAL`
:   `mask`에 예비 플래그를 지정했다. (현재 그런 플래그가 하나 있는데, 상수 `STATX__RESERVED`이고 값은 0x80000000U이다.)

`ELOOP`
:   경로명을 순회하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   `pathname`이 너무 길다.

`ENOENT`
:   `pathname`의 어느 요소가 존재하지 않거나, `pathname`이 빈 문자열인데 `flags`에 `AT_EMPTY_PATH`를 지정하지 않았다.

`ENOMEM`
:   메모리 (즉 커널 메모리) 부족.

`ENOTDIR`
:   `pathname`의 경로 선두부의 어느 요소가 디렉터리가 아니거나, `pathname`이 상대 경로인데 `dirfd`가 디렉터리 아닌 파일을 가리키는 파일 디스크립터이다.

## VERSIONS

리눅스 커널 4.11에서 `statx()`가 추가되었다. glibc 2.28에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`statx()`는 리눅스 전용이다.

## SEE ALSO

`ls(1)`, `stat(1)`, <tt>[[access(2)]]</tt>, <tt>[[chmod(2)]]</tt>, <tt>[[chown(2)]]</tt>, <tt>[[readlink(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[utime(2)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[inode(7)]]</tt>, <tt>[[symlink(7)]]</tt>

----

2021-03-22
