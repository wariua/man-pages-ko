## NAME

sigevent - 비동기 루틴으로부터의 알림을 위한 구조체

## SYNOPSIS

```c
#include <signal.h>

union sigval {            /* 알림과 함께 전달되는 데이터 */
    int     sival_int;    /* 정수 값 */
    void   *sival_ptr;    /* 포인터 값 */
};

struct sigevent {
    int    sigev_notify;  /* 알림 방법 */
    int    sigev_signo;   /* 알림 시그널 */
    union sigval sigev_value;
                          /* 알림과 함께 전달되는 데이터 */
    void (*sigev_notify_function) (union sigval);
                          /* 스레드 알림에 쓰는 함수
                             (SIGEV_THREAD) */
    void  *sigev_notify_attributes;
                          /* 알림 스레드의 속성
                             (SIGEV_THREAD) */
    pid_t  sigev_notify_thread_id;
                          /* 신호를 받을 스레드의 ID
                             (SIGEV_THREAD_ID), 리눅스 전용 */
};
```

## DESCRIPTION

다양한 API에서 `sigevent` 구조체를 사용하여 프로세스가 이벤트 (가령 비동기 요청 완료, 타이머 만료, 메시지 도착) 알림을 받을 방식을 기술한다.

SYNOPSIS에서 보여준 정의는 대략적인 것이다. `sigevent` 구조체의 일부 필드들이 공용체의 일부로 정의되어 있을 수도 있다. 프로그램에서는 `sigev_notify`에 지정한 값에 관련된 필드만을 사용해야 한다.

`sigev_notify` 필드는 알림을 어떻게 수행할지 지정한다. 다음 값들 중 하나일 수 있다.

`SIGEV_NONE`
:   "null" 알림. 사건이 발생했을 때 아무것도 하지 않는다.

`SIGEV_SIGNAL`
:   `sigev_signo`에 지정된 시그널을 보내서 프로세스에게 알린다.

    <tt>[[sigaction(2)]]</tt> `SA_SIGINFO` 플래그를 써서 등록한 시그널 핸들러로 시그널을 잡는 경우에는 핸들러 두 번째 인자로 전달되는 `siginfo_t` 구조체에 다음 필드들을 설정한다.

    `si_code`
    :   알림을 보내는 API에 따라 달라지는 어떤 값으로 이 필드를 설정한다.

    `si_signo`
    :   시그널 번호로 (즉, `sigev_signo`와 값은 값으로) 이 필드를 설정한다.

    `si_value`
    :   `sigev_value`에서 지정한 값으로 이 필드를 설정한다.

    API에 따라서 `siginfo_t` 구조체 내의 다른 필드들까지 설정할 수도 있다.

    <tt>[[sigwaitinfo(2)]]</tt>로 시그널을 받는 경우에도 같은 정보를 사용할 수 있다.

`SIGEV_THREAD`
:   `sigev_notify_function`을 새 스레드의 시작 함수"인 것처럼" 호출하여 프로세스에게 알린다. (이를 구현할 수 있는 방안들 중에는 각 타이머 알림마다 새 스레드를 생성하는 것도 있고, 한 스레드를 만들어서 모든 알림을 받는 것도 있다.) `sigev_value`를 유일한 인자로 해서 함수를 호출한다. `sigev_notify_attributes`는 NULL이 아니라면 새 스레드의 속성을 정의하는 `pthread_attr_t` 구조체(<tt>[[pthread_attr_init(3)]]</tt> 참고)를 가리켜야 한다.

`SIGEV_THREAD_ID` (리눅스 한정)
:   현재 POSIX 타이머에서만 사용함. <tt>[[timer_create(2)]]</tt> 참고.

## SEE ALSO

<tt>[[timer_create(2)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[getaddrinfo_a(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[mq_notify(3)]]</tt>, <tt>[[aio(7)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
