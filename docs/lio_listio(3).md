## NAME

lio_listio - 여러 I/O 요청 개시하기

## SYNOPSIS

```c
#include <aio.h>

int lio_listio(int mode, struct aiocb *restrict const aiocb_list[restrict],
               int nitems, struct sigevent *restrict sevp);
```

`-lrt`로 링크.

## DESCRIPTION

`lio_listio()` 함수는 `aiocb_list` 배열이 기술하는 I/O 동작들을 개시한다.

`mode` 인자의 값은 다음 중 하나이다.

`LIO_WAIT`
:   모든 동작들이 완료될 때까지 호출이 블록 된다. `sevp` 인자는 무시한다.

`LIO_NOWAIT`
:   I/O 동작들을 처리 큐에 넣고 호출이 즉시 반환한다. I/O 동작들이 모두 완료되면 `sevp` 인자로 지정한 대로 비동기 알림이 이뤄진다. 자세한 내용은 <tt>[[sigevent(7)]]</tt> 참고. `sevp`가 NULL이면 비동기 알림이 이뤄지지 않는다.

`aiocb_list` 인자는 I/O 동작을 기술하는 `aiocb` 구조체 포인터들의 배열이다. 그 동작들이 실행되는 순서는 명세돼 있지 않다. `nitems` 인자는 `aiocb_list` 배열의 크기를 나타낸다. `aiocb_list`에서 널 포인터는 무시한다.

`aiocb_list`의 각 제어 블록 안에 있는 `aio_lio_opcode` 필드가 개시할 I/O 동작을 지정한다.

`LIO_READ`
:   읽기 동작을 개시한다. 이 제어 블록을 지정해서 <tt>[[aio_read(3)]]</tt>를 호출한 것처럼 동작을 큐에 넣는다.

`LIO_WRITE`
:   쓰기 동작을 개시한다. 이 제어 블록을 지정해서 <tt>[[aio_write(3)]]</tt>를 호출한 것처럼 동작을 큐에 넣는다.

`LIO_NOP`
:   이 제어 블록을 무시한다.

각 제어 블록 안의 나머지 필드들은 <tt>[[aio_read(3)]]</tt> 및 <tt>[[aio_write(3)]]</tt>에서와 의미가 같다. 각 제어 블록의 `aio_sigevent` 필드를 사용해 개별 I/O 동작별로 알림 방식을 지정할 수 있다. (<tt>[[sigevent(7)]]</tt> 참고.)

## RETURN VALUE

`mode`가 `LIO_NOWAIT`인 경우, 모든 I/O 동작이 성공적으로 큐에 들어갔으면 `lio_listio()`가 0을 반환한다. 아니면 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

`mode`가 `LIO_WAIT`인 경우, 모든 I/O 동작이 성공적으로 완료됐을 때 `lio_listio()`가 0을 반환한다. 아니면 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

`lio_listio()`의 반환 상태는 호출 자체에 대한 정보를 제공할 뿐 개별 동작들에 대한 정보는 제공하지 않는다. 한 개 이상의 I/O 동작이 실패할 수도 있지만 그게 다른 동작들의 완료를 막지는 않는다. `aiocb_list`의 개별 I/O 동작의 상태를 <tt>[[aio_error(3)]]</tt>로 알아낼 수 있다. 그리고 동작이 완료됐을 때 그 반환 상태를 <tt>[[aio_return(3)]]</tt>으로 얻을 수 있다. <tt>[[aio_read(3)]]</tt>와 <tt>[[aio_write(3)]]</tt>에서 기술하는 이유들로 개별 I/O 동작이 실패할 수 있다.

## ERRORS

`lio_listio()` 함수가 다음 이유로 실패할 수 있다.

`EAGAIN`
:   자원 부족.

`EAGAIN`
:   `nitems`로 지정한 I/O 동작 개수 때문에 제한치 `AIO_MAX`를 초과하게 된다.

`EINTR`
:   `mode`가 `LIO_WAIT`이며 I/O 동작이 모두 완료되기 전에 시그널을 잡았다. <tt>[[signal(7)]]</tt> 참고. (비동기 I/O 완료 알림에 쓰는 시그널들 중 하나일 수도 있다.)

`EINVAL`
:   `mode`가 유효하지 않거나, `nitems`가 제한치 `AIO_LISTIO_MAX`를 초과한다.

`EIO`
:   `aiocb_list`로 지정한 동작이 하나 이상 실패했다. 응용에서 <tt>[[aio_return(3)]]</tt>을 사용해 각 동작의 상태를 확인할 수 있다.

`lio_listio()`가 `EAGAIN`이나 `EINTR`, `EIO`로 실패하는 경우에는 `aiocb_list`의 동작들 중 일부가 개시되었을 수 있다. `lio_listio()`가 다른 이유로 실패하는 경우에는 어떤 I/O 동작도 개시되지 않은 것이다.

## VERSIONS

glibc 2.1부터 `lio_listio()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `lio_listio()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

제어 블록들을 사용하기 전에 0으로 채우는 게 좋다. I/O 동작이 진행 중인 동안 제어 블록이 변경되어선 안 된다. 읽어서 채우거나 기록하려는 버퍼 영역에 동작 중에 접근해서는 안 되며, 접근 시 규정돼 있지 않은 결과가 발생할 수 있다. 관련 메모리 영역들이 유효한 상태로 유지돼야 한다.

동시에 이뤄지는 I/O 동작들에 같은 `aiocb` 구조체를 지정하는 결과는 규정돼 있지 않다.

## SEE ALSO

<tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_error(3)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_suspend(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[aio(7)]]</tt>

----

2021-03-22
