## NAME

aio_read - 비동기 읽기

## SYNOPSIS

```c
#include <aio.h>

int aio_read(struct aiocb *aiocbp);
```

`-lrt`로 링크.

## DESCRIPTION

`aio_read()` 함수는 `aiocbp`가 가리키는 버퍼에 기술된 I/O 요청을 큐에 넣는다. 이 함수는 <tt>[[read(2)]]</tt>의 비동기 형태다. 다음 호출에서

```c
read(fd, buf, count)
```

인자들이 각각 `aiocbp`가 가리키는 구조체의 `aio_fildes`, `aio_buf`, `aio_nbytes` 필드에 (순서대로) 대응한다. (`aiocb` 구조체에 대한 설명은 <tt>[[aio(7)]]</tt> 참고.)

파일 오프셋과 상관없이 절대 위치 `aiocbp->aio_offset`부터 데이터를 읽는다. 호출 후의 파일 오프셋 값은 명세돼 있지 않다.

"비동기"라는 것은 요청을 큐에 넣자마자 이 호출이 반환한다는 뜻이다. 호출 반환 때 읽기가 완료돼 있을 수도 있고 아닐 수도 있다. 완료 여부를 <tt>[[aio_error(3)]]</tt>로 검사한다. 그리고 완료된 I/O 동작의 반환 상태를 <tt>[[aio_return(3)]]</tt>으로 얻는다. `aiocbp->aio_sigevent`를 적절히 설정해서 I/O 완료 알림을 비동기적으로 받을 수 있다. 자세한 내용은 <tt>[[sigevent(7)]]</tt> 참고.

`_POSIX_PRIORITIZED_IO`가 정의돼 있고 이 파일에서 지원한다면 호출 프로세스의 우선순위에서 `aiocbp->aio_reqprio`를 뺀 우선순위로 비동기 동작이 제출된다.

`aiocbp->aio_lio_opcode` 필드는 무시된다.

정규 파일에서 최대 오프셋 너머에서는 어떤 데이터도 읽지 않는다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 요청을 큐에 넣지 않고 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다. 오류를 나중에 탐지한 경우 <tt>[[aio_return(3)]]</tt>(상태 -1 반환)이나 <tt>[[aio_error(3)]]</tt>(`errno`로 받았을 `EBADF` 같은 오류 상태)를 통해 보고하게 된다.

## ERRORS

`EAGAIN`
:   자원 부족.

`EBADF`
:   `aio_fildes`가 읽기 가능하게 열린 유효한 파일 디스크립터가 아니다.

`EINVAL`
:   `aio_offset`, `aio_reqprio`, `aio_nbytes` 중 하나 이상이 유효하지 않다.

`ENOSYS`
:   `aio_read()`가 구현돼 있지 않다.

`EOVERFLOW`
:   파일이 정규 파일이며 파일 끝 전에서 읽기를 시작해서 최소 한 바이트를 읽으려 하는데 시작 위치가 파일의 최대 오프셋을 지난 지점이다.

## VERSIONS

glibc 2.1부터 `aio_read()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `aio_read()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

제어 블록 버퍼를 사용하기 전에 0으로 채우는 게 좋다. 읽기 동작이 진행 중인 동안 제어 블록이 변경되어선 안 된다. 읽어서 채울 버퍼 영역에 동작 중에 접근해서는 안 되며, 접근 시 규정돼 있지 않은 결과가 발생할 수 있다. 관련 메모리 영역들이 유효한 상태로 유지돼야 한다.

동시에 이뤄지는 I/O 동작들에 같은 `aiocb` 구조체를 지정하는 결과는 규정돼 있지 않다.

## EXAMPLES

<tt>[[aio(7)]]</tt> 참고.

## SEE ALSO

<tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_error(3)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_suspend(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[aio(7)]]</tt>

----

2021-03-22
