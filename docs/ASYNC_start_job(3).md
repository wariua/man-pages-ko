## NAME

ASYNC_get_wait_ctx, ASYNC_init_thread, ASYNC_cleanup_thread, ASYNC_start_job, ASYNC_pause_job, ASYNC_get_current_job, ASYNC_block_pause, ASYNC_unblock_pause, ASYNC_is_capable - 비동기 작업 관리 함수들

## SYNOPSIS

```c
#include <openssl/async.h>

int ASYNC_init_thread(size_t max_size, size_t init_size);
void ASYNC_cleanup_thread(void);

int ASYNC_start_job(ASYNC_JOB **job, ASYNC_WAIT_CTX *ctx, int *ret,
                    int (*func)(void *), void *args, size_t size);
int ASYNC_pause_job(void);

ASYNC_JOB *ASYNC_get_current_job(void);
ASYNC_WAIT_CTX *ASYNC_get_wait_ctx(ASYNC_JOB *job);
void ASYNC_block_pause(void);
void ASYNC_unblock_pause(void);

int ASYNC_is_capable(void);
```

## DESCRIPTION

OpenSSL에서는 ASYNC_JOB을 통해 비동기 기능을 구현한다. ASYNC_JOB은 실행을 시작할 수 있는 코드를 나타내며 어떤 이벤트가 발생할 때까지 실행된다. 그 시점에서 코드를 멈출 수 있으며, 그러면 어떤 후속 이벤트에 따라 작업을 재개할 수 있게 되기 전까지는 제어가 사용자 코드로 돌아간다.

ASYNC_JOB 생성은 꽤 비용이 큰 동작이다. 따라서 효율성을 위해 작업들을 미리 만들어 두고 여러 번 재사용할 수 있다. 풀에 유지하다가 필요해질 때 풀에서 꺼내서 사용하고, 작업이 끝나면 풀로 돌려준다. 사용자 응용이 다중 스레드 방식이라면 스레드마다 `ASYNC_init_thread()`를 호출해서 비동기 작업들을 초기화 할 수 있다. 사용자 코드가 끝나기 전에 스레드별 자원을 정리할 필요가 있다. 보통은 자동으로 이뤄지지만 (`OPENSSL_init_crypto(3)` 참고) `ASYNC_cleanup_thread()`를 써서 명시적으로 개시할 수도 있다. `ASYNC_cleanup_thread()` 호출 때 그 스레드에 미완료인 비동기 작업이 있어선 안 된다. 혹시라도 있으면 메모리 누수를 유발하게 된다.

`max_size` 인자는 풀에 유지할 ASYNC_JOB의 수를 제한한다. `max_size`를 0으로 설정하면 상한을 두지 않는다. ASYNC_JOB이 필요한데 풀 안에 남아 있는 게 없는 경우 그 풀에서 관리하는 ASYNC_JOB 총개수가 `max_size`를 초과하지 않는 한에서 새 항목을 자동으로 생성하게 된다. 그리고 풀을 처음 초기화 할 때 ASYNC_JOB `init_size` 개를 즉시 생성한다. 풀을 처음 쓰기 전에 `ASYNC_init_thread()`를 호출하지 않으면 `max_size`를 0(상한 없음)으로 하고 `init_size`를 0(미리 만드는 ASYNC_JOB 없음)으로 해서 자동으로 호출한다.

`ASYNC_start_job()` 함수를 호출해서 비동기 작업을 시작한다. 처음에는 `*job`이 NULL이어야 한다. `ctx`는 <tt>[[ASYNC_WAIT_CTX_new(3)]]</tt> 함수를 통해 만든 ASYNC_WAIT_CTX 객체를 가리켜야 한다. `ret`는 작업 완료 시 비동기 함수의 반환 값을 저장할 위치를 가리켜야 한다. `func`는 비동기적으로 시작할 함수를 나타낸다. 작업 시작 시 `args`가 가리키는 크기 `size`인 데이터를 복사해서 `func` 인자로 전달한다. `ASYNC_start_job()`은 다음 값들 중 하나를 반환한다.

`ASYNC_ERR`
:   작업 시작 시도 중 오류가 발생했다. 자세한 내용은 OpenSSL 오류 큐(가령 `ERR_print_errors(3)`)를 확인하라.

`ASYNC_NO_JOBS`
:   풀에 현재 사용 가능한 작업이 없다. 이 호출을 이후에 다시 시도할 수 있다.

`ASYNC_PAUSE`
:   작업이 성공적으로 시작됐지만 완료되기 전에 "정지"되었다. (아래 `ASYNC_pause_job()` 참고.) 작업에 대한 핸들이 `*job`에 들어간다. (원한다면) 다른 작업을 수행하고 이후에 작업을 재시작할 수 있다. 작업을 재시작하려면 `*job`에 작업 핸들을 줘서 다시 `ASYNC_start_job()`을 호출하면 된다. 작업을 재시작할 때 `func`, `args`, `size` 매개변수는 무시된다. 작업을 재시작할 때는 **반드시** 그 작업을 처음 시작했던 스레드에서 `ASYNC_start_job()`을 호출해야 한다.

`ASYNC_FINISH`
:   작업이 끝났다. `*job`이 NULL이 되고 `func`의 반환 값이 `*ret`에 들어간다.

어느 시점이든 스레드별로 최대 1개 작업이 활동적으로 실행 중일 수 있다. (그리고 여러 작업이 중지돼 있을 수 있다.) `ASYNC_get_current_job()`을 이용해 현재 실행 중인 ASYNC_JOB에 대한 포인터를 얻을 수 있다. 현재 실행 중인 작업이 없으면 NULL을 반환한다.

작업 문맥 내에서 `ASYNC_pause_job()`을 실행하면 (즉 `ASYNC_start_job()` 인자로 준 함수 `func`에서 직간접적으로 호출하면) `ASYNC_start_job()`을 호출한 응용으로 즉시 제어가 되돌아가서 `ASYNC_start_job()` 호출이 `ASYNC_PAUSE`를 반환한다. 이후에 `*job` 매개변수에 있는 해당 ASYNC_JOB을 줘서 `ASYNC_start_job()`을 호출하면 `ASYNC_pause_job()` 호출 지점에서 실행이 재개된다. 작업 문맥 내에 있지 않은 상태에서 `ASYNC_pause_job()`을 호출하면 아무 동작도 이뤄지지 않으며 `ASYNC_pause_job()`이 즉시 반환한다.

`ASYNC_get_wait_ctx()`를 이용해 `job`의 ASYNC_WAIT_CTX에 대한 포인터를 얻을 수 있다. ASYNC_WAIT_CTX에는 "대기" 파일 디스크립터를 연계할 수 있다. 응용에서 select나 poll 같은 시스템 함수 호출을 이용해 파일 디스크립터가 "읽기" 준비가 되기를 기다릴 수 있다. ("읽기" 준비가 되는 것은 작업을 재개해야 한다는 표시이다.) 제공받는 파일 디스크립터가 없다면 응용에서 주기적으로 작업을 재시작하는 방식으로 작업이 실행을 이어갈 준비가 되어 있는지 확인해 봐야 할 것이다.

일반적인 사용례로는 비동기 지원 엔진이 있을 수 있다. 일단 사용자 코드에서 암호 연산을 개시한다. 엔진이 그 연산을 비동기적으로 개시하고서 <tt>[[ASYNC_WAIT_CTX_set_wait_fd(3)]]</tt>에 이어 `ASYNC_pause_job()`을 호출해서 사용자 코드로 제어를 돌려준다. 그러면 사용자 코드에서는 다른 작업을 수행하거나 그 대기 파일 디스크립터에 "select" 내지 기타 유사 함수를 호출해서 작업이 준비 상태가 되기를 기다릴 수 있다. 그리고 엔진에서는 대기 파일 디스크립터를 "읽기 가능"으로 만들어 작업을 재개해야 한다는 신호를 줄 수 있다. 실행이 재개되면 엔진에서는 대기 파일 디스크립터에서 깨우는 신호를 없애야 한다.

`ASYNC_block_pause()` 함수는 현재 활성 작업이 중지되는 것을 막는다. 이후 `ASYNC_unblock_pause()`를 호출할 때까지 차단이 유지된다. 이 함수들은 중복 호출이 가능하다. 가령 `ASYNC_block_pause()`를 두 번 호출했으면 `ASYNC_unblock_pause()`를 두 번 호출해야 중지가 재활성화된다. 현재 활성 작업이 없는 상태에서 이 함수들을 호출하는 건 아무 효과가 없다. 이 기능은 교착 시나리오를 피하는 데 도움이 될 수 있다. 예를 들어 어떤 ASYNC_JOB 실행 중에 응용에서 락을 획득한다고 하자. 그러고서 어떤 암호 함수를 호출하고 거기서 `ASYNC_pause_job()`을 부른다. 이렇게 하면 ASYNC_JOB을 생성했던 코드로 제어가 되돌아간다. 그런 다음 그 코드에서 원래 작업을 재개하기 전에 그 동일 락을 획득하려고 하면 교착이 발생할 수 있다. 락을 잡자마자 `ASYNC_block_pause()`를 호출하고 락을 놓기 바로 전에 `ASYNC_unblock_pause()`를 호출하면 이런 경우가 생길 수 없다.

일부 플랫폼은 비동기 동작을 지원하지 못한다. `ASYNC_is_capable()` 함수를 이용해 현재 플랫폼이 비동기 동작을 지원하는지 여부를 탐지할 수 있다.

## RETURN VALUES

`ASYNC_init_thread()`는 성공 시 1을 반환하고 아니면 0을 반환한다.

`ASYNC_start_job()`은 위에 설명한 대로 `ASYNC_ERR`, `ASYNC_NO_JOBS`, `ASYNC_PAUSE`, `ASYNC_FINISH` 중 하나를 반환한다.

`ASYNC_pause_job()`은 오류 시 0을 반환하고 성공 시 1을 반환한다. ASYNC_JOB 문맥 안이 아닐 때 호출하면 성공으로 쳐서 1을 반환한다.

`ASYNC_get_current_job()`은 현재 실행 중인 ASYNC_JOB에 대한 포인터를 반환하고 작업 문맥 안이 아니면 NULL을 반환한다.

`ASYNC_get_wait_ctx()`는 작업의 ASYNC_WAIT_CTX에 대한 포인터를 반환한다.

`ASYNC_is_capable()`은 현재 플랫폼이 비동기를 지원하면 1을 반환하고 아니면 0을 반환한다.

## NOTES

윈도우 플랫폼에서는 `openssl/async.h` 헤더에 필요한 몇 가지 타입들이 보통 `windows.h`를 포함시켜야 사용 가능해진다. 그런데 가장 먼저 포함시키는 헤더들 중 하나인 `windows.h`를 언제 포함시킬지를 응용 개발자가 통제할 수 있어야 하는 경우가 많다. 따라서 `async.h`에 앞서 `windows.h`를 포함시키는 것을 응용 개발자의 책임으로 규정한다.

## EXAMPLE

다음 예는 핵심 비동기 API 대부분의 사용 방식을 보여 준다.

```c
#ifdef _WIN32
# include <windows.h>
#endif
#include <stdio.h>
#include <unistd.h>
#include <openssl/async.h>
#include <openssl/crypto.h>

int unique = 0;

void cleanup(ASYNC_WAIT_CTX *ctx, const void *key, OSSL_ASYNC_FD r, void *vw)
{
    OSSL_ASYNC_FD *w = (OSSL_ASYNC_FD *)vw;

    close(r);
    close(*w);
    OPENSSL_free(w);
}

int jobfunc(void *arg)
{
    ASYNC_JOB *currjob;
    unsigned char *msg;
    int pipefds[2] = {0, 0};
    OSSL_ASYNC_FD *wptr;
    char buf = 'X';

    currjob = ASYNC_get_current_job();
    if (currjob != NULL) {
        printf("Executing within a job\n");
    } else {
        printf("Not executing within a job - should not happen\n");
        return 0;
    }

    msg = (unsigned char *)arg;
    printf("Passed in message is: %s\n", msg);

    if (pipe(pipefds) != 0) {
        printf("Failed to create pipe\n");
        return 0;
    }
    wptr = OPENSSL_malloc(sizeof(OSSL_ASYNC_FD));
    if (wptr == NULL) {
        printf("Failed to malloc\n");
        return 0;
    }
    *wptr = pipefds[1];
    ASYNC_WAIT_CTX_set_wait_fd(ASYNC_get_wait_ctx(currjob), &unique,
                               pipefds[0], wptr, cleanup);

    /*
     * 보통은 얼마 후 어떤 외부 이벤트에 의해 이렇게 되며
     * 이건 시연용이다. ASYNC_pause_job()을 통해 메인으로
     * 반환한 후에 바로 깨어날 준비가 됐다는 신호가 간다.
     */
    write(pipefds[1], &buf, 1);

    /* 제어를 메인으로 반환 */
    ASYNC_pause_job();

    /* 기상 신호 비우기 */
    read(pipefds[0], &buf, 1);

    printf ("Resumed the job after a pause\n");

    return 1;
}

int main(void)
{
    ASYNC_JOB *job = NULL;
    ASYNC_WAIT_CTX *ctx = NULL;
    int ret;
    OSSL_ASYNC_FD waitfd;
    fd_set waitfdset;
    size_t numfds;
    unsigned char msg[13] = "Hello world!";

    printf("Starting...\n");

    ctx = ASYNC_WAIT_CTX_new();
    if (ctx == NULL) {
        printf("Failed to create ASYNC_WAIT_CTX\n");
        abort();
    }

    for (;;) {
        switch (ASYNC_start_job(&job, ctx, &ret, jobfunc, msg, sizeof(msg))) {
        case ASYNC_ERR:
        case ASYNC_NO_JOBS:
            printf("An error occurred\n");
            goto end;
        case ASYNC_PAUSE:
            printf("Job was paused\n");
            break;
        case ASYNC_FINISH:
            printf("Job finished with return value %d\n", ret);
            goto end;
        }

        /* 작업이 깨어나기를 기다리기 */
        printf("Waiting for the job to be woken up\n");

        if (!ASYNC_WAIT_CTX_get_all_fds(ctx, NULL, &numfds)
                || numfds > 1) {
            printf("Unexpected number of fds\n");
            abort();
        }
        ASYNC_WAIT_CTX_get_all_fds(ctx, &waitfd, &numfds);
        FD_ZERO(&waitfdset);
        FD_SET(waitfd, &waitfdset);
        select(waitfd + 1, &waitfdset, NULL, NULL, NULL);
    }

end:
    ASYNC_WAIT_CTX_free(ctx);
    printf("Finishing\n");

    return 0;
}
```

위 예시 프로그램 실행 시의 예상 출력은 다음과 같다.

```text
Starting...
Executing within a job
Passed in message is: Hello world!
Job was paused
Waiting for the job to be woken up
Resumed the job after a pause
Job finished with return value 1
Finishing
```

## SEE ALSO

`crypto(7)`, `ERR_print_errors(3)`

## HISTORY

OpenSSL 1.1.0에서 `ASYNC_init_thread()`, `ASYNC_cleanup_thread()`, `ASYNC_start_job()`, `ASYNC_pause_job()`, `ASYNC_get_current_job()`, `ASYNC_get_wait_ctx()`, `ASYNC_block_pause()`, `ASYNC_unblock_pause()`, `ASYNC_is_capable()`이 처음 추가되었다.

## COPYRIGHT

Copyright 2015-2016 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use this file except in compliance with the License.  You can obtain a copy in the file LICENSE in the source distribution or at <https://www.openssl.org/source/license.html>.

----

2017-12-31
