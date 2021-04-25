## NAME

readahead - 파일을 페이지 캐시로 미리 읽어 들이기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <fcntl.h>

ssize_t readahead(int fd, off64_t offset, size_t count);
```

## DESCRIPTION

`readahead()`는 파일에서 미리 읽기를 개시해서 이후의 파일 읽기가 캐시에서 처리되어 디스크 I/O에서 블록 되지 않게 한다. (미리 읽기가 충분히 일찍 개시되었으며 중간에 시스템의 다른 활동이 캐시에서 그 페이지들을 내려 보내지 않았다고 가정한다.)

`fd` 인자는 읽을 파일을 나타내는 파일 디스크립터이다. `offset` 인자는 데이터를 읽기 시작할 지점을 나타내며 `count`는 읽을 바이트 수를 나타낸다. I/O가 페이지 단위로 수행되므로 실질적으로는 `offset`을 페이지 경계로 내림 하고 `(offset+count)` 이상 되는 위치에 있는 다음 페이지 경계까지의 바이트들을 읽는다. `readahead()`는 파일 끝 너머까지 읽지 않는다. 파일 디스크립터 `fd`가 가리키는 열린 파일 기술 항목의 파일 오프셋이 바뀌지 않는다.

## RETURN VALUE

성공 시 `readahead()`는 0을 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니거나 읽기 가능하게 열리지 않았다.

`EINVAL`
:   `fd`가 `readahead()`를 적용할 수 있는 파일 종류를 가리키지 않고 있다.

## VERSIONS

리눅스 2.4.13에서 `readahead()` 시스템 호출이 등장했다. glibc 버전 2.3부터 지원을 제공한다.

## CONFORMING TO

`readahead()` 시스템 호출은 리눅스 전용이므로 이식성이 있어야 하는 응용에서는 사용하지 말아야 한다.

## NOTES

<tt>[[syscall(2)]]</tt>에서 설명하는 이유들 때문에 일부 32비트 아키텍처에서는 이 시스템 호출의 호출 시그너처가 다르다.

## BUGS

`readahead()`는 배경 읽기 스케줄링을 시도하고서 즉시 반환한다. 하지만 요청 블록들의 위치를 알기 위해 필요한 파일 시스템 메타데이터를 읽는 동안 블록 될 수도 있다. ext[234] 사용 시 익스텐트 대신 간접 블록을 쓰는 큰 파일에서 이런 경우가 자주 일어나며, 그래서 요청 데이터를 읽어 들일 때까지 호출이 블록 되는 것처럼 보인다.

## SEE ALSO

<tt>[[lseek(2)]]</tt>, <tt>[[madvise(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[posix_fadvise(2)]]</tt>, `read(2)`

----

2021-03-22
