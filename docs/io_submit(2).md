## NAME

io_submit - 비동기 I/O 처리 요청 제출하기

## SYNOPSIS

```c
#include <linux/aio_abi.h>          /* 필요한 타입 정의 */

int io_submit(aio_context_t ctx_id, long nr, struct iocb **iocbpp);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`io_submit()` 시스템 호출은 AIO 문맥 `ctx_id`에서 처리 큐에 `nr` 개 I/O 요청 블록을 넣는다. `iocbpp` 인자는 `nr` 개 AIO 제어 블록의 배열이어야 하며 이를 문맥 `ctx_id`로 제출하게 된다.

`linux/aio_abi.h`에 정의돼 있는 `iocb`(I/O 제어 블록) 구조체에 I/O 동작을 제어하는 매개변수들이 있다.

```c
#define <linux/aio_abi.h>

struct iocb {
    __u64   aio_data;
    __u32   PADDED(aio_key, aio_rw_flags);
    __u16   aio_lio_opcode;
    __s16   aio_reqprio;
    __u32   aio_fildes;
    __u64   aio_buf;
    __u64   aio_nbytes;
    __s64   aio_offset;
    __u64   aio_reserved2;
    __u32   aio_flags;
    __u32   aio_resfd;
};
```

이 구조체의 필드들은 다음과 같다.

<dl>
<dt><code>aio_data</code></dt>
<dd>I/O 완료 시 이 데이터가 <code>io_event</code> 구조체의 <code>data</code> 필드로 복사된다. (<tt>[[io_getevents(2)]]</tt> 참고.)</dd>

<dt><code>aio_key</code></dt>
<dd>커널 내부용 필드이다. <code>io_submit()</code> 호출 후에 이 필드를 변경하지 말 것.</dd>

<dt><code>aio_rw_flags</code></dt>
<dd>

구조체와 함께 전달하는 읽기/쓰기 플래그들이다. 유효한 값은 다음과 같다.

 <dl>
 <dt><code>RWF_APPEND</code> (리눅스 4.16부터)</dt>
 <dd>파일 끝에 데이터를 덧붙임. <tt>[[pwritev2(2)]]</tt>에 있는 같은 이름의 플래그 설명과 <tt>[[open(2)]]</tt>의 <code>O_APPEND</code> 설명 참고.</dd>

 <dt><code>RWF_DSYNC</code> (리눅스 4.7부터)</dt>
 <dd>동기화 I/O 데이터 무결성 요건에 따라 쓰기 동작이 완료됨. <tt>[[pwritev2(2)]]</tt>에 있는 같은 이름의 플래그 설명과 <tt>[[open(2)]]</tt>의 <code>O_DSYNC</code> 설명 참고.</dd>

 <dt><code>RWF_HIPRI</code> (리눅스 4.6부터)</dt>
 <dd>높은 우선도 요청. 가능하면 폴링.</dd>

 <dt><code>RWF_NOWAIT</code> (리눅스 4.14부터)</dt>
 <dd>커널 내에서 파일 블록 할당이나 변경 페이지 플러싱, 뮤텍스 락 같은 동작이나 혼잡한 블록 장치 때문에 I/O가 블록 되려는 경우에 대기하지 않는다. 이 중 한 조건이라도 해당하면 <code>io_event</code> 구조체의 <code>res</code> 필드에 반환 값 <code>-EAGAIN</code>을 담아서 제어 블록을 즉시 반환한다. (<tt>[[io_getevents(2)]]</tt> 참고.)</dd>

 <dt><code>RWF_SYNC</code> (리눅스 4.7부터)</dt>
 <dd>동기화 I/O 파일 무결성 요건에 따라 쓰기 동작이 완료됨. <tt>[[pwritev2(2)]]</tt>에 있는 같은 이름의 플래그 설명과 <tt>[[open(2)]]</tt>의 <code>O_SYNC</code> 설명 참고.</dd>
 </dl>
</dd>

<dt><code>aio_opcode</code></dt>
<dd>

<code>iocb</code> 구조체로 수행할 I/O 종류를 나타낸다. 유효한 값들이 <code>linux/aio_abi.h</code>에 열거형으로 정의돼 있다.

```c
enum {
    IOCB_CMD_PREAD = 0,
    IOCB_CMD_PWRITE = 1,
    IOCB_CMD_FSYNC = 2,
    IOCB_CMD_FDSYNC = 3,
    IOCB_CMD_NOOP = 6,
    IOCB_CMD_PREADV = 7,
    IOCB_CMD_PWRITEV = 8,
};
```
</dd>

<dt><code>aio_reqprio</code></dt>
<dd>요청 우선순위를 나타낸다.</dd>

<dt><code>aio_fildes</code></dt>
<dd>I/O 동작을 수행할 파일 디스크립터이다.</dd>

<dt><code>aio_buf</code></dt>
<dd>읽기 또는 쓰기 동작에서 데이터 이동에 사용할 버퍼이다.</dd>

<dt><code>aio_nbytes</code></dt>
<dd><code>aio_buf</code>가 가리키는 버퍼의 크기이다.</dd>

<dt><code>aio_offset</code></dt>
<dd>I/O 동작을 수행할 파일 오프셋이다.</dd>

<dt><code>aio_flags</code></dt>
<dd>

<code>iocb</code> 구조체에 연계할 플래그들의 집합이다. 유효한 값은 다음과 같다.

 <dl>
 <dt><code>IOCB_FLAG_RESFD</code></dt>
 <dd>비동기 I/O 제어 로직에서 완료 시 <code>aio_resfd</code>에 있는 파일 디스크립터로 알림을 줘야 한다.</dd>

 <dt><code>IOCB_FLAG_IOPRIO</code> (리눅스 4.18부터)</dt>
 <dd><code>aio_reqprio</code> 필드를 <code>linux/ioprio.h</code>에 정의된 <code>IOPRIO_VALUE</code>로 해석한다.</dd>
 </dl>
</dd>

<dt><code>aio_resfd</code></dt>
<dd>비동기 I/O 완료 시에 알림을 보낼 파일 디스크립터이다.</dd>
</dl>

## RETURN VALUE

성공 시 `io_submit()`은 제출된 `iocb` 개수를 반환한다. (`nr`보다 작을 수도 있으며 `nr`이 0이면 0일 수도 있다.) 실패 반환에 대해선 NOTES를 보라.

## ERRORS

<dl>
<dt><code>EAGAIN</code></dt>
<dd><code>iocb</code>를 큐에 넣기 위한 자원이 충분치 않다.</dd>
<dt><code>EBADF</code></dt>
<dd>첫 번째 <code>iocb</code>에 지정한 파일 디스크립터가 유효하지 않다.</dd>
<dt><code>EFAULT</code></dt>
<dd>한 자료 구조가 유효하지 않은 데이터를 가리키고 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>ctx_id</code>로 지정한 AIO 문맥이 유효하지 않다. <code>nr</code>이 0보다 작다. <code>*iocbpp[0]</code>에 있는 <code>iocb</code>가 제대로 초기화 되어 있지 않다. 지정한 동작이 <code>iocb</code>의 파일 디스크립터에 유효하지 않다. <code>aio_reqprio</code> 필드의 값이 유효하지 않다.</dd>
<dt><code>ENOSYS</code></dt>
<dd>이 아키텍처에 <code>io_submit()</code>이 구현돼 있지 않다.</dd>
<dt><code>EPERM</code></dt>
<dd><code>aio_reqprio</code> 필드가 클래스 <code>IOPRIO_CLASS_RT</code>로 설정돼 있지만 제출 동작을 하는 문맥에 <code>CAP_SYS_ADMIN</code> 역능이 없다.</dd>
</dl>

## VERSIONS

리눅스 2.5에서 비동기 I/O 시스템 호출들이 처음 등장했다.

## CONFORMING TO

`io_submit()`은 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출할 수도 있다. 하지만 아마 그보다는 `libaio`에서 제공하는 `io_submit()` 래퍼 함수를 쓰고 싶을 것이다.

참고로 `libaio` 래퍼 함수에서는 `ctx_id` 인자에 다른 타입(`io_context_t`)을 쓴다. 또한 `libaio` 래퍼에서는 일반적인 C 라이브러리 오류 표시 관행을 따르지 않는다. 즉 오류 시에 음수 오류 번호를 (ERRORS에 나열된 값들 중 하나의 음수 값을) 반환한다. <tt>[[syscall(2)]]</tt>을 통해 시스템 호출을 부르는 경우에는 반환 값이 일반적인 오류 표시 관행을 따른다. 즉 -1이고 `errno`에 오류를 나타내는 (양수) 값이 설정된다.

## SEE ALSO

<tt>[[io_cancel(2)]]</tt>, <tt>[[io_destroy(2)]]</tt>, <tt>[[io_getevents(2)]]</tt>, <tt>[[io_setup(2)]]</tt>, <tt>[[aio(7)]]</tt>

----

2018-04-30
