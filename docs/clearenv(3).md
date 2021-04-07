## NAME

clearenv - 환경 비우기

## SYNOPSIS

```c
#include <stdlib.h>

int clearenv(void);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>clearenv()</code>:</dt>
<dd>
<code>/* glibc 2.19부터: */ _DEFAULT_SOURCE</code><br>
<code>    || /* glibc 버전 <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE</code>
</dd>
</dl>

## DESCRIPTION

`clearenv()` 함수는 환경에서 모든 이름-값 쌍을 비우고 외부 변수 `environ`의 값을 NULL로 설정한다. 이 호출 후에 <tt>[[putenv(3)]]</tt> 및 <tt>[[setenv(3)]]</tt>를 이용해 환경에 새 변수를 추가할 수 있다.

## RETURN VALUE

`clearenv()` 함수는 성공 시 0을 반환하고 실패 시 0 아닌 값을 반환한다.

## VERSIONS

glibc 2.0부터 사용 가능.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `clearenv()` | 스레드 안전성 | MT-Unsafe const:env |

## CONFORMING TO

다양한 유닉스 변종들(DG/UX, HP-UX, QNX, ...). POSIX.9 (FORTRAN77 바인딩). POSIX.1-1996에서는 `clearenv()`와 <tt>[[putenv(3)]]</tt>를 받아들이지 않았지만 마음을 바꿔서 이후 어느 버전에서 그 함수들을 추가하기로 했다. (B.4.6.1절 참고.) 하지만 POSIX.1-2001에서 <tt>[[putenv(3)]]</tt>만 추가하고 `clearenv()`는 거부했다.

## NOTES

`clearenv()`를 쓸 수 없는 시스템에서는 아마 다음 할당으로도 될 것이다.

```
environ = NULL;
```

<tt>[[exec(3)]]</tt>로 실행되는 프로그램에 전달되는 환경을 신중하게 통제하고 싶은 보안 중시 응용들에서 `clearenv()` 함수가 유용할 수 있다. 그런 응용에서는 먼저 환경을 비운 다음 정선한 환경 변수들을 추가하게 될 것이다.

참고로 `clearenv()`의 주된 효과는 포인터 <tt>[[environ(7)]]</tt>의 값을 조정하는 것이다. 이 함수가 환경 정의를 담은 버퍼들의 내용을 삭제하지는 않는다.

DG/UX 및 Tru64 맨 페이지에는 이렇게 적혀 있다: <tt>[[putenv(3)]]</tt>, <tt>[[getenv(3)]]</tt>, `clearenv()` 외의 어떤 방법으로 `environ`을 변경했으면 `clearenv()`가 오류를 반환하게 되며 프로세스 환경이 그대로 유지된다.

## SEE ALSO

<tt>[[getenv(3)]]</tt>, <tt>[[putenv(3)]]</tt>, <tt>[[setenv(3)]]</tt>, <tt>[[unsetenv(3)]]</tt>, <tt>[[environ(7)]]</tt>

----

2017-09-15
