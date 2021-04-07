## NAME

strnlen - 고정 크기 문자열의 길이 알아내기

## SYNOPSIS

```c
#include <string.h>

size_t strnlen(const char *s, size_t maxlen);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>strnlen()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.10부터:</dt>
 <dd><code>_POSIX_C_SOURCE >= 200809L</code></dd>
 <dt>glibc 2.10 전:</dt>
 <dd><code>_GNU_SOURCE</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`strnlen()` 함수는 `s`가 가리키는 문자열에서 종료용 널 바이트(`'\0'`)를 제외한 바이트 수를 반환하되 `maxlen`을 최대로 한다. 이 과정에서 `strnlen()`은 `s`가 가리키는 문자열의 처음 `maxlen` 개 문자만 살펴보며 절대 `s+maxlen`와 그 너머는 보지 않는다.

## RETURN VALUE

`strnlen()` 함수는 그 값이 `maxlen`보다 작으면 `strlen(s)`을 반환한다. `s`가 가리키는 처음 `maxlen` 개 문자들 중에 널 종료(`'\0'`)가 없으면 `maxlen`을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strnlen()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2008.

## SEE ALSO

`strlen(3)`

----

2019-03-06
