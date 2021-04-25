## NAME

pthread_cleanup_push, pthread_cleanup_pop - 스레드 취소 정리 핸들러 집어넣기와 꺼내기

## SYNOPSIS

```c
#include <pthread.h>

void pthread_cleanup_push(void (*routine)(void *), void *arg);
void pthread_cleanup_pop(int execute);
```

`-pthread`로 링크.

## DESCRIPTION

이 함수들은 호출 스레드의 스레드 취소 정리 핸들러 스택을 조작한다. 정리 핸들러는 스레드가 취소될 때 (또는 아래에서 설명하는 여러 다른 경우에) 자동으로 실행되는 함수이다. 예를 들어 핸들러에서 뮤텍스를 놓아서 프로세스 내 다른 스레드들에게 사용 가능하게 할 수 있을 것이다.

`pthread_cleanup_push()` 함수는 정리 핸들러 스택 상단에 `routine`을 집어넣는다. 이후 `routine`이 호출될 때 `arg`를 인자로 받게 된다.

`pthread_cleanup_pop()` 함수는 정리 핸들러 스택 상단의 루틴을 제거하며, `execute`가 0이 아니면 그 루틴을 실행한다.

다음 경우에 취소 정리 핸들러들을 스택에서 꺼내서 실행한다.

1. 스레드가 취소될 때 스택에 있는 정리 핸들러들을 스택에 집어넣은 순서 반대로 꺼내서 실행한다.

2. 스레드가 <tt>[[pthread_exit(3)]]</tt> 호출로 종료할 때 앞 항목 설명처럼 모든 정리 핸들러들을 실행한다. (스레드 시작 함수에서 `return`을 수행해서 스레드가 종료하는 경우에는 정리 핸들러가 호출되지 *않는다*.)

3. 스레드가 0 아닌 `execute` 인자로 `pthread_cleanup_pop()`을 호출할 때 최상단의 정리 핸들러를 꺼내서 실행한다.

POSIX.1에서는 `pthread_cleanup_push()`와 `pthread_cleanup_pop()`을 각각 '`{`'와 '`}`'를 담은 텍스트로 확장되는 매크로로 구현하는 것을 허용한다. 따라서 이 함수들의 호출이 같은 함수 안에 있도록, 그리고 같은 문법적 내포 단계에 있도록 해야 한다. (달리 말하면 특정 코드 구간을 의 실행하는 동안만 정리 핸들러가 설정되어 있다.)

<tt>[[setjmp(3)]]</tt>(<tt>[[sigsetjmp(3)]]</tt>)로 점프 버퍼를 채운 이후에 짝을 이루는 호출 없이 `pthread_cleanup_push()`나 `pthread_cleanup_pop()`을 호출한 상태에서 <tt>[[longjmp(3)]]</tt>(<tt>[[siglongjmp(3)]]</tt>)를 호출하면 규정되어 있지 않은 결과가 나온다. 마찬가지로 정리 핸들러 내에서 <tt>[[setjmp(3)]]</tt>(<tt>[[sigsetjmp(3)]]</tt>)로 점프 버퍼를 채운 경우가 아니라면 핸들러 내에서 <tt>[[longjmp(3)]]</tt>(<tt>[[siglongjmp(3)]]</tt>) 호출 시 규정되어 있지 않은 결과가 나온다.

## RETURN VALUE

이 함수들은 값을 반환하지 않는다.

## ERRORS

오류가 없다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_cleanup_push()`,<br>`pthread_cleanup_pop()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

리눅스에서 `pthread_cleanup_push()`와 `pthread_cleanup_pop()` 함수는 각각 '`{`'와 '`}`'를 담은 텍스트로 확장되는 매크로로 구현되어 있다. 이 함수들의 호출 짝 안에서 선언한 변수들이 그 스코프 내에서만 보인다는 뜻이다.

POSIX.1에서는 `pthread_cleanup_push()`와 `pthread_cleanup_pop()`으로 감싼 블록을 `return`, `break`, `continue`, `goto`를 이용해 도중에 빠져나가는 결과가 규정되어 있지 않다고 한다. 이식 가능한 응용에서는 그렇게 하는 것을 피해야 한다.

## EXAMPLES

아래 프로그램은 이 페이지에서 기술하는 함수들의 간단한 사용 방식을 보여 준다. 프로그램에서 만드는 스레드에서 `pthread_cleanup_push()`와 `pthread_cleanup_pop()`으로 둘러싸인 루프를 실행한다. 그 루프에서는 전역 변수 `cnt`를 초당 한 번씩 증가시킨다. 어떤 명령행 인자를 주는가에 따라서 메인 스레드가 다른 스레드에게 취소 요청을 보내거나 다른 스레드가 루프를 빠져나가서 (`return`으로) 정상적으로 종료하도록 전역 변수를 설정한다.

다음 셸 세션에서는 메인 스레드가 다른 스레드에게 취소 요청을 보낸다.

```text
$ ./a.out
New thread started
cnt = 0
cnt = 1
Canceling thread
Called clean-up handler
Thread was canceled; cnt = 0
```

스레드가 취소되고 취소 정리 핸들러가 실행돼서 전역 변수 `cnt`의 값을 0으로 재설정한 것을 볼 수 있다.

다음 실행에서는 다른 스레드가 정상 종료하도록 메인 프로그램에서 전역 변수를 설정한다.

```text
$ ./a.out x
New thread started
cnt = 0
cnt = 1
Thread terminated normally; cnt = 2
```

(`cleanup_pop_arg`가 0이므로) 정리 핸들러가 실행되지 않았고 그래서 `cnt` 값이 재설정되지 않은 것을 볼 수 있다.

다음 실행에서는 다른 스레드가 정상 종료하도록 메인 프로그램에서 전역 변수를 설정하고 `cleanup_pop_arg`에 0 아닌 값을 준다.

```text
$ ./a.out x 1
New thread started
cnt = 0
cnt = 1
Called clean-up handler
Thread was canceled; cnt = 0
```

스레드가 취소되지 않았지만 `pthread_cleanup_pop()`에 0 아닌 인자를 주었기 때문에 정리 핸들러가 실행된 것을 볼 수 있다.

### 프로그램 소스

```c
#include <pthread.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

static int done = 0;
static int cleanup_pop_arg = 0;
static int cnt = 0;

static void
cleanup_handler(void *arg)
{
    printf("Called clean-up handler\n");
    cnt = 0;
}

static void *
thread_start(void *arg)
{
    time_t start, curr;

    printf("New thread started\n");

    pthread_cleanup_push(cleanup_handler, NULL);

    curr = start = time(NULL);

    while (!done) {
        pthread_testcancel();           /* 취소점 */
        if (curr < time(NULL)) {
            curr = time(NULL);
            printf("cnt = %d\n", cnt);  /* 취소점 */
            cnt++;
        }
    }

    pthread_cleanup_pop(cleanup_pop_arg);
    return NULL;
}

int
main(int argc, char *argv[])
{
    pthread_t thr;
    int s;
    void *res;

    s = pthread_create(&thr, NULL, thread_start, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_create");

    sleep(2);           /* 새 스레드 잠시 돌리기 */

    if (argc > 1) {
        if (argc > 2)
            cleanup_pop_arg = atoi(argv[2]);
        done = 1;

    } else {
        printf("Canceling thread\n");
        s = pthread_cancel(thr);
        if (s != 0)
            handle_error_en(s, "pthread_cancel");
    }

    s = pthread_join(thr, &res);
    if (s != 0)
        handle_error_en(s, "pthread_join");

    if (res == PTHREAD_CANCELED)
        printf("Thread was canceled; cnt = %d\n", cnt);
    else
        printf("Thread terminated normally; cnt = %d\n", cnt);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_cleanup_push_defer_np(3)]]</tt>, <tt>[[pthread_setcancelstate(3)]]</tt>, <tt>[[pthread_testcancel(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
