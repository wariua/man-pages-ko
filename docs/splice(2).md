## NAME

splice - 파이프에 데이터 이어 붙이기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <fcntl.h>

ssize_t splice(int fd_in, loff_t *off_in, int fd_out,
               loff_t *off_out, size_t len, unsigned int flags);
```

## DESCRIPTION

`splice()`는 커널 주소 공간과 사용자 주소 공간 사이 복사 없이 두 파일 디스크립터 간에 데이터를 옮긴다. 파일 디스크립터 `fd_in`에서 파일 디스크립터 `fd_out`으로 최대 `len` 바이트까지 데이터를 이동시키는데, 두 파일 디스크립터 중 하나는 파이프를 가리켜야 한다.

`fd_in`과 `off_in`에 다음 의미론이 적용된다.

* `fd_in`이 파이프를 가리키면 `off_in`이 NULL이어야 한다.

* `fd_in`이 파이프를 가리키지 않고 `off_in`이 NULL이라면 파일 오프셋부터 시작해서 `fd_in`에서 바이트들을 읽어 들이며, 파일 오프셋을 적절히 조정한다.

* `fd_in`이 파이프를 가리키지 않고 `off_in`이 NULL이 아니라면 `off_in`은 `fd_in`에서 바이트들을 읽어 들이기 시작할 오프셋을 나타내는 버퍼를 가리켜야 한다. 이 경우에는 `fd_in`의 파일 오프셋을 바꾸지 않는다.

마찬가지 내용이 `fd_out`과 `off_out`에 적용된다.

`flags` 인자는 다음 값을 0개 이상 OR 해서 구성한 비트 마스크이다.

`SPLICE_F_MOVE`
:   페이지 복사 대신 이동을 시도한다. 커널에게 주는 힌트일 뿐이다. 파이프의 페이지를 커널이 옮길 수 없거나 파이프 버퍼가 가리키는 페이지가 가득 차 있지 않으면 여전히 페이지를 복사할 수도 있다. 이 플래그의 초기 구현에 버그가 많았고, 그래서 리눅스 2.6.21부터는 no-op이다. (`splice()` 호출에서는 계속 허용한다.) 향후에 제대로 된 구현으로 되살아날 수도 있다.

`SPLICE_F_NONBLOCK`
:   I/O에서 블록 하지 않는다. 파이프 이어 붙이기 동작을 논블록 방식으로 만든다. 그래도 `splice()`가 블록 할 수도 있다. 이어 붙이는 파일 디스크립터에서 (`O_NONBLOCK` 플래그가 설정되어 있지 않다면) 블록 할 수도 있기 때문이다.

`SPLICE_F_MORE`
:   이어지는 `splice()`로 데이터가 더 올 예정이다. `fd_out`이 소켓을 가리키는 경우에 유용한 힌트이다. (<tt>[[send(2)]]</tt>의 `MSG_MORE` 설명과 <tt>[[tcp(7)]]</tt>의 `TCP_CORK` 설명 참고.)

`SPLICE_F_GIFT`
:   `splice()`에서는 쓰지 않는다. <tt>[[vmsplice(2)]]</tt> 참고.

## RETURN VALUE

성공 완료 시 `splice()`는 파이프에 이어 붙인 바이트 수를 반환한다.

반환 값 0은 입력 끝을 나타낸다. `fd_in`이 파이프를 가리키는 경우에는 이동시킬 데이터가 없었으며 파이프의 쓰기 쪽에 연결된 쓰기 수행자가 없어서 블록 하는 것이 의미가 없었음을 뜻한다.

오류 시 `splice()`는 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EAGAIN`
:   `flags`에 `SPLICE_F_NONBLOCK`이 지정되었거나 파일 디스크립터들 중 하나에 논블로킹 표시(`O_NONBLOCK`)가 돼 있는데 동작이 블록 되려 한다.

`EBADF`
:   한쪽 내지 양쪽 파일 디스크립터가 유효하지 않거나 읽기-쓰기 모드가 올바르지 않다.

`EINVAL`
:   대상 파일 시스템에서 이어 붙이기를 지원하지 않는다.

`EINVAL`
:   대상 파일이 덧붙이기 모드로 열려 있다.

`EINVAL`
:   어느 파일 디스크립터도 파이프를 가리키고 있지 않다.

`EINVAL`
:   탐색 불가능 장치(가령 파이프)에 오프셋을 주었다.

`EINVAL`
:   `fd_in`과 `fd_out`이 같은 파이프를 가리키고 있다.

`ENOMEM`
:   메모리 부족.

`ESPIPE`
:   `off_in`이나 `off_out`이 NULL이 아닌데 대응하는 파일 디스크립터가 파이프를 가리키고 있다.

## VERSIONS

리눅스 2.6.17에서 `splice()` 시스템 호출이 처음 등장했다. glibc 버전 2.5에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

세 가지 시스템 호출 `splice()`, <tt>[[vmsplice(2)]]</tt>, <tt>[[tee(2)]]</tt>를 통해 사용자 프로그램에서 임의 커널 버퍼를 완전히 제어할 수 있는데, 그 버퍼는 커널 내에서 파이프에 쓰는 버퍼와 같은 타입으로 구현되어 있다. 이 시스템 호출들은 대략 다음 작업을 수행한다.

`splice()`
:   버퍼에서 임의 파일 디스크립터로, 또는 반대로, 또는 한 버퍼에서 다른 버퍼로 데이터를 옮긴다.

<tt>[[tee(2)]]</tt>
:   한 버퍼에서 다른 버퍼로 데이터를 "복사"한다.

<tt>[[vmsplice(2)]]</tt>
:   사용자 공간에서 버퍼로 데이터를 "복사"한다.

복사라고 하지만 일반적으로 실제 복사를 피한다. 이를 위해 커널에서는 커널 메모리 페이지에 대한 참조 카운트 포인터들의 집합으로 파이프 버퍼를 구현한다. 페이지를 가리키는 (출력 버퍼를 위한) 새 포인터를 생성하고 그 페이지에 대한 참조 카운트를 올리는 것으로 버퍼 내에 페이지 "사본"을 생성한다. 즉, 버퍼 페이지가 아니라 포인터만 복사한다.

리눅스 2.6.30 및 이전에서는 `fd_in`과 `fd_out` 중 한쪽만 파이프여야 했다. 리눅스 2.6.31부터 두 인자 모두 파이프를 가리킬 수 있다.

## EXAMPLE

<tt>[[tee(2)]]</tt> 참고.

## SEE ALSO

<tt>[[copy_file_range(2)]]</tt>, <tt>[[sendfile(2)]]</tt>, <tt>[[tee(2)]]</tt>, <tt>[[vmsplice(2)]]</tt>, <tt>[[pipe(7)]]</tt>

----

2019-05-09