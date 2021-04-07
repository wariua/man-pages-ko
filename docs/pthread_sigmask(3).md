## NAME

pthread_sigmask - 블록 된 시그널 마스크 조사하고 바꾸기

## SYNOPSIS

```c
#include <signal.h>

int pthread_sigmask(int how, const sigset_t *set, sigset_t *oldset);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>pthread_sigmask()</code>:</dt>
<dd><code>_POSIX_C_SOURCE >= 199506L || _XOPEN_SOURCE >= 500</code></dd>
</dl>

## DESCRIPTION

`pthread_sigmask()` 함수는 <tt>[[sigprocmask(2)]]</tt>와 비슷하되 다중 스레드 프로그램에서의 사용을 POSIX.1에서 명시적으로 명세하고 있다는 차이가 있다. 다른 차이점들을 이 페이지에서 언급한다.

이 함수의 인자와 동작에 대한 설명은 <tt>[[sigprocmask(2)]]</tt>를 보라.

## RETURN VALUE

성공 시 `pthread_sigmask()`는 0을 반환한다. 오류 시 오류 번호를 반환한다.

## ERRORS

<tt>[[sigprocmask(2)]]</tt> 참고.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_sigmask()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

새 스레드는 생성자의 시그널 마스크 사본을 물려받는다.

glibc의 `pthread_sigmask()` 함수에서는 NPTL 스레딩 구현 내부에서 쓰는 두 가지 실시간 시그널을 막으려는 시도를 조용히 무시한다. 자세한 내용은 <tt>[[nptl(7)]]</tt>을 보라.

## EXAMPLE

아래 프로그램에서는 주 스레드에서 몇 가지 시그널들을 막고 나서 <tt>[[sigwait(3)]]</tt>을 통해 그 시그널들을 가져오는 전용 스레드를 만든다. 다음 셸 세션이 사용 방식을 보여 준다.

```
$ ./a.out &
[1] 5423
$ kill -QUIT %1
Signal handling thread got signal 3
$ kill -USR1 %1
Signal handling thread got signal 10
$ kill -TERM %1
[1]+  Terminated              ./a.out
```

### 프로그램 소스

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <errno.h>

/* 간단한 오류 처리 함수 */

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

static void *
sig_thread(void *arg)
{
    sigset_t *set = arg;
    int s, sig;

    for (;;) {
        s = sigwait(set, &sig);
        if (s != 0)
            handle_error_en(s, "sigwait");
        printf("Signal handling thread got signal %d\n", sig);
    }
}

int
main(int argc, char *argv[])
{
    pthread_t thread;
    sigset_t set;
    int s;

    /* SIGQUIT과 SIGUSR1 막기. main()이 만드는 다른 스레드들은
       이 시그널 마스크 사본을 물려받게 된다. */

    sigemptyset(&set);
    sigaddset(&set, SIGQUIT);
    sigaddset(&set, SIGUSR1);
    s = pthread_sigmask(SIG_BLOCK, &set, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_sigmask");

    s = pthread_create(&thread, NULL, &sig_thread, (void *) &set);
    if (s != 0)
        handle_error_en(s, "pthread_create");

    /* 주 스레드에서 계속해서 다른 스레드들을 만들거나 다른
       작업 수행 */

    pause();            /* 프로그램 테스트를 위한 중지 */
}
```

## SEE ALSO

<tt>[[sigaction(2)]]</tt>, <tt>[[sigpending(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_kill(3)]]</tt>, <tt>[[sigsetops(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[signal(7)]]</tt>

----

2019-03-06
