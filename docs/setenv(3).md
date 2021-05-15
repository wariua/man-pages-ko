## NAME

setenv - 환경 변수 바꾸거나 추가하기

## SYNOPSIS

```c
#include <stdlib.h>

int setenv(const char *name, const char *value, int overwrite);
int unsetenv(const char *name);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`setenv()`, `unsetenv()`
:   `_POSIX_C_SOURCE >= 200112L`<br>
    `    || /* glibc <= 2.19: */ _BSD_SOURCE`

## DESCRIPTION

`setenv()` 함수는 환경에 `name`이 존재하지 않으면 변수 `name`을 `value` 값으로 추가한다. 환경에 `name`이 이미 존재하는 경우에는 `overwrite`가 0이 아니면 그 값을 `value`로 바꾼다. `overwrite`가 0이면 `name`의 값을 바꾸지 않는다. (그리고 `setenv()`는 성공 상태를 반환한다.) 이 함수는 (<tt>[[putenv(3)]]</tt>와 반대로) `name`과 `value`가 가리키는 문자열의 복사본을 만든다.

`unsetenv()` 함수는 환경에서 변수 `name`을 삭제한다. 환경에 `name`이 존재하지 않는 경우에는 함수가 성공하고 환경이 안 바뀐다.

## RETURN VALUE

`setenv()` 및 `unsetenv()` 함수는 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `name`이 NULL이거나, 길이 0인 문자열을 가리키고 있거나, '=' 문자를 담고 있다.

`ENOMEM`
:   환경에 새 변수를 추가하기 위한 메모리가 충분치 않음.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `setenv()`, `unsetenv()` | 스레드 안전성 | MT-Unsafe const:env |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, 4.3BSD.

## NOTES

POSIX.1에서는 `setenv()`나 `unsetenv()`가 재진입 가능이기를 요구하지 않는다.

glibc 2.2.2 전에선 `unsetenv()`의 원형이 `void`를 반환하는 것이었다. 이후 버전들에서는 SYNOPSIS에 나와 있는 POSIX.1 준수 원형을 따른다.

## BUGS

POSIX.1에서는 `name`에 '=' 문자가 있으면 `setenv()`가 `EINVAL` 오류로 실패해야 한다고 명세한다. 하지만 glibc 버전 2.3.4 전에선 `name`에 '=' 부호를 허용했다.

## SEE ALSO

<tt>[[clearenv(3)]]</tt>, <tt>[[getenv(3)]]</tt>, <tt>[[putenv(3)]]</tt>, <tt>[[environ(7)]]</tt>

----

2021-03-22
