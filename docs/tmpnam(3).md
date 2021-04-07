## NAME

tmpnam, tmpnam_r - 임시 파일 이름 만들기

## SYNOPSIS

```c
#include <stdio.h>

char *tmpnam(char *s);
char *tmpnam_r(char *s);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>tmpnam_r()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.19부터:</dt>
 <dd><code>_DEFAULT_SOURCE</code></dd>
 <dt>glibc 2.19까지:</dt>
 <dd><code>_BSD_SOURCE || _SVID_SOURCE</code></dd>
 </dl>
</dd>

## DESCRIPTION

**주의**: 이 함수들을 쓰지 말 것. 대신 <tt>[[mkstemp(3)]]</tt>나 <tt>[[tmpfile(3)]]</tt>을 사용하라.

`tmpnam()` 함수는 유효한 파일명인 문자열에 대한 포인터를 반환한다. 어느 시점에는 그 이름으로 된 파일이 존재하지 않으므로 순진한 프로그래머라면 임시 파일에 적당한 이름이라고 생각할 수도 있다. 인자 `s`가 NULL이면 내부 정적 버퍼에다 이름을 생성하므로 다음 번 `tmpnam()` 호출로 덮어 써질 수도 있다. `s`가 NULL이 아니면 `s`가 가리키는 (길이가 최소 `L_tmpnam`인) 문자 배열로 이름을 복사하며 성공 시 값 `s`를 반환한다.

생성되는 경로명에는 디렉터리 선두부 `P_tmpdir`이 있다. (`L_tmpnam`과 `P_tmpdir` 모두 `<stdio.h>`에 정의돼 있다. 아래서 언급하는 `TMP_MAX`도 마찬가지.)

`tmpnam_r()` 함수는 `tmpnam()`과 같은 일을 수행하되 `s`가 NULL이면 (오류를 나타내는) NULL을 반환한다.

## RETURN VALUE

이 함수들은 유일한 임시 파일명에 대한 포인터를 반환한다. 유일한 이름을 생성할 수 없으면 NULL을 반환한다.

## ERRORS

어떤 오류도 정의되어 있지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `tmpnam()` | 스레드 안전성 | MT-Unsafe race:tmpnam/!s |
| `tmpnam_r()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`tmpnam()`: SVr4, 4.3BSD, C89, C99, POSIX.1-2001. POSIX.1-2008에서 `tmpnam()`을 구식으로 표시하였다.

`tmpnam_r()`은 비표준 확장이며 몇몇 다른 시스템에서도 사용 가능하다.

## NOTES

`tmpnam()` 함수는 `TMP_MAX` 번까지는 호출 때마다 다른 문자열을 만들어 낸다. `TMP_MAX` 번 넘게 호출하는 경우의 동작 방식은 구현에서 규정한다.

이 함수들이 추측하기 어려운 이름을 만들어 내기는 하지만 그래도 경로명이 반환되는 시점과 프로그램에서 그 경로명을 여는 시점 사이에 다른 프로그램이 <tt>[[open(2)]]</tt>으로나 심볼릭 링크 형태로 그 경로명을 만들어 낼 수도 있다. 그리고 그게 보안상의 구멍으로 이어질 수 있다. 그런 가능성을 피하려면 경로명을 열 때 <tt>[[open(2)]]</tt>의 `O_EXCL` 플래그를 사용하면 된다. 또는 더 바람직하게는 <tt>[[mkstemp(3)]]</tt>나 <tt>[[tmpfile(3)]]</tt>을 쓰면 된다.

스레드를 사용하는 이식 가능한 응용에서는 `_POSIX_THREADS`나 `_POSIX_THREAD_SAFE_FUNCTIONS`가 정의돼 있는 경우 NULL 인자로 `tmpnam()`을 호출할 수 없다.

## BUGS

이 함수들을 절대 쓰지 마라. 대신 <tt>[[mkstemp(3)]]</tt>나 <tt>[[tmpfile(3)]]</tt>을 사용하라.

## SEE ALSO

<tt>[[mkstemp(3)]]</tt>, <tt>[[mktemp(3)]]</tt>, <tt>[[tempnam(3)]]</tt>, <tt>[[tmpfile(3)]]</tt>

----

2017-09-15
