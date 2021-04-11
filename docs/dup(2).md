## NAME

dup, dup2, dup3 - 파일 디스크립터 복제하기

## SYNOPSIS

```c
#include <unistd.h>

int dup(int oldfd);
int dup2(int oldfd, int newfd);

#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <fcntl.h>              /* O_* 상수 정의 얻기 */
#include <unistd.h>

int dup3(int oldfd, int newfd, int flags);
```

## DESCRIPTION

`dup()` 시스템 호출은 파일 디스크립터 `oldfd`의 사본을 만든다. 안 쓰는 가장 낮은 파일 디스크립터 번호를 새 디스크립터에 쓴다.

성공 반환 후에는 이전 파일 디스크립터와 새 파일 디스크립터를 서로 바꿔 가며 사용할 수도 있다. 같은 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)을 가리키기 때문에 오프셋과 파일 상태를 공유한다. 예를 들어 한쪽 파일 디스크립터에서 <tt>[[lseek(2)]]</tt>으로 파일 오프셋을 변경하면 다른 쪽에서도 오프셋이 바뀐다.

두 파일 디스크립터가 파일 디스크립터 플래그('exec에서 닫기' 플래그)는 공유하지 않는다. 복제 디스크립터에서는 'exec에서 닫기'(`FD_CLOEXEC`, <tt>[[fcntl(2)]]</tt> 참고) 플래그가 꺼져 있다.

### `dup2()`

`dup2()` 시스템 호출은 `dup()`과 같은 일을 하되 안 쓰는 가장 낮은 파일 디스크립터 번호를 쓰는 게 아니라 `newfd`에 지정한 파일 디스크립터 번호를 쓴다. 파일 디스크립터 `newfd`가 이미 열려 있으면 조용히 닫고서 재사용한다.

파일 디스크립터 `newfd`를 닫고 재사용하는 단계를 *원자적으로* 수행한다. 이 점이 중요한 건 <tt>[[close(2)]]</tt>와 `dup()`을 이용해 동등한 기능성을 구현하려 하면 `newfd`가 두 단계 사이에서 재사용될 수도 있는 경쟁 조건에 직면하게 되기 때문이다. 메인 프로그램을 중단시킨 시그널 핸들러에서 파일 디스크립터를 할당하거나 병렬 스레드에서 파일 디스크립터를 할당하여 그런 재사용이 일어날 수 있을 것이다.

다음 사항에 주의해야 한다.

* `oldfd`가 유효한 파일 디스크립터가 아닌 경우 호출이 실패하며 `newfd`는 닫히지 않는다.

* `oldfd`가 유효한 파일 디스크립터이고 `newfd`가 `oldfd`와 값이 같은 경우 `dup2()`는 아무것도 하지 않고 `newfd`를 반환한다.

### `dup3()`

`dup3()`는 다음 사항을 제외하면 `dup2()`와 같다.

* 호출자가 `flags`에 `O_CLOEXEC`를 지정해서 새 파일 디스크립터에 'exec에서 닫기' 플래그가 설정되게 할 수 있다. 이게 유용할 수 있는 이유에 대해선 <tt>[[open(2)]]</tt>의 해당 플래그 설명을 보라.

* `oldfd`가 `newfd`와 같은 경우에 `dup3()`는 `EINVAL` 오류로 실패한다.

## RETURN VALUE

성공 시 이 시스템 호출들은 새 파일 디스크립터를 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`EBADF`
:   `oldfd`가 열린 파일 디스크립터가 아니다.

`EBADF`
:   `newfd`가 파일 디스크립터 허용 범위를 벗어난다. (<tt>[[getrlimit(2)]]</tt>의 `RLIMIT_NOFILE` 설명 참고.)

`EBUSY`
:   (리눅스 한정) <tt>[[open(2)]]</tt>과 `dup()`의 경쟁 조건 시에 `dup2()`나 `dup3()`에서 반환할 수 있다.

`EINTR`
:   `dup2()`나 `dup3()` 호출이 시그널에 의해 중단되었다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   (`dup3()`) `flags`에 유효하지 않은 값이 있다.

`EINVAL`
:   (`dup3()`) `oldfd`가 `newfd`와 같다.

`EMFILE`
:   열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다. (<tt>[[getrlimit(2)]]</tt>의 `RLIMIT_NOFILE` 설명 참고.)

## VERSIONS

리눅스 2.6.27에서 `dup3()`가 추가되었다. glibc 버전 2.9부터 지원이 사용 가능하다.

## CONFORMING TO

`dup()`, `dup2()`: POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

`dup3()`는 리눅스 전용이다.

## NOTES

`newfd`가 범위를 벗어날 때 `dup2()`가 반환하는 오류가 `fcntl(..., F_DUPFD, ...)`이 반환하는 오류와 다르다. 어떤 시스템에서는 `dup2()`가 `F_DUPFD`처럼 `EINVAL`을 반환하기도 한다.

`newfd`가 열려 있었을 때 <tt>[[close(2)]]</tt> 시점에 오류가 보고되더라도 그 정보가 유실된다. 이 점이 신경 쓰이는 경우에 (프로그램이 단일 스레드이고 시그널 핸들러에서 파일 디스크립터를 할당하지 않는 경우가 아니라면) `dup2()` 호출 전에 `newfd`를 닫는 것은 위에서 설명한 경쟁 조건 때문에 올바른 해법이 *아니다*. 대신 다음과 같은 식의 코드를 쓸 수 있을 것이다.

```c
/* 'newfd'를 복제해서 아래에서 close() 오류 검사에 사용.
   EBADF 오류는 'newfd'가 열려 있지 않다는 뜻. */

tmpfd = dup(newfd);
if (tmpfd == -1 && errno != EBADF) {
    /* 예상 외의 dup() 오류 처리 */
}

/* 'oldfd'를 'newfd'로 원자적으로 복제 */

if (dup2(oldfd, newfd) == -1) {
    /* dup2() 오류 처리 */
}

/* 이제 'newfd'가 원래 가리키던 파일에 대해서
   close() 오류 검사 */

if (tmpfd != -1) {
    if (close(tmpfd) == -1) {
        /* close 오류 처리 */
    }
}
```

## SEE ALSO

<tt>[[close(2)]]</tt>, <tt>[[fcntl(2)]]</tt>, <tt>[[open(2)]]</tt>

----

2017-09-15
