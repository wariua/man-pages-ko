## NAME

process_vm_readv, process_vm_writev - 프로세스 주소 공간들 간에 데이터 전송하기

## SYNOPSIS

```c
#include <sys/uio.h>

ssize_t process_vm_readv(pid_t pid,
                       const struct iovec *local_iov,
                       unsigned long liovcnt,
                       const struct iovec *remote_iov,
                       unsigned long riovcnt,
                       unsigned long flags);
ssize_t process_vm_writev(pid_t pid,
                       const struct iovec *local_iov,
                       unsigned long liovcnt,
                       const struct iovec *remote_iov,
                       unsigned long riovcnt,
                       unsigned long flags);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`process_vm_readv()`, `process_vm_writev()`:
:   `_GNU_SOURCE`

## DESCRIPTION

이 시스템 호출들은 호출 프로세스("지역 프로세스")와 `pid`로 나타낸 프로세스("원격 프로세스")의 주소 공간들 사이에서 데이터를 전송한다. 데이터가 커널을 거치지 않고 두 프로세스 주소 공간 간에 직접 이동한다.

`process_vm_readv()` 시스템 호출은 원격 프로세스에서 지역 프로세스로 데이터를 전송한다. 전송할 데이터를 `remote_iov`와 `riovcnt`로 나타낸다. `remote_iov`는 프로세스 `pid` 내 주소 범위를 기술하는 배열의 포인터이고 `riovcnt`는 `remote_iov`의 항목 개수를 나타낸다. 그 데이터가 `local_iov`와 `liovcnt`로 나타낸 위치로 전송된다. `local_iov`는 호출 프로세스 내 주소 범위를 기술하는 배열의 포인터이고 `liovcnt`는 `local_iov`의 항목 개수를 나타낸다.

`process_vm_writev()` 시스템 호출은 `process_vm_readv()`의 반대이다. 즉, 지역 프로세스에서 원격 프로세스로 데이터를 전송한다. 전송 방향을 제외하고 인자 `liovcnt`, `local_iov`, `riovcnt`, `remote_iov`의 의미는 `process_vm_readv()`에서와 같다.

`local_iov`와 `remote_iov` 인자는 `iovec` 구조체 배열을 가리킨다. 이 구조체는 `<sys/uio.h>`에 다음과 같이 정의되어 있다.

```c
struct iovec {
    void  *iov_base;    /* 시작 주소 */
    size_t iov_len;     /* 옮길 바이트 수 */
};
```

배열 순서대로 버퍼들을 처리한다. 즉 `process_vm_readv()`에서 `local_iov[0]`를 완전히 채운 다음 `local_iov[1]`로 진행하는 식이다. 또한 `remote_iov[0]`를 완전히 읽은 다음 `remote_iov[1]`로 진행한다.

마찬가지로 `process_vm_writev()`에서는 `local_iov[0]`의 내용을 모두 쓴 다음 `local_iov[1]`로 진행하고 `remote_iov[0]`를 완전히 채운 다음 `remote_iov[1]`로 진행한다.

`remote_iov[i].iov_len`과 `local_iov[i].iov_len`의 길이 값이 같을 필요가 없다. 즉, 지역 버퍼 한 개를 원격 버퍼 여러 개로 쪼개거나 그 반대로 하는 것이 가능하다.

`flags` 인자는 현재 사용하지 않으며 0으로 설정해야 한다.

`liovcnt` 및 `riovcnt` 인자에 지정한 값은 `IOV_MAX` 이하여야 한다. (`IOV_MAX`는 `<limits.h>`에 정의되어 있거나 `sysconf(_SC_IOV_MAX)` 호출로 얻을 수 있다.)

전송을 하기 전에 개수 인자들과 `local_iov`를 확인한다. 개수가 너무 크거나, `local_iov`가 유효하지 않거나, 그 주소가 지역 프로세스에서 접근 불가능한 영역을 가리키는 경우 어떤 벡터도 처리하지 않고 즉시 오류를 반환한다.

하지만 이 시스템 호출들은 읽기/쓰기를 하기 직전까지 원격 프로세스의 메모리 영역을 확인하지 않는다. 따라서 `remote_iov`의 한 항목이 원격 프로세스에서 유효하지 않은 메모리 영역을 가리키는 경우 불완전 읽기/쓰기가 일어날 수도 있다. (RETURN VALUE 참고.) 그 지점 이후로는 읽기/쓰기를 더 시도하지 않는다. 길이를 모르는 (널 종료 C 문자열 같은) 데이터를 원격 프로세스로부터 읽으려 할 때 이를 고려하여 원격 `iovec` 항목 하나가 메모리 페이지(보통 4 KiB) 여러 개에 걸치지 않게 하는 게 좋다. (그 원격 읽기를 `remote_iov` 항목 두 개로 쪼개고 `local_iov` 한 개에 쓰는 것으로 합쳐지게 하면 된다. 첫 번째 읽기 항목이 페이지 경계까지 가고 두 번째가 다음 페이지 경계에서 시작하는 것이다.)

다른 프로세스에 읽거나 쓰는 권한은 ptrace 접근 모드 `PTRACE_MODE_ATTACH_REALCREDS` 검사로 결정된다. <tt>[[ptrace(2)]]</tt> 참고.

## RETURN VALUE

성공 시 `process_vm_readv()`는 읽은 바이트 수를 반환하고 `process_vm_writev()`는 쓴 바이트 수를 반환한다. 불완전 읽기/쓰기가 일어난 경우 반환 값이 요청한 바이트 총수보다 작을 수 있다. (불완전 전송은 `iovec` 항목 단위로 이뤄진다. 이 시스템 호출들은 `iovec` 항목 하나를 쪼개는 불완전 전송을 수행하지 않는다.) 호출자는 반환 값을 확인해서 불완전 읽기/쓰기가 일어났는지 여부를 알아보는 게 좋다.

오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `local_iov`가 나타내는 메모리가 호출자의 접근 가능 메모리 공간 밖에 있다.

`EFAULT`
:   `remote_iov`가 나타내는 메모리가 프로세스 `pid`의 접근 가능 메모리 공간 밖에 있다.

`EINVAL`
:   `local_iov`나 `remote_iov`의 `iov_len` 값들의 합이 `ssize_t` 값 범위를 넘어간다.

`EINVAL`
:   `flags`가 0이 아니다.

`EINVAL`
:   `liovcnt`나 `riovcnt`가 너무 크다.

`ENOMEM`
:   `iovec` 구조체의 내부 사본을 위한 메모리를 할당하지 못했다.

`EPERM`
:   호출자가 프로세스 `pid`의 주소 공간에 접근할 권한을 가지고 있지 않다.

`ESRCH`
:   ID가 `pid`인 프로세스가 존재하지 않는다.

## VERSIONS

리눅스 3.2에서 이 시스템 호출들이 추가되었다. glibc 버전 2.15부터 지원을 제공한다.

## CONFORMING TO

이 시스템 호출들은 비표준 리눅스 확장이다.

## NOTES

`process_vm_readv()`와 `process_vm_writev()`가 수행하는 데이터 전송은 어떤 식으로도 원자성이 보장되지 않는다.

이 시스템 호출들은 한 번의 복사 동작으로 메시지를 교환할 수 있게 하여 빠른 메시지 전달을 가능케 하도록 설계되었다. (예를 들어 공유 메모리나 파이프를 사용한다면 두 번의 복사가 필요할 것이다.)

## EXAMPLES

다음 코드 샘플이 `process_vm_readv()` 사용 방식을 보여 준다. PID가 10인 프로세스로부터 주소 0x10000에 있는 20바이트를 읽어서 처음 10바이트를 `buf1`에 쓰고 두 번째 10바이트를 `buf2`에 쓴다.

```c
#include <sys/uio.h>

int
main(void)
{
    struct iovec local[2];
    struct iovec remote[1];
    char buf1[10];
    char buf2[10];
    ssize_t nread;
    pid_t pid = 10;            /* 원격 프로세스의 PID */

    local[0].iov_base = buf1;
    local[0].iov_len = 10;
    local[1].iov_base = buf2;
    local[1].iov_len = 10;
    remote[0].iov_base = (void *) 0x10000;
    remote[0].iov_len = 20;

    nread = process_vm_readv(pid, local, 2, remote, 1, 0);
    if (nread != 20)
        return 1;
    else
        return 0;
}
```

## SEE ALSO

<tt>[[readv(2)]]</tt>, <tt>[[writev(2)]]</tt>

----

2021-03-22
