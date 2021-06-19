## NAME

aio_return - 비동기 I/O 동작의 반환 상태 얻기

## SYNOPSIS

```c
#include <aio.h>

ssize_t aio_return(struct aiocb *aiocbp);
```

`-lrt`로 링크.

## DESCRIPTION

`aio_return()` 함수는 `aiocbp`가 가리키는 제어 블록의 비동기 I/O 요청에 대해 최종 반환 상태를 반환한다. (`aiocb` 구조체에 대한 설명은 <tt>[[aio(7)]]</tt> 참고.)

한 요청에 대해 이 함수를 한 번만, <tt>[[aio_error(3)]]</tt>가 `EINPROGRESS` 아닌 뭔가를 반환했을 때만 호출해야 한다.

## RETURN VALUE

비동기 I/O 동작이 완료된 경우에 이 함수는 동기적인 <tt>[[read(2)]]</tt>, <tt>[[write(2)]]</tt>, `fsync(2)`, `fdatasync(2)` 호출이 반환했을 값을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

비동기 I/O 동작이 아직 완료되지 않은 경우에 `aio_return()`의 반환 값과 영향은 규정돼 있지 않다.

## ERRORS

`EINVAL`
:   `aiocbp`가 아직 반환 상태를 가져오지 않은 비동기 I/O 요청의 제어 블록을 가리키고 있지 않다.

`ENOSYS`
:   `aio_return()`이 구현돼 있지 않다.

## VERSIONS

glibc 2.1부터 `aio_return()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `aio_return()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## EXAMPLES

<tt>[[aio(7)]] 참고.

## SEE ALSO

<tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_error(3)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_suspend(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[aio(7)]]</tt>

----

2021-03-22
