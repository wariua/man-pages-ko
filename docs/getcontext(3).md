## NAME

getcontext, setcontext - 사용자 문맥 얻거나 설정하기

## SYNOPSIS

```c
#include <ucontext.h>

int getcontext(ucontext_t *ucp);
int setcontext(const ucontext_t *ucp);
```

## DESCRIPTION

시스템 V 계열 환경에서는 `<ucontext.h>`에 `mcontext_t`와 `ucontext_t`라는 두 가지 타입이 있으며 네 가지 함수 `getcontext()`, `setcontext()`, <tt>[[makecontext(3)]]</tt>, <tt>[[swapcontext(3)]]</tt>를 통해 한 프로세스 내의 여러 제어 스레드들 사이에서 사용자 수준 문맥 전환이 가능하다.

`mcontext_t` 타입은 장치 의존적이며 내용이 감춰져 있다. `ucontext_t` 타입은 최소한 다음 필드들을 가지고 있는 구조체이다.

```c
typedef struct ucontext_t {
    struct ucontext_t *uc_link;
    sigset_t          uc_sigmask;
    stack_t           uc_stack;
    mcontext_t        uc_mcontext;
    ...
} ucontext_t;
```

`sigset_t`와 `stack_t`는 `<signal.h>`에 정의되어 있다. 여기서 `uc_link`는 현재 문맥이 종료되었을 때 재개될 문맥을 가리키며 (현재 문맥을 <tt>[[makecontext(3)]]</tt>으로 만든 경우), `uc_sigmask`는 이 문맥에서 차단된 시그널들의 집합이고 (<tt>[[sigprocmask(2)]]</tt> 참고), `uc_stack`은 이 문맥에서 사용하는 스택이고 (<tt>[[sigaltstack(2)]]</tt> 참고), `uc_context`는 저장된 문맥의 장치별 표현으로 호출 스레드의 장치 레지스터들이 포함된다.

`getcontext()` 함수는 `ucp`가 가리키는 구조체를 현재의 활성 문맥으로 초기화 한다.

`setcontext()` 함수는 `ucp`가 가리키는 사용자 문맥을 복원한다. 호출 성공 시 반환하지 않는다. 그 문맥은 `getcontext()`나 <tt>[[makecontext(3)]]</tt> 호출로 얻은 것이거나 시그널 핸들러에서 세 번째 인자로 받은 것이어야 한다. (<tt>[[sigaction(2)]]</tt>의 `SA_SIGINFO` 플래그 설명 참고.)

`getcontext()` 호출로 얻은 문맥인 경우 그 호출이 방금 반환한 것처럼 프로그램 실행이 이어진다.

<tt>[[makecontext(3)]]</tt> 호출로 얻은 문맥인 경우 <tt>[[makecontext(3)]]</tt> 호출 두 번째 인자로 지정했던 함수 `func` 호출로 프로그램 실행이 이어진다. `func` 함수가 반환하면 <tt>[[makecontext(3)]]</tt> 호출 첫 번째 인자로 지정했던 `ucp` 구조체의 `uc_link` 멤버로 계속 진행한다. 그 멤버가 NULL이면 스레드가 끝난다.

시그널 핸들러 호출로 얻은 문맥인 경우에 이전 표준 문서에서는 "시그널로 중단된 인스트럭션 다음의 프로그램 인스트럭션으로 프로그램 실행이 이어진다"고 되어 있었다. 하지만 SUSv2에서 이 문장이 제거되었고 현재 판정은 "그 결과가 명세되어 있지 않음"이다.

## RETURN VALUE

성공 시 `getcontext()`는 0을 반환하며 `setcontext()`는 반환하지 않는다. 오류 시 둘 모두 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

정의되어 있지 않음.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getcontext()`, `setcontext()` | 스레드 안전성 | MT-Safe race:ucp |

## CONFORMING TO

SUSv2, POSIX.1-2001, POSIX.1-2008에서 이식성 문제를 이유로 `getcontext()` 명세를 제거하였으며 대신 POSIX 스레드를 사용하게 응용을 재작성하기를 권고하고 있다.

## NOTES

이 메커니즘이 가장 먼저 구체화된 것은 <tt>[[setjmp(3)]]</tt>/<tt>[[longjmp(3)]]</tt> 메커니즘이었다. 그 메커니즘에서는 시그널 문맥 처리를 규정하지 않았고, 그래서 다음 단계가 <tt>[[sigsetjmp(3)]]</tt>/<tt>[[siglongjmp(3)]]</tt> 쌍이었다. 그리고 현재 메커니즘에서는 훨씬 더 유연한 제어가 가능하다. 한편으로 `getcontext()`에서 반환된 것이 처음 호출에서 돌아온 것인지 `setcontext()` 호출을 통한 것인지 알아낼 손쉬운 방법이 없다. 사용자가 따로 확인 방식을 고안해야 하는데, 레지스터들이 복원되므로 레지스터 변수는 사용할 수 없다.

시그널이 발생했을 때 커널이 현재 사용자 문맥을 저장하고 시그널 핸들러를 위한 새 문맥을 만든다. 핸들러에서 <tt>[[longjmp(3)]]</tt>을 사용하도록 놔둬선 안 된다. 문맥과 관련해 어떻게 될지 규정되어 있지 않다. 대신 <tt>[[siglongjmp(3)]]</tt>나 `setcontext()`를 사용하라.

## SEE ALSO

<tt>[[sigaction(2)]]</tt>, <tt>[[sigaltstack(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[longjmp(3)]]</tt>, <tt>[[makecontext(3)]]</tt>, <tt>[[sigsetjmp(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
