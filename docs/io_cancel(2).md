## NAME

io_cancel - 미처리 비동기 I/O 동작 취소하기

## SYNOPSIS

```c
#include <linux/aio_abi.h>          /* 필요한 타입 정의 */

int io_cancel(aio_context_t ctx_id, struct iocb *iocb,
              struct io_event *result);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

*주의*: 이 페이지에선 리눅스 시스템 호출 인터페이스를 설명한다. `libaio`에서 제공하는 래퍼 함수에서는 `ctx_id` 인자에 다른 타입을 쓴다. NOTES 참고.

`io_cancel()` 시스템 호출은 앞서 <tt>[[io_submit(2)]]</tt>으로 제출한 비동기 I/O 동작을 취소하려고 시도한다. `iocb` 인자는 취소할 동작을 나타내며 `ctx_id` 인자는 동작을 제출했던 AIO 문맥이다. 동작을 성공적으로 취소한 경우 이벤트가 완료 큐로 들어가지 않고 `result`가 가리키는 메모리로 복사된다.

## RETURN VALUE

성공 시 `io_cancel()`은 0을 반환한다. 실패 반환에 대해선 NOTES를 보라.

## ERRORS

`EAGAIN`
:   지정한 `iocb`가 취소되지 않았다.

`EFAULT`
:   한 자료 구조가 유효하지 않은 데이터를 가리키고 있다.

`EINVAL`
:   `ctx_id`로 지정한 AIO 문맥이 유효하지 않다.

`ENOSYS`
:   이 아키텍처에 `io_cancel()`이 구현돼 있지 않다.

## VERSIONS

리눅스 2.5에서 비동기 I/O 시스템 호출들이 처음 등장했다.

## CONFORMING TO

`io_cancel()`은 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출할 수도 있다. 하지만 아마 그보다는 `libaio`에서 제공하는 `io_cancel()` 래퍼 함수를 쓰고 싶을 것이다.

참고로 `libaio` 래퍼 함수에서는 `ctx_id` 인자에 다른 타입(`io_context_t`)을 쓴다. 또한 `libaio` 래퍼에서는 일반적인 C 라이브러리 오류 표시 관행을 따르지 않는다. 즉 오류 시에 음수 오류 번호를 (ERRORS에 나열된 값들 중 하나의 음수 값을) 반환한다. <tt>[[syscall(2)]]</tt>을 통해 시스템 호출을 부르는 경우에는 반환 값이 일반적인 오류 표시 관행을 따른다. 즉 -1이고 `errno`에 오류를 나타내는 (양수) 값이 설정된다.

## SEE ALSO

<tt>[[io_destroy(2)]]</tt>, <tt>[[io_getevents(2)]]</tt>, <tt>[[io_setup(2)]]</tt>, <tt>[[io_submit(2)]]</tt>, <tt>[[aio(7)]]</tt>

----

2021-03-22
