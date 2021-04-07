## NAME

sigaltstack - 시그널 스택 문맥 설정하고 얻기

## SYNOPSIS

```c
#include <signal.h>

int sigaltstack(const stack_t *ss, stack_t *old_ss);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>sigaltstack()</code></dt>
<dd>
<code>_XOPEN_SOURCE >= 500</code><br>
<code>    || /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200809L</code><br>
<code>    || /* glibc 버전 <= 2.19: */ _BSD_SOURCE</code>
</dd>
</dl>

## DESCRIPTION

`sigaltstack()`을 통해 프로세스에서 새 대체 시그널 스택을 지정하고 기존 대체 시그널 스택의 상태를 얻어 올 수 있다. 시그널 핸들러 설정에서 요청하면 시그널 핸들러 실행 동안 대체 시그널 스택을 사용한다.

대체 시그널 스택 사용을 위한 일반적인 절차는 다음과 같다.

1. 대체 시그널 스택에 사용할 메모리 영역을 할당한다.
2. `sigaltstack()`을 사용해 대체 시그널 스택의 존재와 위치를 시스템에게 알린다.
3. <tt>[[sigaction(2)]]</tt>으로 시그널 핸들러를 설정할 때 `SA_ONSTACK` 플래그를 지정해서 시그널 핸들러를 대체 시그널 스택 상에서 실행해야 함을 시스템에게 알린다.

`ss` 인자는 새 대체 시그널 스택을 지정하는데 사용하며 `old_ss` 인자는 현재 설정된 시그널 스택에 대한 정보를 얻는 데 사용한다. 이 중 한 가지 작업만 수행하고 싶다면 다른 인자를 NULL로 지정할 수 있다.

이 함수 인자들의 타입인 `stack_t` 타입은 다음과 같이 정의되어 있다.

```c
typedef struct {
    void  *ss_sp;     /* 스택의 기준 주소 */
    int    ss_flags;  /* 플래그 */
    size_t ss_size;   /* 스택의 바이트 수 */
} stack_t;
```

새 대체 시그널 스택을 설정하려면 이 구조체의 필드를 다음과 같이 설정한다.

<dl>
<dt><code>ss.ss_flags</code></dt>
<dd>

이 필드는 0 또는 다음 플래그를 담는다.

 <dl>
 <dt><code>SS_AUTODISARM</code> (리눅스 4.7부터)</dt>
 <dd>

 시그널 핸들러 진입 시 대체 시그널 스택 설정을 비운다. 시그널 핸들러가 반환할 때 이전 대체 시그널 스택 설정을 복원한다.

 <tt>[[swapcontext(3)]]</tt>로 시그널 핸들러에서 다른 문맥으로 전환하는 것을 안전하게 만들기 위해 이 플래그가 추가되었다. 이 플래그가 없으면 이후의 시그널 처리가 전환 전 시그널 핸들러의 상태를 오염시키게 된다. 이 플래그를 지원하지 않는 커널에서 이 플래그를 주면 `sigaltstack()`이 `EINVAL` 오류로 실패한다.
 </dd>
 </dl>
</dd>

<dt><code>ss.ss_sp</code></dt>
<dd>
이 필드는 스택의 시작 주소를 나타낸다. 대체 스택에서 시그널 핸들러를 호출할 때 <code>ss.ss_sp</code>로 받은 주소를 커널이 자동으로 기반 하드웨어 아키텍처에 맞는 주소 경계로 정렬한다.
</dd>

<dt><code>ss.ss_size</code></dt>
<dd>
이 필드는 스택의 크기를 나타낸다. 대체 시그널 스택의 크기에 대한 일반적인 요구들을 충족하는 크기가 되도록 상수 <code>SIGSTKSZ</code>가 정의되어 있다. 그리고 상수 <code>MINSIGSTKSZ</code>는 시그널 핸들러 실행에 필요한 최소 크기를 규정한다.
</dd>
</dl>

기존 스택을 비할성화 하려면 `ss.ss_flags`를 `SS_DISABLE`로 지정하면 된다. 이 경우 커널이 `ss.ss_flags`의 다른 플래그와 `ss`의 나머지 필드들을 무시한다.

`old_ss`가 NULL이 아니면 이를 이용해 `sigaltstack()` 호출 전에 적용되어 있던 대체 시그널 스택에 대한 정보를 반환한다. `old_ss.ss_sp` 필드와 `old_ss.ss_size` 필드가 그 스택의 시작 주소와 크기를 돌려준다. `old_ss.ss_flags` 필드가 다음 값들 중 하나를 돌려줄 수 있다.

<dl>
<dt><code>SS_ONSTACK</code></dt>
<dd>
프로세스가 현재 대체 시그널 스택 상에서 실행 중이다. (참고로 프로세스가 대체 시그널 스택 위에서 실행 중이면 그 스택을 바꾸는 것이 불가능하다.)
</dd>

<dt><code>SS_DISABLE</code></dt>
<dd>

대체 시그널 스택이 현재 비활성화되어 있다.

또는 프로세스가 현재 <code>SS_AUTODISARM</code> 플래그로 설정한 대체 시그널 스택에서 실행 중이어도 이 값을 반환한다. 이 경우 <tt>[[swapcontext(3)]]</tt>로 시그널 핸들러에서 다른 문맥으로 전환하는 것이 안전하다. 또 <code>sigaltstack()</code>을 다시 호출해서 또 다른 대체 시그널 스택을 설정하는 것도 가능하다.
</dd>

<dt><code>SS_AUTODISARM</code></dt>
<dd>
대체 시그널 스택이 위에서 기술한 것처럼 자동 해제하도록 표시되어 있다.
</dd>
</dl>

`ss`를 NULL로 지정하고 `old_ss`를 NULL 아닌 값으로 지정하면 변경 없이 현재의 대체 시그널 스택 설정을 얻을 수 있다.

## RETURN VALUE

`sigaltstack()`은 성공 시 0을 반환하고 실패 시 -1을 반환하면서 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EFAULT</code></dt>
<dd><code>ss</code>나 <code>old_ss</code>가 NULL이 아니며 프로세스의 주소 공간 밖의 영역을 가리키고 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>ss</code>가 NULL이 아니며 <code>ss_flags</code> 필드가 유효하지 않은 플래그를 담고 있다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>새 대체 시그널 스택에 지정한 크기 <code>ss.ss_size</code>가 <code>MINSIGSTKSZ</code>보다 작다.</dd>
<dt><code>EPERM</code></dt>
<dd>사용 중인 동안 (즉 프로세스가 이미 현행 대체 시그널 스택 위에서 실행 중인 동안) 대체 시그널 스택 변경 시도가 이뤄졌다.</dd>
</dl>

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `sigaltstack()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SUSv2, SVr4

`SS_AUTODISARM` 플래그는 리눅스 확장이다.

## NOTES

대체 시그널 스택을 가장 흔히 쓰는 곳은 정상적인 프로세스 스택에 사용 가능 공간이 고갈된 경우 발생하는 `SIGSEGV` 시그널을 다룰 때이다. 프로세스 스택 상에서 `SIGSEGV`에 대한 시그널 핸들러를 호출할 수 없는 이런 경우를 다루려면 대체 시그널 스택을 사용해야 한다.

프로세스가 표준 스택을 고갈시킬 수도 있다고 예상된다면 대체 시그널 스택을 설정하는 것이 유용하다. 예를 들면 위로 자라는 힙과 마주칠 정도로 스택이 크게 자라거나 `setrlimit(RLIMIT_STACK, &rlim)` 호출로 설정한 제한에 도달하여 그런 상황이 발생할 수 있다. 표준 스택이 고갈되면 커널이 프로세스에게 `SIGSEGV` 시그널을 보낸다. 이런 상황에서 그 시그널을 잡을 유일한 방법이 대체 시그널 스택이다.

리눅스가 지원하는 하드웨어 아키텍처들 대부분에서는 스택이 아래로 자란다. `sigaltstack()`에서는 스택 성장 방향을 자동으로 고려해 준다.

대체 시그널 스택에서 실행 중인 시그널 핸들러에서 호출하는 함수들 역시 그 대체 시그널 스택을 사용하게 된다. (프로세스가 대체 시그널 스택에서 실행 중인 동안 다른 시그널에 대한 핸들러가 호출되는 경우도 마찬가지이다.) 표준 스택과 달리 시스템에서 대체 시그널 스택을 자동으로 늘여 주지 않는다. 대체 시그널 스택에 할당된 크기를 넘어서면 예상 불가능한 결과를 유발하게 된다.

<tt>[[execve(2)]]</tt> 호출 성공 시 기존 대체 시그널 스택이 있으면 제거한다. <tt>[[fork(2)]]</tt>로 생성한 자식 프로세스는 부모의 대체 시그널 스택 설정 복사본을 물려받는다.

`sigaltstack()`은 이전의 `sigstack()` 호출을 대신한다. 하위 호환성을 위해 glibc에서는 `sigstack()`도 제공한다. 새로운 응용들은 모두 `sigaltstack()`을 이용해 작성하는 게 좋다.

### 역사

4.2BSD에 `sigstack()` 시스템 호출이 있었다. 살짝 다른 구조체를 사용했는데 호출자가 스택 성장 방향을 알고 있어야 한다는 심각한 단점이 있었다.

## EXAMPLE

다음 코드 조각은 `sigaltstack()`을 (그리고 <tt>[[sigaction(2)]]</tt>을) 이용해 `SIGSEGV` 시그널 핸들러가 사용할 대체 시그널 스택을 설치하는 것을 보여 준다.

```c
stack_t ss;

ss.ss_sp = malloc(SIGSTKSZ);
if (ss.ss_sp == NULL) {
    perror("malloc");
    exit(EXIT_FAILURE);
}

ss.ss_size = SIGSTKSZ;
ss.ss_flags = 0;
if (sigaltstack(&ss, NULL) == -1) {
    perror("sigaltstack");
    exit(EXIT_FAILURE);
}

sa.sa_flags = SA_ONSTACK;
sa.sa_handler = handler();      /* 시그널 핸들러 주소 */
sigemptyset(&sa.sa_mask);
if (sigaction(SIGSEGV, &sa, NULL) == -1) {
    perror("sigaction");
    exit(EXIT_FAILURE);
}
```

## BUGS

리눅스 2.2 이전에서 `ss.sa_flags`에 지정할 수 있는 유일한 플래그는 `SS_DISABLE`이었다. 리눅스 2.4 커널 릴리스까지 과정에서 누군가 내용을 혼동하여 커널이 `ss.ss_flags`에서 `SS_ONSTACK`을 받아들이도록 했다. 이렇게 했을 때의 동작 방식은 `ss_flags`가 0일 때와 같다. 다른 구현들에서, 그리고 POSIX.1에 따르면 `SS_ONSTACK`은 `old_ss.ss_flags`에서 알려주는 플래그로만 등장한다. `ss.ss_flags`에서 이 플래그를 지정할 필요가 전혀 없다. (그리고 사실 그렇게 하면 이식성이 떨어진다. 어떤 구현들에서는 `ss.ss_flags`에 `SS_ONSTACK`을 지정하면 오류를 뱉기 때문이다.)

## SEE ALSO

<tt>[[execve(2)]]</tt>, <tt>[[setrlimit(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[siglongjmp(3)]]</tt>, <tt>[[sigsetjmp(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2017-11-08
