## NAME

makecontext, swapcontext - 사용자 문맥 조작하기

## SYNOPSIS

```c
#include <ucontext.h>

void makecontext(ucontext_t *ucp, void (*func)(), int argc, ...);

int swapcontext(ucontext_t *oucp, const ucontext_t *ucp);
```

## DESCRIPTION

시스템 V 계열 환경에서는 `<ucontext.h>`에 `ucontext_t`라는 타입이 있으며 네 가지 함수 <tt>[[getcontext(3)]]</tt>, <tt>[[setcontext(3)]]</tt>, `makecontext()`, `swapcontext()`를 통해 한 프로세스 내의 여러 제어 스레드들 사이에서 사용자 수준 문맥 전환이 가능하다.

그 타입과 앞쪽 두 함수에 대해선 <tt>[[getcontext(3)]]</tt>를 보라.

`makecontext()` 함수는 (<tt>[[getcontext(3)]]</tt> 호출에서 얻은) `ucp`가 가리키는 문맥을 변경한다. `makecontext()`를 호출하기 전에 호출자는 이 문맥을 위한 새 스택을 할당하여 그 주소를 `ucp->uc_stack`에 지정해야 하며 후속 문맥을 정의하여 그 주소를 `ucp->uc_link`에 지정해야 한다.

이후에 (<tt>[[setcontext(3)]]</tt>나 `swapcontext()`를 이용해) 이 문맥을 활성화할 때 함수 `func`가 호출되면서 `argc` 다음에 오는 일련의 정수(`int`) 인자들이 전달된다. 호출자는 이 인자들의 개수를 `argc`에 지정해야 한다. 이 함수가 반환하고 나면 후속 문맥이 활성화된다. 후속 문맥 포인터가 NULL이면 스레드가 끝난다.

`swapcontext()` 함수는 `oucp`가 가리키는 구조체에 현재 문맥을 저장한 다음 `ucp`가 가리키는 문맥을 활성화한다.

## RETURN VALUE

성공 시 `swapcontext()`는 반환하지 않는다. (하지만 나중에 `oucp`가 활성화될 때 돌아올 수도 있으며, 그 경우 `swapcontext()`가 0을 반환하는 것처럼 보인다.) 오류 시 `swapcontext()`는 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`ENOMEM`
:   남은 스택 공간이 충분하지 않음.

## VERSIONS

glibc에서 버전 2.1부터 `makecontext()` 및 `swapcontext()`를 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `makecontext()` | 스레드 안전성 | MT-Safe race:ucp |
| `swapcontext()` | 스레드 안전성 | MT-Safe race:oucp race:ucp |

## CONFORMING TO

SUSv2, POSIX.1-2001, POSIX.1-2008에서 이식성 문제를 이유로 `makecontext()` 및 `swapcontext()` 명세를 제거하였으며 대신 POSIX 스레드를 사용하게 응용을 재작성하기를 권고하고 있다.

## NOTES

`ucp->uc_stack`의 해석은 <tt>[[sigaltstack(2)]]</tt>에서와 마찬가지이다. 즉, 스택의 성장 방향과 무관하게 이 구조체에는 스택으로 사용할 메모리 영역의 시작점과 길이를 담는다. 따라서 사용자 프로그램이 그 방향에 대해 신경쓸 필요가 없다.

`int`와 포인터 타입이 같은 크기인 아키텍처에서는 (가령 x86-32에서는 두 타입 모두 32비트임) `makecontext()`에서 `argc` 다음에 오는 인자들로 포인터를 전달하는 것이 어찌어찌 가능할 수도 있다. 하지만 이식성이 보장되지 않고 표준에 따르면 규정되어 있지 않으며 포인터가 `int`보다 큰 아키텍처에서 동작하지 않게 된다. 그럼에도 불구하고 glibc에서 버전 2.8부터 `makecontext()`를 조금 변경해서 일부 64비트 아키텍처들(가령 x86-64)에서 그렇게 하는 것을 허용한다.

## EXAMPLE

아래 예시 프로그램은 <tt>[[getcontext(3)]]</tt>, `makecontext()`, `swapcontext()` 사용 방식을 보여 준다. 프로그램을 실행하면 다음 결과가 나온다.

```text
$ ./a.out
main: swapcontext(&uctx_main, &uctx_func2)
func2: started
func2: swapcontext(&uctx_func2, &uctx_func1)
func1: started
func1: swapcontext(&uctx_func1, &uctx_func2)
func2: returning
func1: returning
main: exiting
```

### 프로그램 소스

```c
#include <ucontext.h>
#include <stdio.h>
#include <stdlib.h>

static ucontext_t uctx_main, uctx_func1, uctx_func2;

#define handle_error(msg) \
    do { perror(msg); exit(EXIT_FAILURE); } while (0)

static void
func1(void)
{
    printf("func1: started\n");
    printf("func1: swapcontext(&uctx_func1, &uctx_func2)\n");
    if (swapcontext(&uctx_func1, &uctx_func2) == -1)
        handle_error("swapcontext");
    printf("func1: returning\n");
}

static void
func2(void)
{
    printf("func2: started\n");
    printf("func2: swapcontext(&uctx_func2, &uctx_func1)\n");
    if (swapcontext(&uctx_func2, &uctx_func1) == -1)
        handle_error("swapcontext");
    printf("func2: returning\n");
}

int
main(int argc, char *argv[])
{
    char func1_stack[16384];
    char func2_stack[16384];

    if (getcontext(&uctx_func1) == -1)
        handle_error("getcontext");
    uctx_func1.uc_stack.ss_sp = func1_stack;
    uctx_func1.uc_stack.ss_size = sizeof(func1_stack);
    uctx_func1.uc_link = &uctx_main;
    makecontext(&uctx_func1, func1, 0);

    if (getcontext(&uctx_func2) == -1)
        handle_error("getcontext");
    uctx_func2.uc_stack.ss_sp = func2_stack;
    uctx_func2.uc_stack.ss_size = sizeof(func2_stack);
    /* argc > 1 아니면 후속 문맥이 func1() */
    uctx_func2.uc_link = (argc > 1) ? NULL : &uctx_func1);
    makecontext(&uctx_func2, func2, 0);

    printf("main: swapcontext(&uctx_main, &uctx_func2)\n");
    if (swapcontext(&uctx_main, &uctx_func2) == -1)
        handle_error("swapcontext");

    printf("main: exiting\n");
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[sigaction(2)]]</tt>, <tt>[[sigaltstack(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[getcontext(3)]]</tt>, <tt>[[sigsetjmp(3)]]</tt>

----

2019-03-06
