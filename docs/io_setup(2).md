## NAME

io_setup - 비동기 I/O 문맥 만들기

## SYNOPSIS

```c
#include <linux/aio_abi.h>          /* 필요한 타입 정의 */

long io_setup(unsigned int nr_events, aio_context_t *ctx_idp);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

*주의*: 이 페이지에선 리눅스 시스템 호출 인터페이스를 설명한다. `libaio`에서 제공하는 래퍼 함수에서는 `ctx_idp` 인자에 다른 타입을 쓴다. NOTES 참고.

`io_setup()` 시스템 호출은 `nr_events` 개 동작을 동시에 처리하기에 적합한 비동기 I/O 문맥을 생성한다. `ctx_idp` 인자가 이미 존재하는 AIO 문맥을 가리켜선 안 되며 호출 전에 문맥을 0으로 초기화 해야 한다. AIO 문맥을 성공적으로 생성했을 때 `*ctx_idp`에 새 핸들이 들어간다.

## RETURN VALUE

성공 시 `io_setup()`은 0을 반환한다. 실패 반환에 대해선 NOTES를 보라.

## ERRORS

`EAGAIN`
:   지정한 `nr_events`가 `/proc/sys/fs/aio-max-nr`에 규정된 가용 이벤트 제한을 초과한다. (<tt>[[proc(5)]]</tt> 참고.)

`EFAULT`
:   `ctx_idp`에 유효하지 않은 포인터를 전달했다.

`EINVAL`
:   `ctx_idp`가 초기화 되지 않았다. 지정한 `nr_events`가 내부 제한을 초과한다. `nr_events`가 0보다 커야 한다.

`ENOMEM`
:   사용 가능한 커널 자원이 충분치 않다.

`ENOSYS`
:   이 아키텍처에 `io_setup()`이 구현돼 있지 않다.

## VERSIONS

리눅스 2.5에서 비동기 I/O 시스템 호출들이 처음 등장했다.

## CONFORMING TO

`io_setup()`은 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출할 수도 있다. 하지만 아마 그보다는 `libaio`에서 제공하는 `io_setup()` 래퍼 함수를 쓰고 싶을 것이다.

참고로 `libaio` 래퍼 함수에서는 `ctx_idp` 인자에 다른 타입(`io_context_t *`)을 쓴다. 또한 `libaio` 래퍼에서는 일반적인 C 라이브러리 오류 표시 관행을 따르지 않는다. 즉 오류 시에 음수 오류 번호를 (ERRORS에 나열된 값들 중 하나의 음수 값을) 반환한다. <tt>[[syscall(2)]]</tt>을 통해 시스템 호출을 부르는 경우에는 반환 값이 일반적인 오류 표시 관행을 따른다. 즉 -1이고 `errno`에 오류를 나타내는 (양수) 값이 설정된다.

## SEE ALSO

<tt>[[io_cancel(2)]]</tt>, <tt>[[io_destroy(2)]]</tt>, <tt>[[io_getevents(2)]]</tt>, <tt>[[io_submit(2)]]</tt>, <tt>[[aio(7)]]</tt>

----

2021-03-22
