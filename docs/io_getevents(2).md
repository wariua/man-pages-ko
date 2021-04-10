## NAME

io_getevents - 완료 큐의 비동기 I/O 이벤트 읽어 들이기

## SYNOPSIS

```c
#include <linux/aio_abi.h>          /* 필요한 타입 정의 */
#include <linux/time.h>             /* 'struct timespec' 정의 */

int io_getevents(aio_context_t ctx_id, long min_nr, long nr,
                 struct io_event *events, struct timespec *timeout);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`io_getevents()` 시스템 호출은 `ctx_id`로 지정한 AIO 문맥의 완료 큐에서 최소 `min_nr` 개에서 최대 `nr` 개까지 이벤트를 읽어 들이려고 시도한다.

`timeout`은 이벤트를 기다릴 시간 길이를 나타내며 다음 구조체로 상대적 타임아웃으로 지정한다.

```c
struct timespec {
    time_t tv_sec;      /* 초 */
    long   tv_nsec;     /* 나노초 [0 .. 999999999] */
};
```

지정 시간을 시스템 클럭 해상도에 따라 올림 하며 이르게 만료하지 않는다고 보장된다.

`timeout`을 NULL로 지정하는 것은 최소 `min_nr` 개 이벤트를 얻을 때까지 무한정 블록 하라는 뜻이다.

## RETURN VALUE

성공 시 `io_getevents()`는 읽어 들인 이벤트 수를 반환한다. `timeout`이 만료된 경우에는 0이나 `min_nr`보다 작은 어떤 값일 수 있다. 또한 호출이 시그널 핸들러에 의해 중단된 경우에는 `min_nr`보다 작은 0 아닌 어떤 값일 수 있다.

실패 반환에 대해선 NOTES를 보라.

## ERRORS

`EFAULT`
:   `events`나 `timeout`이 유효하지 않은 포인터이다.

`EINTR`
:   시그널 핸들러에 의해 중단되었다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `ctx_id`가 유효하지 않다. `min_nr`이 범위 밖이거나 `nr`이 범위 밖이다.

`ENOSYS`
:   이 아키텍처에 `io_getevents()`가 구현돼 있지 않다.

## VERSIONS

리눅스 2.5에서 비동기 I/O 시스템 호출들이 처음 등장했다.

## CONFORMING TO

`io_getevents()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출할 수도 있다. 하지만 아마 그보다는 `libaio`에서 제공하는 `io_getevents()` 래퍼 함수를 쓰고 싶을 것이다.

참고로 `libaio` 래퍼 함수에서는 `ctx_id` 인자에 다른 타입(`io_context_t`)을 쓴다. 또한 `libaio` 래퍼에서는 일반적인 C 라이브러리 오류 표시 관행을 따르지 않는다. 즉 오류 시에 음수 오류 번호를 (ERRORS에 나열된 값들 중 하나의 음수 값을) 반환한다. <tt>[[syscall(2)]]</tt>을 통해 시스템 호출을 부르는 경우에는 반환 값이 일반적인 오류 표시 관행을 따른다. 즉 -1이고 `errno`에 오류를 나타내는 (양수) 값이 설정된다.

## BUGS

유효하지 않은 `ctx_id`가 `EINVAL` 오류를 생성하는 대신 세그먼테이션 폴트를 유발할 수 있다.

## SEE ALSO

<tt>[[io_cancel(2)]]</tt>, <tt>[[io_destroy(2)]]</tt>, <tt>[[io_setup(2)]]</tt>, <tt>[[io_submit(2)]]</tt>, <tt>[[aio(7)]]</tt>, <tt>[[time(7)]]</tt>

----

2017-09-15
