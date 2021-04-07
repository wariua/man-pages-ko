## NAME

siginterrupt - 시그널이 시스템 호출을 중단시킬 수 있게 허용하기

## SYNOPSIS

```c
#include <signal.h>

int siginterrupt(int sig, int flag);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>siginterrupt()</code>:</dt>
<dd>
<code>_XOPEN_SOURCE >= 500</code><br>
<code>    || /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200809L</code><br>
<code>    || /* glibc 버전 <= 2.19: */ _BSD_SOURCE</code>
</dd>
</dl>

## DESCRIPTION

`siginterrupt()` 함수는 시그널 `sig`에 의해 시스템 호출이 중단될 때의 재시작 동작 방식을 변경한다. `flag` 인자가 거짓(0)이면 지정한 시그널 `sig`에 의해 중단된 경우 시스템 호출이 재시작 된다. 이것이 리눅스에서 기본 동작 방식이다.

`flag`가 참(1)이고 어떤 데이터도 이동하지 않았으면 시그널 `sig`에 의해 중단된 시스템 호출이 -1을 반환하게 되고 `errno`가 `EINTR`로 설정된다.

`flag`가 참(1)이고 데이터 이동이 시작됐으면 시스템 호출이 중단되고 실제 전송된 데이터의 양을 반환하게 된다.

## RETURN VALUE

`siginterrupt()` 함수는 성공 시 0을 반환한다. 시그널 번호 `sig`가 유효하지 않으면 -1을 반환하고 오류 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EINVAL</code></dt>
<dd>지정한 시그널 번호가 유효하지 않다.</dd>
</dl>

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `siginterrupt()` | 스레드 안전성 | MT-Unsafe const:sigintr |

## CONFORMING TO

4.3BSD, POSIX.1-2001. POSIX.1-2008에서는 `siginterrupt()`를 구식으로 표시하며 대신 <tt>[[sigaction(2)]]</tt>을 `SA_RESTART` 플래그와 함께 사용하기를 권장한다.

## SEE ALSO

<tt>[[signal(2)]]</tt>

----

2016-03-15
