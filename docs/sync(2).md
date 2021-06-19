## NAME

sync, syncfs - 파일 시스템 캐시를 디스크로 보내기

## SYNOPSIS

```c
#include <unistd.h>

void sync(void);

int syncfs(int fd);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sync()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19: */ _BSD_SOURCE`

`syncfs()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`sync()`는 파일 시스템 메타데이터 및 캐싱 된 파일 데이터에 대한 미기록 변경 내용을 모두 기반 파일 시스템에 기록하게 한다.

`syncfs()`는 `sync()`와 비슷하되 열린 파일 디스크립터 `fd`가 가리키는 파일을 담은 파일 시스템만 동기화한다.

## RETURN VALUE

`syncfs()`는 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`sync()`는 항상 성공한다.

`syncfs()`는 적어도 다음 이유로 실패할 수 있다.

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니다.

`EIO`
:   동기화 중에 오류가 발생했다. 파일 시스템 상의 파일에 쓴 데이터에 관련된 오류일 수도 있고 파일 시스템 자체에 대한 메타데이터에 대한 것일 수도 있다.

`ENOSPC`
:   동기화 중에 디스크 공간이 고갈되었다.

`ENOSPC`, `EDQUOT`
:   <tt>[[write(2)]]</tt> 시스템 호출 시점에 공간을 할당하지 않는 NFS나 다른 파일 시스템에 데이터를 써넣었으며 이전의 어떤 쓰기 동작이 저장 공간 불충분 때문에 실패했다.

## VERSIONS

리눅스 2.6.39에서 `syncfs()`가 처음 등장했다. glibc 버전 2.14에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`sync()`: POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

`syncfs()`는 리눅스 전용이다.

## NOTES

glibc 2.2.2부터 여러 표준을 따라서 리눅스의 `sync()` 원형이 위와 같다. glibc 2.2.1 및 이전에서는 "`int sync(void)`"였으며 `sync()`가 항상 0을 반환했다.

표준 명세(가령 POSIX.1-2001)에 따르면 `sync()`가 쓰기를 예약하되 실제 쓰기가 끝나기 전에 반환할 수도 있다. 하지만 리눅스에서는 I/O 완료를 기다리며, 그래서 `sync()`와 `syncfs()`가 시스템 내지 파일 시스템의 모든 파일에 대해 `fsync()`를 호출하는 것과 같은 보장을 해 준다.

5.8 전의 커널 주 버전에서는 잘못된 파일 디스크립터를 줄 때만 `syncfs()`가 실패(`EBADF`)한다. 5.8부터는 최근 `syncfs()` 호출 이후로 하나 이상의 아이노드 기록에 실패한 경우에도 오류를 보고한다.

## BUGS

리눅스 버전 1.3.20 전에서는 반환 전에 I/O가 완료되기를 기다리지 않았다.

## SEE ALSO

`sync(1)`, <tt>[[fdatasync(2)]]</tt>, <tt>[[fsync(2)]]</tt>

----

2021-03-22
