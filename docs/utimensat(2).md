## NAME

utimensat, futimens - 나노초 정밀도로 파일 타임스탬프 바꾸기

## SYNOPSIS

```c
#include <fcntl.h> /* AT_* 상수 정의 */
#include <sys/stat.h>

int utimensat(int dirfd, const char *pathname,
              const struct timespec times[2], int flags);
int futimens(int fd, const struct timespec times[2]);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`utimensat()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_ATFILE_SOURCE`

`futimens()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

`utimensat()`과 `futimens()`는 나노초 정밀도로 파일의 타임스탬프를 갱신한다. 이와 대비되는 것이 과거의 <tt>[[utime(2)]]</tt>과 <tt>[[utimes(2)]]</tt>인데, 파일 타임스탬프를 설정할 때 각각 초 단위 정밀도와 마이크로초 정밀도만 허용한다.

`utimensat()`에서는 `pathname`으로 주는 경로명을 통해 파일을 지정한다. `futimens()`에서는 열려 있는 파일 디스크립터 `fd`를 통해 타임스탬프를 갱신할 파일을 지정한다.

두 호출 모두에서 배열 `times`로 새 파일 타임스탬프를 지정한다. `times[0]`은 새로운 "최근 접근 시간"(`atime`)을 나타내고 `times[1]`은 새로운 "최근 수정 시간"(`mtime`)을 나타내다. `times`의 각 항목은 에포크, 즉 1970-01-01 00:00:00 +0000 (UTC) 후로 지난 초와 나노초 수로 시간을 나타내다. 이 정보를 다음 형식의 구조체로 전달한다.

```c
struct timespec {
    time_t tv_sec;        /* 초 */
    long   tv_nsec;       /* 나노초 */
};
```

갱신되는 파일 타임스탬프는 파일 시스템에서 지원하는 지정한 시간보다 크지 않은 가장 큰 값으로 설정된다.

`timespec` 구조체의 `tv_nsec` 필드가 특수한 값 `UTIME_NOW`이면 대응하는 파일 타임스탬프를 현재 시간으로 설정한다. `timespec` 구조체의 `tv_nsec` 필드가 특수한 값 `UTIME_OMIT`이면 대응하는 파일 타임스탬프를 바꾸지 않고 놔둔다. 두 경우 모두에서 대응하는 `tv_sec` 필드의 값은 무시한다.

`times`가 NULL이면 두 타임스탬프 모두를 현재 시간으로 설정한다.

### 권한 요건

두 파일 타임스탬프 모두를 현재 시간으로 설정하려면 (즉 `times`가 NULL이거나 두 `tv_nsec` 필드 모두 `UTIME_NOW` 지정) 다음 중 하나여야 한다.

1. 호출자가 그 파일에 쓰기 접근권을 가지고 있어야 한다.
2. 호출자의 실효 사용자 ID가 파일 소유자와 일치해야 한다.
3. 호출자가 적절한 특권을 가지고 있어야 한다.

두 타임스탬프 모두를 현재 시간으로 설정하는 것 외의 어떤 변경을 하려면 (즉 `times`가 NULL이 아니고 `tv_nsec` 필드 하나라도 `UTIME_NOW`이나 `UTIME_OMIT`가 아님) 위의 조건 2번과 3번 중 하나가 해당되어야 한다.

두 `tv_nsec` 필드가 모두 `UTIME_OMIT`으로 지정되어 있으면 파일 소유권 검사나 권한 검사를 수행하지 않으며 파일 타임스탬프가 변경되지 않는다. 하지만 그 경우에도 다른 오류 조건들을 탐지할 수 있다.

### `utimensat()` 한정 사항

`pathname`이 상대적인 경우에는 기본적으로 (<tt>[[utimes(2)]]</tt>에서 하듯 호출 프로세스의 현재 작업 디렉터리 기준이 아니라) 열린 파일 디스크립터 `dirfd`가 가리키는 디렉터리를 기준으로 해석한다. 이것이 유용할 수 있는 이유에 대한 설명은 <tt>[[openat(2)]]</tt>을 보라.

`pathname`이 상대적이고 `dirfd`가 특수한 값 `AT_FDCWD`인 경우에는 (<tt>[[utimes(2)]]</tt>처럼) 호출 프로세스의 현재 작업 디렉터리를 기준으로 `pathname`을 해석한다.

`pathname`이 절대적인 경우에는 `dirfd`를 무시한다.

`flags` 필드는 비트 마스크이다. 0일 수도 있고 `<fcntl.h>`에 정의된 다음 상수를 포함할 수도 있다.

`AT_SYMLINK_NOFOLLOW`
:   `pathname`이 심볼릭 링크를 나타내는 경우에 가리키는 파일이 아니라 그 링크의 타임스탬프를 갱신한다.

## RETURN VALUE

성공 시 `utimensat()` 및 `futimens()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   `times`가 NULL이거나 두 `tv_nsec` 값이 모두 `UTIME_NOW`이며, 호출자의 실효 사용자 ID가 파일 소유자와 일치하지 않고 호출자가 파일에 쓰기 접근권을 가지고 있지 않으며 호출자에게 특권이 없다 (리눅스: `CAP_FOWNER` 역능이나 `CAP_DAC_OVERRIDE` 역능을 가지고 있지 않다).

`EBADF`
:   (`futimens()`) `fd`가 유효한 파일 디스크립터가 아니다.

`EBADF`
:   (`utimensat()`) `pathname`이 상대 경로명인데 `dirfd`가 `AT_FDCWD`도 아니고 유효한 파일 디스크립터도 아니다.

`EFAULT`
:   `times`가 유효하지 않은 주소를 가리킨다. 또는 `dirfd`가 `AT_FDCWD`였는데 `pathname`이 NULL이거나 유효하지 않은 주소이다.

`EINVAL`
:   `flags`에 유효하지 않은 값.

`EINVAL`
:   한 `tv_nsec` 필드에 유효하지 않은 값 (0에서 999,999,999까지 범위 밖의 값이고 `UTIME_NOW`나 `UTIME_OMIT`이 아님). 또는 한 `tv_sec` 필드에 유효하지 않은 값.

`EINVAL`
:   `pathname`이 NULL이고, `dirfd`가 `AT_FDCWD`가 아니고, `flags`가 `AT_SYMLINK_NOFOLLOW`를 담고 있다.

`ELOOP`
:   (`utimensat()`) `pathname`을 해석하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   (`utimensat()`) `pathname`이 너무 길다.

`ENOENT`
:   (`utimensat()`) `pathname`의 어느 요소가 존재하는 디렉터리나 파일을 가리키지 않거나, `pathname`이 빈 문자열이다.

`ENOTDIR`
:   (`utimensat()`) `pathname`이 상대 경로인데 `dirfd`가 `AT_FDCWD`도 아니고 디렉터리를 가리키는 파일 디스크립터도 아니다. 또는 `pathname`의 한 선두 요소가 디렉터리가 아니다.

`EPERM`
:   호출자가 타임스탬프들 중 하나 또는 모두를 현재 시간 아닌 값으로 바꾸려 했거나, 타임스탬프 하나를 현재 시간으로 바꾸고 나머지 타임스탬프는 그대로 두려 했으며, 다음 중 하나이다.

    * 호출자의 실효 사용자 ID가 파일 소유자와 일치하지 않으며, 호출자에게 특권이 없다 (리눅스: `CAP_FOWNER` 역능을 가지고 있지 않다).

    * 파일이 덧붙임 전용이나 불변으로 표시되어 있다. (<tt>[[chattr(1)]]</tt> 참고)

`EROFS`
:   파일이 읽기 전용 파일 시스템 상에 있다.

`ESRCH`
:   (`utimensat()`) `pathname`의 한 선두 요소에 대해 탐색 권한이 거부되었다.

## VERSIONS

리눅스 커널 2.6.22에서 `utimensat()`이 추가되었다. glibc 버전 2.6에서 지원이 추가되었다.

glibc 2.6에서 `futimens()` 지원이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `utimensat()`, `futimens()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`futimens()`와 `utimensat()`은 POSIX.1-2008에 명세되어 있다.

## NOTES

`utimensat()`은 <tt>[[futimesat(2)]]</tt>을 구식화한다.

리눅스에서는 불변(immutable)으로 표시된 파일의 타임스탬프를 바꿀 수 없으며 덧붙임 전용(append-only)으로 표시된 파일에 유일하게 허용되는 변경은 타임스탬프를 현재 시간으로 설정하는 것이다. (이는 리눅스에서 <tt>[[utime(2)]]</tt>과 <tt>[[utimes(2)]]</tt>의 역사적 동작 방식과 일치한다.)

두 `tv_nsec` 필드가 모두 `UTIME_OMIT`으로 지정되어 있는 경우 리눅스의 `utimensat()` 구현은 `dirfd`와 `pathname`이 가리키는 파일이 존재하지 않아도 성공한다.

### C 라이브러리/커널 ABI 차이

리눅스에서 `futimens()`는 `utimensat()` 시스템 호출 위에서 구현한 라이브러리 함수이다. 이를 지원하기 위해 리눅스의 `utimensat()` 시스템 호출에서는 비표준 기능을 한 가지 구현하고 있다. `pathname`이 NULL이면 호출에서 (어떤 종류의 파일도 가리킬 수 있는) 파일 디스크립터 `dirfd`가 가리키는 파일의 타임스탬프를 수정한다. 이 기능을 이용하여 `futimens(fd, times)` 호출을 다음과 같이 구현한다.

```c
utimensat(fd, NULL, times, 0);
```

참고로 glibc의 `utimensat()`에서는 `pathname` 값으로 NULL을 주는 것을 허용하지 않는다. 이 경우 래퍼 함수가 오류 `EINVAL`을 반환한다.

## BUGS

2.6.26 전의 커널들에는 `utimensat()`과 `futimens()`에 여러 버그들이 있다. 이 버그들은 POSIX.1 초안 명세와의 불일치이거나 리눅스의 역사적 동작 방식과의 불일치이다.

* POSIX.1에서는 한 `tv_nsec` 필드의 값이 `UTIME_NOW`나 `UTIME_OMIT`이면 대응하는 `tv_sec` 필드를 무시해야 한다고 명세한다. 그런데 `tv_sec` 필드의 값이 0이기를 요구한다. (아니면 `EINVAL` 오류 발생.)

* 다양한 버그들이 의미하는 바는 권한 검사에 있어서 두 `tv_nsec` 필드가 모두 `UTIME_NOW`로 설정된 경우를 항상 `times`를 NULL로 지정한 것과 같게 다루지는 않는다는 것, 그리고 한 `tv_nsec` 값이 `UTIME_NOW`이고 다른 값이 `UTIME_OMIT`인 경우를 `times`를 임의 시간 값들을 담은 구조체 배열에 대한 포인터로 지정한 것과 같게 다루지 않는다는 것이다. 버그들 때문에 일부 경우에서 a) 갱신을 수행할 권한이 없어야 하는 프로세스가 파일 타임스탬프를 갱신할 수 있고, b) 갱신을 수행할 권한이 있어야 하는 프로세스가 파일 타임스탬프를 갱신할 수 없고, c) 오류 경우에 틀린 `errno` 값을 반환한다.

* POSIX.1에서는 두 가지 타임스탬프를 모두 현재 시간으로 갱신하기 위해 *파일에 대한 쓰기 접근권*을 가진 프로세스가 `times`를 NULL로 해서, 또는 `times`가 두 `tv_nsec` 필드가 모두 `UTIME_NOW`인 구조체 배열을 가리키게 해서 호출을 할 수 있다고 한다. 하지만 `futimens()`에서 그 대신 *파일 디스크립터의 접근 모드에서 쓰기를 허용하는지* 여부를 확인한다.

## SEE ALSO

<tt>[[chattr(1)]]</tt>, `touch(1)`, <tt>[[futimesat(2)]]</tt>, <tt>[[openat(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[utimes(2)]]</tt>, `futimes(3)`, <tt>[[inode(7)]]</tt>, <tt>[[path_resolution(7)]]</tt>, <tt>[[symlink(7)]]</tt>

----

2021-03-22
