## NAME

pthread_cancel - 스레드에게 취소 요청 보내기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_cancel(pthread_t thread);
```

`-pthread`로 링크.

## DESCRIPTION

`pthread_cancel()` 함수는 스레드 `thread`에게 취소 요청을 보낸다. 대상 스레드가 취소 요청에 반응할지, 그렇다면 언제일지는 그 스레드의 통제 하에 있는 두 가지 속성, 즉 취소 가능성 *상태*와 *유형*에 달려 있다.

스레드의 취소 가능성 상태는 <tt>[[pthread_setcancelstate(3)]]</tt>로 결정하며 *활성*(새 스레드의 기본값)이거나 *비활성*일 수 있다. 스레드에서 취소를 비활성화했으면 다시 취소를 활성화할 때까지 취소 요청이 큐에 남아 있는다. 스레드에서 취소를 활성화했으면 취소 가능성 유형이 취소 발생 시점을 결정한다.

스레드의 취소 유형은 <tt>[[pthread_setcanceltype(3)]]</tt>으로 결정하며 *비동기*나 *연기*(새 스레드의 기본값)일 수 있다. 비동기 취소는 스레드가 언제든 (일반적으로 즉시이지만 시스템에서 보장하지 않음) 취소될 수 있다는 뜻이다. 연기 취소는 스레드에서 *취소점*인 함수를 호출할 때까지 취소가 연기된다는 뜻이다. 취소점이거나 취소점일 수도 있는 함수들의 목록이 <tt>[[pthreads(7)]]</tt>에 있다.

취소 요청에 반응할 때 `thread`에서는 다음 단계들이 (차례로) 이뤄진다.

1. 취소 정리 핸들러를 (집어넣은 순서 반대로) 꺼내서 호출한다. (<tt>[[pthread_cleanup_push(3)]]</tt>) 참고.)

2. 스레드별 데이터 소멸자들이 명세 안 된 순서로 호출된다. (<tt>[[pthread_key_create(3)]]</tt> 참고.)

3. 스레드가 종료된다. (<tt>[[pthread_exit(3)]]</tt> 참고.)

위 단계들은 `pthread_cancel()` 호출과 비동기적으로 이뤄진다. `pthread_cancel()`의 반환 상태는 취소 요청이 성공적으로 큐에 들어갔는지를 호출자에게 알려 줄 뿐이다.

취소된 스레드가 종료한 후에 <tt>[[pthread_join(3)]]</tt>으로 그 스레드와 합류하면 스레드 종료 상태로 `PTHREAD_CANCELED`를 얻는다. (스레드와 합류하는 것은 취소가 완료됐는지 알 수 있는 유일한 방법이다.)

## RETURN VALUE

성공 시 `pthread_cancel()`은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`ESRCH`
:   ID가 `thread`인 스레드를 찾을 수 없다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_cancel()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

리눅스에서는 시그널을 이용해 취소를 구현한다. NPTL 스레딩 구현에서는 첫 번째 실시간 시그널(즉 시그널 32)을 이 용도에 쓴다. LinuxThreads에서는 실시간 시그널이 사용 가능하면 두 번째 실시간 시그널을 쓰고 아니면 `SIGUSR2`를 쓴다.

## EXAMPLE

아래 프로그램에서는 스레드를 생성했다가 취소한다. 메인 스레드가 취소된 스레드와 합류해서 종료 상태가 `PTHREAD_CANCELED`인지 확인한다. 다음 셸 세션은 프로그램 실행 시 어떻게 되는지 보여 준다.

```text
$ ./a.out
thread_func(): started; cancellation disabled
main(): sending cancellation request
thread_func(): about to enable cancellation
main(): thread was canceled
```

### 프로그램 소스

```c
#include <pthread.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <unistd.h>

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

static void *
thread_func(void *ignored_argument)
{
    int s;

    /* 취소를 잠시 비활성화해서 취소 요청에
       즉시 반응하지 않도록 하기 */

    s = pthread_setcancelstate(PTHREAD_CANCEL_DISABLE, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_setcancelstate");

    printf("thread_func(): started; cancellation disabled\n");
    sleep(5);
    printf("thread_func(): about to enable cancellation\n");

    s = pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_setcancelstate");

    /* sleep()은 취소점임 */

    sleep(1000);        /* 잠들어 있는 동안 취소됨 */

    /* 절대 여기로 안 옴 */

    printf("thread_func(): not canceled!\n");
    return NULL;
}

int
main(void)
{
    pthread_t thr;
    void *res;
    int s;

    /* 스레드 시작하고 취소 요청을 보내기 */

    s = pthread_create(&thr, NULL, &thread_func, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_create");

    sleep(2);           /* 스레드에게 시작할 시간 주기 */

    printf("main(): sending cancellation request\n");
    s = pthread_cancel(thr);
    if (s != 0)
        handle_error_en(s, "pthread_cancel");

    /* 스레드와 합류해서 종료 상태 보기 */

    s = pthread_join(thr, &res);
    if (s != 0)
        handle_error_en(s, "pthread_join");

    if (res == PTHREAD_CANCELED)
        printf("main(): thread was canceled\n");
    else
        printf("main(): thread wasn't canceled (shouldn't happen!)\n");
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[pthread_cleanup_push(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_exit(3)]]</tt>, <tt>[[pthread_join(3)]]</tt>, <tt>[[pthread_key_create(3)]]</tt>, <tt>[[pthread_setcancelstate(3)]]</tt>, <tt>[[pthread_setcanceltype(3)]]</tt>, <tt>[[pthread_testcancel(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2019-03-06
