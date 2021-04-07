## NAME

aio_write - 비동기 쓰기

## SYNOPSIS

```c
#include <aio.h>

int aio_write(struct aiocb *aiocbp);
```

`-lrt`로 링크.

## DESCRIPTION

`aio_write()` 함수는 `aiocbp`가 가리키는 버퍼에 기술된 I/O 요청을 큐에 넣는다. 이 함수는 `write(2)`의 비동기 형태이다. 다음 호출에서

```c
write(fd, buf, count)
```

인자들이 각각 `aiocbp`가 가리키는 구조체의 `aio_fildes`, `aio_buf`, `aio_nbytes` 필드에 (순서대로) 대응한다. (`aiocb` 구조체에 대한 설명은 <tt>[[aio(7)]]</tt> 참고.)

`O_APPEND`가 설정돼 있지 않으면 파일 오프셋과 상관없이 절대 위치 `aiocbp->aio_offset`부터 데이터를 쓴다. `O_APPEND`가 설정돼 있으면 `aio_write()` 호출 순서대로 파일 끝에 데이터를 쓴다. 호출 후의 파일 오프셋 값은 명세돼 있지 않다.

"비동기"라는 것은 요청을 큐에 넣자마자 이 호출이 반환한다는 뜻이다. 호출 반환 때 쓰기가 완료돼 있을 수도 있고 아닐 수도 있다. 완료 여부를 <tt>[[aio_error(3)]]</tt>로 검사한다. 그리고 완료된 I/O 동작의 반환 상태를 <tt>[[aio_return(3)]]</tt>으로 얻는다. `aiocbp->aio_sigevent`를 적절히 설정해서 I/O 완료 알림을 비동기적으로 받을 수 있다. 자세한 내용은 <tt>[[sigevent(7)]]</tt> 참고.

`_POSIX_PRIORITIZED_IO`가 정의돼 있으며 이 파일에서 지원하는 경우 호출 프로세스의 우선순위에서 `aiocbp->aio_reqprio`를 뺀 우선순위로 비동기 동작을 제출한다.

`aiocbp->aio_lio_opcode` 필드는 무시한다.

정규 파일에서 최대 오프셋 너머로는 어떤 데이터도 쓰지 않는다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 요청이 큐에 들어가지 않고, -1을 반환하며 `errno`를 적절히 설정한다. 오류를 나중에 탐지한 경우 <tt>[[aio_return(3)]]</tt>(상태 -1 반환)이나 <tt>[[aio_error(3)]]</tt>(`errno`로 받았을 `EBADF` 같은 오류 상태)를 통해 보고한다.

## ERRORS

<dl>
<dt><code>EAGAIN</code></dt>
<dd>자원 부족.</dd>
<dt><code>EBADF</code></dt>
<dd><code>aio_fildes</code>가 쓰기 가능하게 열린 유효한 파일 디스크립터가 아니다.</dd>
<dt><code>EFBIG</code></dt>
<dd>파일이 정규 파일이며 최소 한 바이트를 쓰려고 하는데 시작 위치가 파일의 최대 오프셋을 지난 지점이다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>aio_offset</code>, <code>aio_reqprio</code>, <code>aio_nbytes</code> 중 하나 이상이 유효하지 않다.</dd>
<dt><code>ENOSYS</code></dt>
<dd><code>aio_write()</code>가 구현돼 있지 않다.</dd>
</dl>

## VERSIONS

glibc 2.1부터 `aio_write()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `aio_write()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

제어 블록 버퍼를 사용하기 전에 0으로 채우는 게 좋다. 쓰기 동작이 진행 중인 동안 제어 블록이 변경되어선 안 된다. 기록하려는 버퍼 영역에 동작 중에 접근해서는 안 되며, 접근 시 규정돼 있지 않은 결과가 발생할 수 있다. 관련 메모리 영역들이 유효한 상태로 유지돼야 한다.

동시에 이뤄지는 I/O 동작들에 같은 `aiocb` 구조체를 지정하는 결과는 규정돼 있지 않다.

## SEE ALSO

<tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_error(3)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_suspend(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[aio(7)]]</tt>

----

2017-09-15
