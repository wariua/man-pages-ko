## NAME

sync, syncfs - 파일 시스템 캐시를 디스크로 보내기

## SYNOPSIS

```c
#include <unistd.h>

void sync(void);

int syncfs(int fd);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>sync()</code>:</dt>
<dd>
<code>_XOPEN_SOURCE >= 500</code><br>
<code>    || /* glibc 2.19부터: */ _DEFAULT_SOURCE</code><br>
<code>    || /* glibc 버전 <= 2.19: */ _BSD_SOURCE</code>
</dd>
<dt><code>syncfs()</code>:</dt>
<dd><code>_GNU_SOURCE</code></dd>
</dl>

## DESCRIPTION

`sync()`는 파일 시스템 메타데이터 및 캐싱 된 파일 데이터에 대한 미기록 변경 내용을 모두 기반 파일 시스템에 기록하게 한다.

`syncfs()`는 `sync()`와 비슷하되 열린 파일 디스크립터 `fd`가 가리키는 파일을 담은 파일 시스템만 동기화한다.

## RETURN VALUE

`syncfs()`는 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`sync()`는 항상 성공한다.

`syncfs()`는 적어도 다음 이유로 실패할 수 있다.

<dl>
<dt><code>EBADF</code></dt>
<dd><code>fd</code>가 유효한 파일 디스크립터가 아니다.</dd>
</dl>

## VERSIONS

리눅스 2.6.39에서 `syncfs()`가 처음 등장했다. glibc 버전 2.14에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`sync()`: POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

`syncfs()`는 리눅스 전용이다.

## NOTES

glibc 2.2.2부터 여러 표준을 따라서 리눅스의 `sync()` 원형이 위와 같다. glibc 2.2.1 및 이전에서는 "`int sync(void)`"였으며 `sync()`가 항상 0을 반환했다.

표준 명세(가령 POSIX.1-2001)에 따르면 `sync()`가 쓰기를 예약하되 실제 쓰기가 끝나기 전에 반환할 수도 있다. 하지만 리눅스에서는 I/O 완료를 기다리며, 그래서 `sync()`와 `syncfs()`가 시스템 내지 파일 시스템의 모든 파일에 대해 <tt>[[fsync(2)]]</tt>를 호출하는 것과 같은 보장을 해 준다.

## BUGS

리눅스 버전 1.3.20 전에서는 반환 전에 I/O가 완료되기를 기다리지 않았다.

## SEE ALSO

`sync(1)`, <tt>[[fdatasync(2)]]</tt>, <tt>[[fsync(2)]]</tt>

----

2017-09-15
