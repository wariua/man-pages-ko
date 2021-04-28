## NAME

aio_error - 비동기 I/O 동작의 오류 상태 얻기

## SYNOPSIS

```c
#include <aio.h>

int aio_error(const struct aiocb *aiocbp);
```

`-lrt`로 링크.

## DESCRIPTION

`aio_error()` 함수는 `aiocbp`가 가리키는 제어 블록의 비동기 I/O 요청에 대한 오류 상태를 반환한다. (`aiocb` 구조체에 대한 설명은 <tt>[[aio(7)]]</tt> 참고.)

## RETURN VALUE

이 함수는 다음 중 하나를 반환한다.

* `EINPROGRESS`: 요청이 아직 완료되지 않은 경우.

* `ECANCELED`: 요청이 취소된 경우.

* 0: 요청이 성공적으로 완료된 경우.

* 양수 오류 번호: 비동기 I/O 동작이 실패한 경우. 동기적인 `read(2)`, `write(2)`, `fsync(2)`, `fdatasync(2)` 호출에서 `errno` 변수에 저장됐을 값과 같다.

## ERRORS

`EINVAL`
:   `aiocbp`가 아직 반환 상태를 가져오지 않은 (<tt>[[aio_return(3)]]</tt> 참고) 비동기 I/O 요청의 제어 블록을 가리키고 있지 않다.

`ENOSYS`
:   `aio_error()`가 구현돼 있지 않다.

## VERSIONS

glibc 2.1부터 `aio_error()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `aio_error()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## EXAMPLES

<tt>[[aio(7)]] 참고.

## SEE ALSO

<tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_suspend(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[aio(7)]]</tt>

----

2021-03-22
