## NAME

tempnam - 임시 파일 이름 만들기

## SYNOPSIS

```c
#include <stdio.h>

char *tempnam(const char *dir, const char *pfx);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`tempnam()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

*이 함수를 절대 쓰지 말 것.* 대신 <tt>[[mkstemp(3)]]</tt>나 <tt>[[tmpfile(3)]]</tt>을 사용하라.

`tempnam()` 함수는 유효한 파일명인 문자열의 포인터를 반환한다. `tempnam()`에서 확인하는 시점에는 그 이름으로 된 파일이 존재하지 않는다. `pfx`가 최대 다섯 바이트의 NULL 아닌 문자열인 경우 생성 경로명의 파일명 부분이 `pfx`로 시작하게 된다. 생성 경로명의 디렉터리 부분이 "적절"해야 (보통 적어도 쓰기 가능해야) 한다.

적절한 디렉터리를 찾기 위해 다음 단계를 거친다.

a) 환경 변수 `TMPDIR`이 존재하여 적절한 디렉터리 이름을 담고 있는 경우에는 그걸 쓴다.

b) 아니면, `dir` 인자가 NULL이 아니고 적절하면 그걸 쓴다.

c) 아니면, (`<stdio.h>`에 정의된) `P_tmpdir`이 적절하면 그걸 쓴다.

d) 마지막으로, 구현에서 정한 디렉터리를 사용할 수 있다.

`tempnam()`이 반환하는 문자열은 <tt>[[malloc(3)]]</tt>으로 할당한 것이므로 <tt>[[free(3)]]</tt>로 해제해 줘야 한다.

## RETURN VALUE

성공 시 `tempnam()` 함수는 유일한 임시 파일명의 포인터를 반환한다. 유일한 이름을 생성할 수 없는 경우 NULL을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`ENOMEM`
:   저장 공간 할당에 실패했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `tempnam()` | 스레드 안전성 | MT-Safe env |

## CONFORMING TO

SVr4, 4.3BSD, POSIX.1-2001. POSIX.1-2008에서 `tempnam()`을 구식으로 표시하였다.

## NOTES

`tempnam()`이 추측하기 어려운 이름을 만들어 내기는 하지만 그래도 `tempnam()`이 경로명을 반환하는 시점과 프로그램에서 그 경로명을 여는 시점 사이에 다른 프로그램이 <tt>[[open(2)]]</tt>으로나 심볼릭 링크 형태로 그 경로명을 만들어 낼 수도 있다. 그리고 그게 보안 취약점으로 이어질 수 있다. 그런 가능성을 피하려면 경로명을 열 때 <tt>[[open(2)]]</tt>의 `O_EXCL` 플래그를 사용하면 된다. 또는 더 바람직하게는 <tt>[[mkstemp(3)]]</tt>나 <tt>[[tmpfile(3)]]</tt>을 쓰면 된다.

SUSv2에서는 `TMPDIR` 사용을 언급하고 있지 않다. glibc에서는 프로그램이 set-user-ID가 아닐 때만 쓴다. SVr4에서 d) 단계에 쓰는 디렉터리는 `/tmp`이다. (glibc에서도 그렇다.)

경로명을 반환하는 데 쓰는 메모리를 동적으로 할당하기 때문에 `tempnam()`은 재진입 가능하며, 따라서 <tt>[[tmpnam(3)]]</tt>과 달리 스레드에 안전하다.

`tempnam()` 함수는 (`<stdio.h>`에 정의된) `TMP_MAX` 번까지는 호출 때마다 다른 문자열을 만들어 낸다. `TMP_MAX` 번 넘게 호출하는 경우의 동작 방식은 구현에서 규정한다.

`tempnam()`은 `pfx` 처음의 최대 다섯 바이트를 사용한다.

`tempnam()`의 glibc 구현에서는 유일한 이름을 찾는 데 실패한 경우 `EEXIST` 오류로 실패한다.

## BUGS

"적절"의 정확한 의미가 규정돼 있지 않다. 디렉터리 접근 가능 여부를 어떻게 판단하는지 명세돼 있지 않다.

## SEE ALSO

<tt>[[mkstemp(3)]]</tt>, <tt>[[mktemp(3)]]</tt>, <tt>[[tmpfile(3)]]</tt>, <tt>[[tmpnam(3)]]</tt>

----

2021-03-22
