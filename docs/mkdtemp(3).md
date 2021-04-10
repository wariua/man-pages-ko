## NAME

mkdtemp - 유일한 임시 디렉터리 만들기

## SYNOPSIS

```c
#include <stdlib.h>

char *mkdtemp(char *template);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`mkdtemp()`:
:   `/* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `|| /* glibc 2.19 및 이전: */ _BSD_SOURCE`<br>
    `|| /* glibc 2.10부터: */ _POSIX_C_SOURCE >= 200809L`

## DESCRIPTION

`mkdtemp()` 함수는 `template`을 가지고 유일한 이름의 임시 디렉터리를 만들어 낸다. `template`의 마지막 여섯 글자가 XXXXXX여야 하며 그 글자들을 바꿔서 디렉터리 이름을 유일하게 만든다. 그리고 0700 권한으로 디렉터리를 생성한다. 변경이 이뤄지므로 `template`이 문자열 상수여서는 안 되며 문자 배열로 선언하는 게 좋다.

## RETURN VALUE

`mkdtemp()` 함수는 성공 시 변경된 템플릿 문자열에 대한 포인터를 반환한다. 오류 시 NULL을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`EINVAL`
:   `template`의 마지막 여섯 글자가 XXXXXX가 아니다. 이때 `template`은 바뀌지 않는다.

`errno`에 가능한 다른 값들은 <tt>[[mkdir(2)]]</tt>을 보라.

## VERSIONS

glibc 2.1.91부터 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mkdtemp()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2008. BSD 계열에 이 함수가 있다.

## SEE ALSO

`mktemp(1)`, <tt>[[mkdir(2)]]</tt>, <tt>[[mkstemp(3)]]</tt>, <tt>[[mktemp(3)]]</tt>, <tt>[[tempnam(3)]]</tt>, <tt>[[tmpfile(3)]]</tt>, <tt>[[tmpnam(3)]]</tt>

----

2016-07-17
