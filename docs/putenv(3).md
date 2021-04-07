## NAME

putenv - 환경 변수 바꾸거나 추가하기

## SYNOPSIS

```c
#include <stdlib.h>

int putenv(char *string);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>putenv()</code></dt>
<dd>
<code>_XOPEN_SOURCE</code><br>
<code>|| /* glibc 2.19부터: */ _DEFAULT_SOURCE</code><br>
<code>|| /* glibc 버전 <= 2.19: */ _SVID_SOURCE</code>
</dd>
</dl>

## DESCRIPTION

`putenv()` 함수는 환경 변수 값을 추가하거나 바꾼다. 인자 `string`은 `name=value` 형태이다. `name`이 환경 내에 존재하지 않으면 `string`을 환경에 추가한다. `name`이 이미 존재하는 경우에는 환경 안의 `name`의 값을 `value`로 바꾼다. `string`이 가리키는 문자열이 환경의 일부가 되며, 따라서 그 문자열을 변경하면 환경이 바뀐다.

## RETURN VALUE

`putenv()` 함수는 성공 시 0을 반환하고 오류 발생 시 0 아닌 값을 반환한다. 오류 발생 시 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>ENOMEM</code></dt>
<dd>새 환경을 할당할 공간이 충분치 않음.</dd>
</dl>

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `putenv()` | 스레드 안전성 | MT-Unsafe const:env |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

## NOTES

`putenv()` 함수가 재진입 가능이 아닐 수도 있다. glibc 2.0에서는 아니지만 glibc 2.1 버전은 재진입 가능이다.

버전 2.1.2부터 glibc 구현이 SUSv2를 준수한다. 즉 `putenv()`에 준 포인터 `string`을 사용한다. 특히 그 문자열이 환경의 일부가 되고, 그래서 이후 변경하면 환경이 바뀌게 된다. (따라서 자동 변수를 인자로 해서 `putenv()`를 호출한 다음 `string`이 여전히 환경의 일부인 상태로 호출 함수에서 반환하는 것은 오류다.) 하지만 glibc 버전 2.0부터 2.1.1까지에서는 다르다. 즉 문자열의 복사본을 쓴다. 이는 한편으로 메모리 누수를 일으키고 다른 한편으로 SUSv2를 위반한다.

4.4BSD 버전은 glibc 2.0처럼 복사본을 쓴다.

SUSv2에서 원형의 `const`를 없앴고 glibc 2.1.3에서 그렇게 됐다.

GNU C 라이브러리 구현에서는 비표준 확장을 제공한다. 다음처럼 `string`에 등호가 포함돼 있지 않으면 지정한 변수를 호출자의 환경에서 제거한다.

```c
putenv("NAME");
```

## SEE ALSO

<tt>[[clearenv(3)]]</tt>, <tt>[[getenv(3)]]</tt>, <tt>[[setenv(3)]]</tt>, <tt>[[unsetenv(3)]]</tt>, <tt>[[environ(7)]]</tt>

----

2019-03-06
