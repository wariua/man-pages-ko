## NAME

aio_suspend - 비동기 I/O 동작 또는 타임아웃 기다리기

## SYNOPSIS

```c
#include <aio.h>

int aio_suspend(const struct aiocb *const aiocb_list[], int nitems,
                const struct timespec *restrict timeout);
```

`-lrt`로 링크.

## DESCRIPTION

`aio_suspend()` 함수는 다음 중 한 경우가 발생할 때까지 호출 스레드를 중지한다.

* 목록 `aiocb_list`의 비동기 I/O 요청이 하나 이상 완료됐다.

* 시그널이 전달됐다.

* `timeout`이 NULL이 아니고 거기 지정된 시간이 지났다. (`timespec` 구조체에 대한 자세한 내용은 <tt>[[nanosleep(2)]]</tt> 참고.)

`nitems` 인자는 `aiocb_list`의 항목 개수를 나타낸다. `aiocb_list`가 가리키는 목록의 각 항목은 NULL이거나 (그러면 무시됨) <tt>[[aio_read(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>로 개시한 I/O의 제어 블록에 대한 포인터여야 한다. (`aiocb` 구조체에 대한 설명은 <tt>[[aio(7)]]</tt> 참고.)

`CLOCK_MONOTONIC`이 지원되면 그 클럭을 타임아웃 시간 측정에 사용한다. (<tt>[[clock_gettime(2)]]</tt> 참고.)

## RETURN VALUE

`aiocb_list`에 지정된 I/O 요청들 중 하나가 완료된 후 이 함수가 반환하는 경우에는 0을 반환한다. 아니면 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EAGAIN`
:   지정한 동작 하나라도 완료되기 전에 호출이 타임아웃되었다.

`EINTR`
:   시그널로 호출이 끝났다. (기다리던 동작들 중 하나의 완료 시그널일 수도 있다.) <tt>[[signal(7)]]</tt> 참고.

`ENOSYS`
:   `aio_suspend()`가 구현돼 있지 않다.

## VERSIONS

glibc 2.1부터 `aio_suspend()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `aio_suspend()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

POSIX에서는 매개변수가 `restrict`여야 한다고 명세하고 있지 않다. glibc 한정이다.

## NOTES

NULL 아닌 `timeout`에 시간을 0으로 지정하면 폴링이 가능하다.

`aio_suspend()` 호출 시점에 `aiocb_list`에 지정된 비동기 I/O 동작이 하나 이상 이미 완료돼 있었던 경우에는 호출이 즉시 반환한다.

`aio_suspend()`가 성공적으로 반환한 다음에 어느 I/O 동작이 완료됐는지 알려면 `aiocb_list`가 가리키는 `aiocb` 구조체 목록을 <tt>[[aio_error(3)]]</tt>로 탐색하면 된다.

## BUGS

glibc의 `aio_suspend()` 구현은 비동기 시그널 안전이 아니며, 이는 POSIX.1의 요구 사항 위반이다.

## SEE ALSO

<tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_error(3)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[aio(7)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
