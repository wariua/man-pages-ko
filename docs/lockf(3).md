## NAME

lockf - 열린 파일에 POSIX 락 적용하고 검사하고 제거하기

## SYNOPSIS

```c
#include <unistd.h>

int lockf(int fd, int cmd, off_t len);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`lockf()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.19부터 */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19 */ _BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

열린 파일의 어느 구간에 대해 POSIX 락을 적용하거나 확인하거나 제거한다. 쓰기용으로 연 파일 디스크립터 `fd`로 파일을 지정한다. `cmd`로 동작을 지정한다. 현재 파일 위치를 `pos`라고 할 때, `len`이 양수면 바이트 위치 `pos`..`pos+len-1`이 구간이 되고 `len`이 음수면 `pos-len`..`pos-1`이, `len`이 0이면 현재 파일 위치에서 현재와 미래의 파일 끝 위치를 아우르는 무한대까지가 구간이 된다.

리눅스에서 `lockf()`는 <tt>[[fcntl(2)]]</tt> 락킹에 씌운 인터페이스일 뿐이다. 다른 여러 시스템에서도 이 방식으로 `lockf()`를 구현한다. 하지만 POSIX.1에서는 `lockf()`와 <tt>[[fcntl(2)]]</tt> 락 사이의 관계를 명세하지 않고 있다. 이식성이 있어야 하는 프로그램에서는 두 인터페이스를 혼용하는 걸 피하는 게 좋을 것이다.

유효한 동작들은 다음과 같다.

`F_LOCK`
:   파일의 지정한 구간에 배타형 락을 설정한다. 그 구간이 일부라도 이미 잠겨 있으면 앞선 락이 해제될 때까지 호출이 블록한다. 그 구간이 이미 잠근 구간과 겹치면 두 구간이 병합된다. 락을 잡고 있는 프로세스가 그 파일에 대한 어떤 파일 디스크립터를 닫자마자 파일 락이 해제된다. 자식 프로세스가 락을 물려받지 않는다.

`F_TLOCK`
:   `F_LOCK`과 같되 파일이 이미 잠겨 있는 경우에 블록하지 않고 오류를 반환한다.

`F_ULOCK`
:   파일의 지정한 구간을 푼다. 이로 인해 어떤 잠긴 구간이 두 개의 구간으로 바뀔 수도 있다.

`F_TEST`
:   락을 확인한다. 지정한 구간이 잠겨 있지 않거나 현재 프로세스에 의해 잠겨 있는 경우에는 0을 반환한다. 다른 프로세스가 락을 잡고 있으면 -1을 반환하고 `errno`를 `EAGAIN`으로 (다른 일부 시스템에선 `EACCES`로) 설정한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES` 또는 `EAGAIN`
:   파일이 잠겨 있는데 `F_TLOCK`이나 `F_TEST`를 지정했다. 또는 파일을 다른 프로세스에서 메모리 매핑했기 때문에 동작이 금지된다.

`EBADF`
:   `fd`가 열린 파일 디스크립터가 아니다. 또는 `cmd`가 `F_LOCK`이나 `F_TLOCK`이고 `fd`가 쓰기 가능한 파일 디스크립터가 아니다.

`EDEADLK`
:   명령이 `F_LOCK`이었고 그 락 동작이 교착을 일으키려 했다.

`EINTR`
:   락 획득을 기다리는 동안 핸들러에 잡힌 시그널 전달에 의해 호출이 중단됐다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `cmd`에 유효하지 않은 동작을 지정했다.

`ENOLCK`
:   구간 락을 너무 많이 열어서 락 테이블이 가득 찼다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `lockf()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4.

## SEE ALSO

<tt>[[fcntl(2)]]</tt>, <tt>[[flock(2)]]</tt>

리눅스 커널 소스 디렉터리 `Documentation/filesystems`의 `locks.txt`와 `mandatory-locking.txt` (옛날 커널에선 이 파일들이 `Documentation` 디렉터리에 있으며 `mandatory-locking.txt`가 `mandatory.txt`다.)

----

2021-03-22
