## NAME

mktemp - 유일한 임시 파일명 만들기

## SYNOPSIS

```c
#include <stdlib.h>

char *mktemp(char *template);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`mktemp()`:
:   glibc 2.12부터:
    :   `(_XOPEN_SOURCE >= 500) && ! (_POSIX_C_SOURCE >= 200112L)`<br>
        `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
        `    || /* glibc <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE`
 
    glibc 2.12 전:
    :   `_BSD_SOURCE || _SVID_SOURCE || _XOPEN_SOURCE >= 500`

## DESCRIPTION

*이 함수를 절대 쓰지 말 것.* BUGS 참고.

`mktemp()` 함수는 `template`을 가지고 유일한 임시 파일명을 만들어 낸다. `template`의 마지막 여섯 글자가 XXXXXX여야 하며 그 글자들을 바꿔서 유일한 파일명을 만든다. 변경이 이뤄지므로 `template`이 문자열 상수여서는 안 되며 문자 배열로 선언하는 게 좋다.

## RETURN VALUE

`mktemp()` 함수는 항상 `template`을 반환한다. 유일한 이름을 생성했다면 결과 이름이 유일하도록 (즉 그 이름이 이미 존재하지 않도록) `template`의 마지막 여섯 바이트가 변경되어 있을 것이다. 유일한 이름을 생성할 수 없었다면 `template`을 빈 문자열로 만들고 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `template`의 마지막 여섯 글자가 XXXXXX가 아니다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mktemp()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.3BSD, POSIX.1-2001. POSIX-1.2008에서 `mktemp()` 명세가 없어졌다.

## BUGS

`mktemp()`를 절대 쓰지 마라. 어떤 구현들에서는 4.3BSD를 따라 XXXXXX를 현재 프로세스 ID와 글자 하나로 바꾼다. 그래서 최대 26개까지만 다른 이름을 반환할 수 있다. 일단 이름을 추측하는 게 쉽고 다른 한편으로 이름 존재 여부 검사와 파일 열기 사이에 경쟁이 있기 때문에 `mktemp()` 사용은 모두 보안 위험 요소다. <tt>[[mkstemp(3)]]</tt>와 <tt>[[mkdtemp(3)]]</tt>로 그 경쟁을 피한다.

## SEE ALSO

`mktemp(1)`, <tt>[[mktemp(3)]]</tt>, <tt>[[mkstemp(3)]]</tt>, <tt>[[tempnam(3)]]</tt>, <tt>[[tmpfile(3)]]</tt>, <tt>[[tmpnam(3)]]</tt>

----

2017-09-15
