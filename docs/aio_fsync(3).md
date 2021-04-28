## NAME

aio_fsync - 비동기 파일 동기화

## SYNOPSIS

```c
#include <aio.h>

int aio_fsync(int op, struct aiocb *aiocbp);
```

`-lrt`로 링크.

## DESCRIPTION

`aio_fsync()` 함수는 `aiocbp->aio_filedes`에 연계된 모든 미처리 비동기 I/O 동작들에 동기화를 한다. (`aiocb` 구조체에 대한 설명은 <tt>[[aio(7)]]</tt> 참고.)

더 정확하게는 `op`가 `O_SYNC`이면 <tt>[[fsync(2)]]</tt>를 호출한 것처럼 현재 큐에 있는 모든 I/O 동작들이 완료된다. 그리고 `op`가 `O_DSYNC`이면 이 호출은 <tt>[[fdatasync(2)]]</tt>의 비동기 형태가 된다.

참고로 이 호출은 요청만 할 뿐이다. 즉 I/O가 완료되기를 기다리지 않는다.

`aiocbp`가 가리키는 구조체에서 `aio_fildes` 외에 이 호출에서 유일하게 사용하는 필드는 `aio_sigevent` 필드(`sigevent` 구조체. <tt>[[sigevent(7)]]</tt>에서 설명)인데, 원하는 비동기 완료 알림 종류를 나타낸다. 다른 필드들은 모두 무시된다.

## RETURN VALUE

성공 시 (동기화 요청을 성공적으로 큐에 넣었으면) 함수가 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EAGAIN`
:   자원 부족.

`EBADF`
:   `aio_fildes`가 쓰기 가능하게 열린 유효한 파일 디스크립터가 아니다.

`EINVAL`
:   이 파일에서 동기화 된 I/O를 지원하지 않는다. 또는 `op`가 `O_SYNC`나 `O_DSYNC`가 아니다.

`ENOSYS`
:   `aio_fsync()`가 구현돼 있지 않다.

## VERSIONS

glibc 2.1부터 `aio_fsync()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `aio_fsync()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## SEE ALSO

<tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_error(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_suspend(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[aio(7)]]</tt>, <tt>[[sigevent(7)]]</tt>

----

2021-03-22
