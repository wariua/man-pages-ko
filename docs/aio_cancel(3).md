## NAME

aio_cancel - 미처리 비동기 IO 요청 취소하기

## SYNOPSIS

```c
#include <aio.h>

int aio_cancel(int fd, struct aiocb *aiocbp);
```

`-lrt`로 링크.

## DESCRIPTION

`aio_cancel()` 함수는 파일 디스크립터 `fd`에 대한 미처리 비동기 I/O 요청들을 취소하려고 시도한다. `aiocbp`가 NULL이면 해당 요청들을 모두 취소한다. 아니면 `aiocbp`가 가리키는 제어 블록이 기술하는 요청만 취소한다. (`aiocb` 구조체에 대한 설명은 <tt>[[aio(7)]]</tt> 참고.)

취소되는 요청에 대해 정상적으로 비동기 알림이 이뤄진다. (<tt>[[aio(7)]]</tt> 및 <tt>[[sigevent(7)]]</tt> 참고.) 요청 반환 상태(<tt>[[aio_return(3)]]</tt>)는 -1이 되고 요청 오류 상태(<tt>[[aio_error(3)]]</tt>)는 `ECANCELED`가 된다. 취소할 수 없는 요청의 제어 블록은 변경되지 않는다.

요청을 취소할 수 없었던 경우에는 I/O 동작 수행 후에 요청이 평상시처럼 종결된다. (이 경우 <tt>[[aio_error(3)]]</tt>가 상태 `EINPROGRESS`를 반환하게 된다.)

`aiocbp`가 NULL이 아닌데 `fd`가 그 비동기 동작을 개시했던 파일 디스크립터와 다른 경우 일어나는 결과는 명세돼 있지 않다.

어떤 동작들이 취소 가능한지는 구현에서 규정한다.

## RETURN VALUE

`aio_cancel()` 함수는 다음 값들 중 하나를 반환한다.

`AIO_CANCELED`
:   모든 요청들이 성공적으로 취소되었다.

`AIO_NOTCANCELED`
:   지정한 요청들 중 적어도 한 개가 진행 중이어서 취소되지 않았다. 이 경우 <tt>[[aio_error(3)]]</tt>를 이용해 개별 요청의 상태를 확인할 수 있다.

`AIO_ALLDONE`
:   모든 요청들이 호출 전에 이미 완료되었다.

`-1`
:   오류가 발생했다. `errno`를 확인해서 오류 원인을 알 수 있다.

## ERRORS

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니다.

`ENOSYS`
:   `aio_cancel()`이 구현돼 있지 않다.

## VERSIONS

glibc 2.1부터 `aio_cancel()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `aio_cancel()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## EXAMPLES

<tt>[[aio(7)]]</tt> 참고.

## SEE ALSO

<tt>[[aio_error(3)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_suspend(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[aio(7)]]</tt>

----

2021-03-22
