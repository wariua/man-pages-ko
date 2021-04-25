## NAME

truncate, ftruncate - 파일을 지정한 길이로 절단하기

## SYNOPSIS

```c
#include <unistd.h>
#include <sys/types.h>

int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`truncate()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200809L`<br>
    `    || /* glibc <= 2.19: */ _BSD_SOURCE`

`ftruncate()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.3.5부터: */ _POSIX_C_SOURCE >= 200112L`<br>
    `    || /* glibc <= 2.19: */ _BSD_SOURCE`

## DESCRIPTION

`truncate()` 및 `ftruncate()` 함수는 `path`가 지명하거나 `fd`가 가리키는 정규 파일을 정확히 `length` 바이트 크기가 되게 한다.

파일이 원래 그 크기보다 컸으면 뒤쪽 데이터를 잃게 된다. 파일이 원래 그 크기보다 작았으면 확장되며, 확장된 부분을 읽으면 널 바이트('\0')가 나온다.

파일 오프셋은 바뀌지 않는다.

크기가 바뀌었으면 파일의 `st_ctime` 및 `st_mtime` 필드(각각 최근 상태 변경 시간 및 최근 수정 시간. <tt>[[inode(7)]]</tt> 참고)가 갱신되며, 모드 비트 set-user-ID 및 set-group-ID가 지워질 수 있다.

`ftruncate()`에서는 파일이 쓰기 가능하게 열려 있어야 한다. `truncate()`에서는 파일이 쓰기 가능해야 한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`truncate()`:

`EACCES`
:   경로 선두부의 어느 요소에 대해 탐색 권한이 거부되었다. 또는 지정한 파일이 사용자에게 쓰기 가능하지 않다. (<tt>[[path_resolution(7)]]</tt>도 참고.)

`EFAULT`
:   `path` 인자가 프로세스에 할당된 주소 공간 밖을 가리키고 있다.

`EFBIG`
:   `length` 인자가 최대 파일 크기보다 크다. (XSI)

`EINTR`
:   끝나기를 기다리며 블록돼 있는 동안 호출이 시그널 핸들러에 의해 중단되었다. <tt>[[fcntl(2)]]</tt> 및 <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `length` 인자가 음수이거나 최대 파일 크기보다 크다.

`EIO`
:   아이노드 갱신 중 I/O 오류가 발생했다.

`EISDIR`
:   지정한 파일이 디렉터리이다.

`ELOOP`
:   경로명을 변환하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   경로명의 어느 요소가 255글자를 넘었다. 또는 전체 경로명이 1023글자를 넘었다.

`ENOENT`
:   지정한 파일이 존재하지 않는다.

`ENOTDIR`
:   경로 선두부의 한 요소가 디렉터리가 아니다.

`EPERM`
:   하위 파일 시스템에서 현재 크기 넘게 파일을 확장하는 걸 지원하지 않는다.

`EPERM`
:   파일 봉인 때문에 동작이 막혔다. <tt>[[fcntl(2)]]</tt> 참고.

`EROFS`
:   지정한 파일이 읽기 전용 파일 시스템에 위치해 있다.

`ETXTBSY`
:   파일이 실행 파일인데 현재 실행 중이다.

같은 오류들이 `ftruncate()`에도 해당하되 `path`에서 뭔가 잘못될 수 있는 대신 파일 디스크립터 `fd`에서 뭔가 잘못될 수 있다.

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니다.

`EBADF` 또는 `EINVAL`
:   `fd`가 쓰기 가능하게 열려 있지 않다.

`EINVAL`
:   `fd`가 정규 파일이나 POSIX 공유 메모리 객체를 가리키고 있지 않다.

`EINVAL` 또는 `EBADF`
:   파일 디스크립터 `fd`가 쓰기 가능하게 열려 있지 않다. POSIX에서는 이 경우에 어느 쪽 오류도 허용하며, 이식 가능한 응용에서는 두 오류를 모두 처리하는 게 좋다. (리눅스에서는 `EINVAL`을 내놓는다.)

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, 4.4BSD, SVr4. (4.2BSD에서 이 호출들이 처음 등장했다.)

## NOTES

POSIX 공유 메모리 객체의 크기를 설정하는 데에도 `ftruncate()`를 쓸 수 있다. <tt>[[shm_open(3)]]</tt> 참고.

DESCRIPTION의 세부 내용들은 XSI 준수 시스템에 대한 것이다. XSI 비준수 시스템의 경우, POSIX 표준에서는 `length`가 파일 길이를 초과할 때 `ftruncate()`의 동작 방식으로 두 가지를 허용하는데 (참고로 그런 상황에서 `truncate()`에 대해선 아무것도 명세돼 있지 않다), 오류를 반환할 수도 있고, 파일을 확장할 수도 있다. 대다수 유닉스 시스템과 마찬가지로 리눅스에서는 자체 파일 시스템을 다룰 때 XSI 요구 사항을 따른다. 하지만 몇몇 외래 파일 시스템에서는 `truncate()`와 `ftruncate()`를 이용해 파일을 현재 길이 너머로 확장하는 걸 허용하지 않는다. 리눅스에서 눈에 띄는 예로 VFAT이 있다.

원래의 리눅스 `truncate()` 및 `ftruncate()` 시스템 호출은 큰 파일 오프셋을 다룰 수 있도록 설계되지 않았다. 그래서 리눅스 2.4에서 큰 파일을 다루기 위한 `truncate64()` 및 `ftruncate64()`를 추가했다. 하지만 glibc의 래퍼 함수에서 투명하게 최신 시스템 호출을 이용하기 때문에 glibc를 쓰는 응용에서는 이런 내용을 모르고 있어도 된다.

<tt>[[syscall(2)]]</tt>에서 설명하는 이유들 때문에 일부 32비트 아키텍처에서는 이 시스템 호출들의 호출 시그너처가 다르다.

## BUGS

glibc 2.12에 헤더 파일 버그가 있어서 `ftruncate()` 선언을 노출시키기 위해 필요한 `_POSIX_C_SOURCE` 최솟값이 200112L가 아닌 200809L였다. 이후 glibc 버전들에서 고쳐졌다.

## SEE ALSO

`truncate(1)`, <tt>[[open(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[path_resolution(7)]]</tt>

----

2021-03-22
