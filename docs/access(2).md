## NAME

access, faccessat, faccessat2 - 파일에 대한 사용자의 권한 확인하기

## SYNOPSIS

```c
#include <unistd.h>

int access(const char *pathname, int mode);

#include <fcntl.h>           /* AT_* 상수 정의 */
#include <unistd.h>

int faccessat(int dirfd, const char *pathname, int mode, int flags);
                /* 아래 C 라이브러리/커널 차이 참고 */

int faccessat2(int dirfd, const char *pathname, int mode, int flags);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`faccessat()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_ATFILE_SOURCE`

## DESCRIPTION

`access()`는 호출 프로세스가 파일 `pathname`에 접근할 수 있는지 여부를 확인한다. `pathname`이 심볼릭 링크이면 따라간다.

`mode`는 수행할 접근 가능 검사(들)을 나타내는데, `F_OK` 값이거나 `R_OK`, `W_OK`, `X_OK` 중 하나 이상을 비트 OR 해서 만든 마스크이다. `F_OK`는 파일 존재 여부를 검사한다. `R_OK`, `W_OK`, `X_OK`는 파일이 존재하며 각각 읽기, 쓰기, 실행 권한을 허가하는지 여부를 검사한다.

호출 프로세스의 *실제* UID 및 GID로 검사를 수행한다. 즉 파일에 대한 동작(가령 <tt>[[open(2)]]</tt>)을 실제 시도할 때처럼 실효 ID로 하는 게 아니다. 마찬가지로 루트 사용자에 대해 실효 역능 집합이 아니라 허용 역능 집합을 검사에 사용한다. 루트 아닌 사용자에 대해선 검사에 빈 역능 집합을 쓴다.

이 때문에 set-user-ID 프로그램과 역능을 부여받은 프로그램에서 자기를 호출한 사용자의 권한을 쉽게 판단할 수 있다. 달리 말하자면 `access()`는 "내가 이 파일을 읽을/쓸/실행할 수 있는가?"라는 질문에 답하지 않는다. 살짝 다른 질문, 즉 "(내가 setuid 바이너리라고 하고) *나를 호출한 사용자가* 이 파일을 읽을/쓸/실행할 수 있는가?"에 답한다. 그래서 set-user-ID 프로그램인 경우에, 악의적 사용자가 읽을 수 없어야 되는 파일을 읽게끔 만드는 걸 막을 수 있게 된다.

호출 프로세스에게 특권이 있는 (즉 실재 UID가 0인) 경우에는 정규 파일의 소유자, 그룹, 기타 중 어디에든 실행 권한이 켜져 있으면 그 파일에 대한 `X_OK` 검사가 성공한다.

### `faccessat()`

`faccessat()`은 여기 설명하는 차이점을 빼면 `access()`와 똑같이 동작한다.

`pathname`에 준 경로명이 상대 경로이면 (상대 경로명에 대해 `access()`에서 하듯 호출 프로세스의 현재 작업 디렉터리를 기준으로 하는 게 아니라) 파일 디스크립터 `dirfd`가 가리키는 디렉터리를 기준으로 경로명을 해석한다.

`pathname`이 상대 경로이고 `dirfd`가 특수 값 `AT_FDCWD`이면 (`access()`처럼) 호출 프로세스의 현재 작업 디렉터리를 기준으로 `pathname`을 해석한다.

`pathname`이 절대 경로이면 `dirfd`를 무시한다.

`flags`는 다음 값들을 0개 이상 OR 해서 구성한다.

`AT_EACCESS`
:   실효 사용자 및 그룹 ID로 접근 검사를 수행한다. 기본적으로 `faccessat()`에서는 (`access()`처럼) 실제 ID를 사용한다.

`AT_SYMLINK_NOFOLLOW`
:   `pathname`이 심볼릭 링크인 경우 따라가지 않는다. 대신 링크 자체에 대한 정보를 반환한다.

`faccessat()`의 필요성에 대한 설명은 <tt>[[openat(2)]]</tt>을 보라.

### `faccessat2()`

위의 `faccessat()` 설명은 POSIX.1 및 glibc 제공 구현체와 부합한다. 하지만 glibc의 구현은 리눅스의 `faccessat()` 시스템 호출에 `flags`가 없는 문제를 땜질하는 불완전한 에뮬레이션이었다. (BUGS 참고.) 올바른 구현이 가능하도록 리눅스 5.8에서 `faccessat2()` 시스템 호출이 추가되었는데, `flags` 인자를 지원하여 `faccessat()` 래퍼 함수를 올바르게 구현할 수 있게 되었다.

## RETURN VALUE

성공 시 (모든 요청 권한이 허가됨, 또는 `mode`가 `F_OK`이고 파일이 존재함) 0을 반환한다. 오류 시 (권한을 묻는 `mode`의 비트 중 최소 하나가 거부됨, 또는 `mode`가 `F_OK`이고 파일이 존재하지 않음, 또는 어떤 다른 오류 발생) -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

다음 경우에 `access()`와 `faccessat()`이 실패한다.

`EACCES`
:   요청한 접근이 파일에 대해서 거부될 것이다. 또는 `pathname`의 경로 선두부의 한 디렉터리에 대해 탐색 권한이 거부되었다. (<tt>[[path_resolution(7)]]</tt> 참고.)

`ELOOP`
:   `pathname`을 해석하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   `pathname`이 너무 길다.

`ENOENT`
:   `pathname`의 어느 요소가 존재하지 않거나 깨진 심볼릭 링크이다.

`ENOTDIR`
:   `pathname`에서 디렉터리로 쓰인 요소가 실제로는 디렉터리가 아니다.

`EROFS`
:   읽기 전용 파일 시스템 상의 파일에 쓰기 권한을 요청했다.

다음 경우에 `access()`와 `faccessat()`이 실패할 수도 있다.

`EFAULT`
:   `pathname`이 접근 가능한 주소 공간 밖을 가리킨다.

`EINVAL`
:   `mode`를 잘못 지정했다.

`EIO`
:   I/O 오류가 발생했다.

`ENOMEM`
:   사용 가능한 커널 메모리가 충분하지 않다.

`ETXTBSY`
:   실행 중인 실행 파일에 쓰기 접근을 요청했다.

`faccessat()`에서 추가로 다음 오류가 발생할 수 있다.

`EBADF`
:   `dirfd`가 유효한 파일 디스크립터가 아니다.

`EINVAL`
:   `flags`에 유효하지 않은 플래그를 지정했다.

`ENOTDIR`
:   `pathname`이 상대 경로이고 `dirfd`가 디렉터리 아닌 파일을 가리키는 파일 디스크립터이다.

## VERSIONS

리눅스 커널 2.6.16에서 `faccessat()`이 추가되었다. glibc 버전 2.4에서 라이브러리 지원이 추가되었다.

리눅스 버전 5.8에서 `faccessat2()`가 추가되었다.

## CONFORMING TO

`access()`: SVr4, 4.3BSD, POSIX.1-2001, POSIX.1-2008.

`faccessat()`: POSIX.1-2008.

`faccessat2()`: 리눅스 전용.

## NOTES

**경고**: 예를 들어 <tt>[[open(2)]]</tt>으로 실제 파일을 열기 전에 이 호출들을 사용해서 파일을 열 권한이 사용자에게 있는지 검사하는 방식은 보안상의 구멍을 만든다. 검사 시점과 파일을 열어서 조작하는 시점 사이의 짧은 간격을 사용자가 악용할 수도 있기 때문이다. **그런 이유로 이 시스템 호출 사용을 피하는 게 좋다.** (방금 서술한 예에서 안전한 대안은 프로세스의 실효 사용자 ID를 실제 ID로 잠시 전환하고서 <tt>[[open(2)]]</tt>을 호출하는 것이다.)

`access()`는 항상 심볼릭 링크를 따라간다. 심볼릭 링크에 대해 권한을 검사해야 하면 `faccessat()`를 `AT_SYMLINK_NOFOLLOW` 플래그로 사용하면 된다.

이 호출들은 `mode`의 접근 방식들 중 하나라도 거부되면 `mode`의 나머지 접근 방식 일부가 허용되는 경우라도 오류를 반환한다.

호출 프로세스에게 적절한 특권이 있는 경우에 (즉 수퍼유저인 경우) POSIX.1-2001에서는 실행 권한 비트가 전혀 설정돼 있지 않더라도 구현에서 `X_OK` 검사에 대해 성공을 표시하는 걸 허용한다. 리눅스에서는 그렇게 하지 않는다.

`pathname` 경로 선두부의 디렉터리 각각에 대해 권한이 탐색(즉 실행) 접근을 허가하는 경우에만 그 파일이 접근 가능하다. 한 디렉터리라도 접근 불가능이면 파일 자체에 대한 권한은 상관없이 `access()` 호출이 실패한다.

접근 비트만 확인할 뿐 파일 종류나 내용은 보지 않는다. 따라서 디렉터리가 쓰기 가능하다고 나온다면 그건 그 디렉터리에 파일을 만들 수 있다는 뜻이지 파일처럼 그 디렉터리에 뭔가를 기록할 수 있다는 뜻은 아닐 것이다. 마찬가지로 DOS 파일이 실행 가능이라고 나올 수 있지만 그래도 <tt>[[execve(2)]]</tt> 호출은 실패하게 된다.

이 호출들은 UID 매핑이 켜진 NFSv2 파일 시스템에서는 올바로 동작하지 않을 수도 있다. UID 매핑이 서버에서 이뤄지며 권한을 검사하는 클라이언트에게는 감춰져 있기 때문이다. (NFS 버전 3 및 이후에서는 서버에서 검사를 수행한다.) FUSE 마운트에도 비슷한 문제가 생길 수 있다.

### C 라이브러리/커널 차이

진짜 `faccessat()` 시스템 호출은 처음 세 인자만 받는다. `AT_EACCESS`와 `AT_SYMLINK_NOFOLLOW` 플래그는 사실 `faccessat()`의 glibc 래퍼 함수 안에 구현돼 있다. 그 플래그들 중 하나라도 지정한 경우에는 래퍼 함수에서 <tt>[[fstatat(2)]]</tt>을 이용해 접근 권한을 알아낸다. 하지만 BUGS를 보라.

### glibc 참고 사항

`faccessat()`이 없는 구식 커널에서는 (그리고 `AT_EACCESS`와 `AT_SYMLINK_NOFOLLOW` 플래그가 지정돼 있지 않을 때) glibc 래퍼 함수가 `access()`를 사용하는 걸로 후퇴한다. `pathname`이 상대 경로명일 때 glibc에서는 `/proc/self/fd` 안의 `dirfd` 인자에 대응하는 심볼릭 링크를 가지고 경로명을 만든다.

## BUGS

리눅스 커널의 `faccessat()` 시스템 호출에서 `flags` 인자를 지원하지 않기 때문에 glibc 2.32 및 이전에서 제공한 `faccessat()` 래퍼 함수에서는 `faccessat()` 시스템 호출과 <tt>[[fstatat(2)]]</tt>을 조합해서 필요한 기능을 흉내낸다. 하지만 그 에뮬레이션에서 ACL을 고려하지 않는다. glibc 2.33부터는 기반 커널에서 제공하는 경우 래퍼 함수에서 `faccessat2()` 시스템 호출을 활용하여 이 버그를 피한다.

커널 2.4(및 이전)에는 수퍼유저에 대한 `X_OK` 검사 처리에 좀 이상한 점이 있다. 디렉터리 아닌 파일에서 모든 부문의 실행 권한이 꺼져 있는 경우에 `access()` 검사가 -1을 반환하는 건 `mode`에 `X_OK`만 지정돼 있을 때이다. `mode`에 `R_OK`나 `W_OK`도 지정돼 있으면 그런 파일에 대해 `access()`가 0을 반환한다. 2.6 초기 (2.6.3까지의) 커널들도 커널 2.4와 같은 식으로 동작했다.

커널 2.6.20 전에서는 기반 파일 시스템을 <tt>[[mount(2)]]</tt> 할 때 `MS_NOEXEC` 플래그를 사용한 경우 이 호출들에서 그 플래그의 효과를 무시했다. 커널 2.6.20부터는 `MS_NOEXEC` 플래그를 존중한다.

## SEE ALSO

<tt>[[chmod(2)]]</tt>, <tt>[[chown(2)]]</tt>, <tt>[[open(2)]]</tt>, <tt>[[setgid(2)]]</tt>, <tt>[[setuid(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[euidaccess(3)]]</tt>, <tt>[[credentials(7)]]</tt>, <tt>[[path_resolution(7)]]</tt>, <tt>[[symlink(7)]]</tt>

----

2021-03-22
