## NAME

acct - 프로세스 통계 켜고 끄기

## SYNOPSIS

```c
#include <unistd.h>

int acct(const char *filename);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`acct()`:
:   glibc 2.21부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 2.20:
    :   `_DEFAULT_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

    glibc 2.19까지:
    :   `_BSD_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

## DESCRIPTION

`acct()` 시스템 호출은 프로세스 통계 수집을 켜거나 끈다. 존재하는 파일의 이름을 인자로 해서 호출하면 통계 수집이 켜진다. 그러면 종료하는 프로세스 각각의 레코드가 종료 시점에 `filename` 파일에 덧붙는다. NULL 인자는 통계 수집이 꺼지게 한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   지정한 파일에 대해 쓰기 권한이 거부되었거나, `filename`의 경로 선두부의 한 디렉터리에 대해 탐색 권한이 거부되었거나 (<tt>[[path_resolution(7)]]</tt> 참고), `filename`이 정규 파일이 아니다.

`EFAULT`
:   `filename`이 접근 가능한 주소 공간 밖을 가리키고 있다.

`EIO`
:   파일 `filename`에 쓰는 중 오류.

`EISDIR`
:   `filename`이 디렉터리다.

`ELOOP`
:   `filename`을 해석하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   `filename`이 너무 길다.

`ENFILE`
:   열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.

`ENOENT`
:   지정한 파일이 존재하지 않는다.

`ENOMEM`
:   메모리 부족.

`ENOSYS`
:   운영 체제 커널을 컴파일할 때 BSD 프로세스 통계 기능을 켜지 않았다. 이 기능을 제어하는 커널 구성 매개변수는 `CONFIG_BSD_PROCESS_ACCT`이다.

`ENOTDIR`
:   `filename`에서 디렉터리인 부분이 실제로는 디렉터리가 아니다.

`EPERM`
:   호출 프로세스가 프로세스 통계를 켜기에 충분한 특권을 가지고 있지 않다. 리눅스에서는 `CAP_SYS_PACCT` 역능이 필요하다.

`EROFS`
:   `filename`이 읽기 전용 파일 시스템의 파일을 가리키고 있다.

`EUSERS`
:   유휴 파일 구조가 더는 없거나 메모리를 다 썼다.

## CONFORMING TO

SVr4, 4.3BSD (POSIX는 아님).

## NOTES

시스템 크래시 발생 시 동작 중인 프로그램들에 대해선 통계 정보가 생성되지 않는다. 특히 종료하지 않는 프로세스에 대한 통계는 절대 기록되지 않는다.

통계 파일에 기록되는 레코드의 구조를 <tt>[[acct(5)]]</tt>에서 기술한다.

## SEE ALSO

<tt>[[acct(5)]]</tt>

----

2021-03-22
