## NAME

readv, writev, preadv, pwritev, preadv2, pwritev2 - 여러 버퍼로 데이터 읽거나 쓰기

## SYNOPSIS

```c
#include <sys/uio.h>

ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
ssize_t writev(int fd, const struct iovec *iov, int iovcnt);

ssize_t preadv(int fd, const struct iovec *iov, int iovcnt,
                off_t offset);
ssize_t pwritev(int fd, const struct iovec *iov, int iovcnt,
                off_t offset);

ssize_t preadv2(int fd, const struct iovec *iov, int iovcnt,
                off_t offset, int flags);
ssize_t pwritev2(int fd, const struct iovec *iov, int iovcnt,
                off_t offset, int flags);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`preadv()`, `pwritev()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE`

## DESCRIPTION

`readv()` 시스템 호출은 파일 디스크립터 `fd`에 연계된 파일에서 `iov`가 기술하는 버퍼들로 `iovcnt` 개 버퍼를 읽어 들인다 ("스캐터 입력").

`writev()` 시스템 호출은 `iov`가 기술하는 `iovcnt` 개 데이터 버퍼를 파일 디스크립터 `fd`에 연계된 파일로 기록한다 ("개더 출력").

포인터 `iov`는 `<sys/uio.h>`에 다음처럼 정의된 `iovec` 구조체의 배열을 가리킨다.

```c
struct iovec {
    void  *iov_base;    /* 시작 주소 */
    size_t iov_len;     /* 이동할 바이트 수 */
};
```

`readv()` 시스템 호출은 여러 버퍼를 채운다는 점을 빼면 <tt>[[read(2)]]</tt>와 마찬가지로 동작한다.

`writev()` 시스템 호출은 여러 버퍼를 써넣는다는 점을 빼면 <tt>[[write(2)]]</tt>와 마찬가지로 동작한다.

버퍼들은 배열 내 순서에 따라 처리된다. 즉 `readv()`에서 `iov[0]`를 완전히 채운 다음 `iov[1]`로 진행하는 식이다. (데이터가 충분히 없으면 `iov`가 가리키는 버퍼들이 모두 채워지지 않을 수도 있다.) 마찬가지로 `writev()`에서는 `iov[0]`의 내용 전체를 써넣은 다음 `iov[1]`로 진행하는 식이다.

`readv()`와 `writev()`가 수행하는 데이터 이동은 원자적이다. 즉 `writev()`가 써넣는 데이터는 한 덩어리로 기록되며 다른 프로세스의 쓰기 출력과 섞이지 않는다. (하지만 <tt>[[pipe(7)]]</tt>의 예외 참고.) 유사하게 `readv()`는 같은 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)을 가리키는 파일 디스크립터를 가진 다른 스레드 내지 프로세스에서의 읽기 동작과 상관없이 파일로부터 연속인 덩어리를 읽는다고 보장된다.

### `preadv()`와 `pwritev()`

`preadv()` 시스템 호출은 `readv()`와 <tt>[[pread(2)]]</tt>의 기능을 합친 것이다. `readv()`와 같은 일을 수행하되 네 번째 인자 `offset`이 추가돼서 입력 동작을 수행할 파일 오프셋을 지정할 수 있다.

`pwritev()` 시스템 호출은 `writev()`와 <tt>[[pwrite(2)]]</tt>의 기능을 합친 것이다. `writev()`와 같은 일을 수행하되 네 번째 인자 `offset`이 추가돼서 출력 동작을 수행할 파일 오프셋을 지정할 수 있다.

이 시스템 호출들은 파일 오프셋을 바꾸지 않는다. `fd`가 가리키는 파일에서 seek이 가능해야 한다.

### `preadv2()`와 `pwritev2()`

이 시스템 호출들은 `preadv()` 및 `pwritev()` 호출과 비슷하되 다섯 번째 인자 `flags`가 추가돼서 호출별로 동작 방식을 변경할 수 있다.

`preadv()` 및 `pwritev()`와 달리 `offset` 인자가 -1이면 현재 파일 오프셋을 사용하고 갱신한다.

`flags` 인자는 다음 플래그를 0개 이상 비트 OR 한 값을 담는다.

`RWF_DSYNC` (리눅스 4.7부터)
:   쓰기별로 지정할 수 있는 <tt>[[open(2)]]</tt> `O_DSYNC` 플래그의 등가물. 이 플래그는 `pwritev2()`에만 유효하며 시스템 호출로 기록하는 데이터 범위에만 적용된다.

`RWF_HIPRI` (리눅스 4.6부터)
:   우선도 높은 읽기/쓰기. 블록 기반 파일 시스템에서 장치 폴링을 할 수 있게 허용한다. 지연이 낮아지지만 자원을 더 쓸 수도 있다. (현재 이 기능은 `O_DIRECT` 플래그로 연 파일 디스크립터에서만 사용 가능하다.)

`RWF_SYNC` (리눅스 4.7부터)
:   쓰기별로 지정할 수 있는 <tt>[[open(2)]]</tt> `O_SYNC` 플래그의 등가물. 이 플래그는 `pwritev2()`에만 유효하며 시스템 호출로 기록하는 데이터 범위에만 적용된다.

`RWF_NOWAIT` (리눅스 4.14부터)
:   즉시 사용 가능하지 않은 데이터를 기다리지 않는다. 이 플래그를 지정하면 기반 저장소로부터 데이터를 읽어 와야 하거나 락을 기다려야 하는 경우에 `preadv2()` 시스템 호출이 즉시 반환하게 된다. 일부 데이터를 성공적으로 읽었으면 읽은 바이트 수를 반환한다. 읽은 바이트가 없으면 -1을 반환하고 `errno`를 `EAGAIN`으로 설정한다. 현재 이 플래그는 `preadv2()`에만 유효하다.

`RWF_APPEND` (리눅스 4.16부터)
:   쓰기별로 지정할 수 있는 <tt>[[open(2)]]</tt> `O_APPEND` 플래그의 등가물. 이 플래그는 `pwritev2()`에만 유효하며 시스템 호출로 기록하는 데이터 범위에만 적용된다. `offset` 인자가 쓰기 동작에 영향을 주지 않으며 항상 파일 끝에 데이터를 덧붙인다. 다만 `offset` 인자가 -1이면 현재 파일 오프셋을 갱신한다.

## RETURN VALUE

성공 시 `readv()`, `preadv()`, `preadv2()`는 읽어 들인 바이트 수를 반환한다. `writev()`, `pwritev()`, `pwritev2()`는 기록한 바이트 수를 반환한다.

성공한 호출에서 요청보다 적은 바이트를 이동한 것이 오류가 아니라는 점에 유의하라. (<tt>[[read(2)]]</tt> 및 <tt>[[write(2)]]</tt> 참고.)

오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

오류는 <tt>[[read(2)]]</tt> 및 <tt>[[write(2)]]</tt>에서와 같다. 더불어 `preadv()`, `preadv2()`, `pwritev()`, `pwritev2()`가 <tt>[[lseek(2)]]</tt>과 같은 이유로 실패할 수도 있다. 추가로 다음 오류들이 정의돼 있다.

`EINVAL`
:   `iov_len` 값들의 합이 `ssize_t` 값을 넘치게 한다.

`EINVAL`
:   벡터 카운트 `iovcnt`가 0보다 작거나 허용 최대치보다 크다.

`EOPNOTSUPP`
:   `flags`에 알 수 없는 플래그를 지정했다.

## VERSIONS

리눅스 2.6.30에서 `preadv()`와 `pwritev()`가 처음 등장했다. glibc 2.10에서 라이브러리 지원이 추가되었다.

리눅스 4.6에서 `preadv2()`와 `pwritev2()`가 처음 등장했다. glibc 2.26에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`readv()`, `writev()`: POSIX.1-2001, POSIX.1-2008, 4.4BSD (4.2BSD에서 이 시스템 호출들이 처음 등장).

`preadv()`, `pwritev()`: 비표준, 하지만 최신 BSD들에도 있음.

`preadv2()`, `pwritev2()`: 비표준 리눅스 확장.

## NOTES

POSIX.1에서는 `iov`에 전달할 수 있는 항목 개수에 대한 제한을 구현에서 둘 수 있게 허용한다. 구현에서 `<limits.h>`에 `IOV_MAX`를 정의하거나 런타임에 `sysconf(_SC_IOV_MAX)` 반환 값을 통해 그 제한값을 선전할 수 있다. 요즘 리눅스 시스템에서는 그 제한이 1024이다. 리눅스 2.0 시절에는 그 제한이 16이었다.

### C 라이브러리/커널 차이

진짜 `preadv()` 및 `pwritev()` 시스템 호출의 호출 시그너처는 SYNOPSIS에 있는 대응하는 GNU C 라이브러리 래퍼 함수와 살짝 다르다. 래퍼 함수의 마지막 인자 `offset`을 시스템 호출에서는 두 개 인자로 나눈다.

```c
unsigned long pos_l, unsigned long pos
```

이 인자들은 각각 `offset`의 하위 32비트와 상위 32비트를 담는다.

### 과거의 C 라이브러리/커널 차이

리눅스 초기 버전에서 `IOV_MAX`가 아주 낮았던 것에 대처하기 위해 glibc의 `readv()` 및 `writev()` 래퍼 함수에서는 기반 커널 시스템 호출이 그 제한 초과 때문에 실패했음을 감지한 경우 어떤 추가 작업을 해 주었다. `readv()`의 경우 래퍼 함수에서 `iov`의 항목들을 모두 담을 만큼 큰 임시 버퍼를 할당해서 그 버퍼로 <tt>[[read(2)]]</tt> 호출을 한 다음 버퍼의 데이터를 `iov` 항목들의 `iov_base` 필드에 지정된 위치로 복사하고서 버퍼를 해제했다. `writev()`의 래퍼 함수도 임시 버퍼와 <tt>[[write(2)]]</tt> 호출로 비슷한 작업을 수행했다.

리눅스 2.2 및 이후부터는 glibc 래퍼 함수에서 이런 추가 작업이 필요 없게 되었다. 하지만 glibc는 버전 2.10까지 이 동작을 계속 제공했다. glibc 버전 2.9부터는 시스템이 리눅스 커널 2.6.18(임의로 선정한 커널 버전)보다 오래 된 버전에서 동작 중임을 라이브러리에서 탐지한 경우에만 래퍼 함수가 이 동작을 제공한다. (최소로 요구하는 리눅스 커널 버전이 2.6.32인) glibc 2.20부터는 언제나 glibc 래퍼 함수에서 시스템 호출을 바로 부른다.

## EXAMPLES

다음 코드 샘플이 `writev()` 사용 방식을 보여 준다.

```c
char *str0 = "hello ";
char *str1 = "world\n";
struct iovec iov[2];
ssize_t nwritten;

iov[0].iov_base = str0;
iov[0].iov_len = strlen(str0);
iov[1].iov_base = str1;
iov[1].iov_len = strlen(str1);

nwritten = writev(STDOUT_FILENO, iov, 2);
```

## SEE ALSO

<tt>[[pread(2)]]</tt>, <tt>[[read(2)]]</tt>, <tt>[[write(2)]]</tt>

----

2018-04-30
