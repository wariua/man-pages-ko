## NAME

mkstemp, mkostemp, mkstemps, mkostemps - 유일한 임시 파일 만들기

## SYNOPSIS

```c
#include <stdlib.h>

int mkstemp(char *template);
int mkostemp(char *template, int flags);
int mkstemps(char *template, int suffixlen);
int mkostemps(char *template, int suffixlen, int flags);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`mkstemp()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200809L`<br>
    `    || /* glibc <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE`

`mkostemp()`:
:   `_GNU_SOURCE`

`mkstemps()`:
:   `/* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE`

`mkostemps()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`mkstemp()` 함수는 `template`을 가지고 유일한 임시 파일명을 만들어 내고, 그 파일을 생성해서 열고, 그 파일에 대한 열린 파일 디스크립터를 반환한다.

`template`의 마지막 여섯 글자가 "XXXXXX"여야 하며 그 글자들을 바꿔서 파일명을 유일하게 만든다. 변경이 이뤄지므로 `template`이 문자열 상수여서는 안 되며 문자 배열로 선언하는 게 좋다.

0600 권한으로 파일을 생성한다. 즉 소유자에게만 읽기 및 쓰기 권한이 있다. 반환되는 파일 디스크립터를 통해 파일에 읽기 및 쓰기 접근을 할 수 있다. <tt>[[open(2)]]</tt>의 `O_EXCL` 플래그를 써서 파일을 열기 때문에 호출자가 파일을 생성한 프로세스라는 게 보장된다.

`mkostemp()` 함수는 `mkstemp()`와 비슷하되 `flags`에 (<tt>[[open(2)]]</tt>에서와 의미가 같은) 비트 `O_APPEND`, `O_CLOEXEC`, `O_SYNC`를 지정할 수 있다는 점이 다르다. 참고로 `mkostemp()`에서 파일을 만들 때 `flags` 인자에 `O_RDWR`, `O_CREAT`, `O_EXCL` 값을 포함시켜서 <tt>[[open(2)]]</tt>에 준다. 따라서 `mkostemp()`의 `flags` 인자에 그 값들을 포함시키는 것은 불필요하며 어떤 시스템에서는 오류가 발생한다.

`mkstemps()` 함수는 `mkstemp()`와 비슷하되 `template`의 문자열에 `suffixlen` 개 문자만큼의 접미부가 있다는 점이 다르다. 즉 `template`이 `prefixXXXXXXsuffix` 형태이며 `mkstemp()`에서처럼 문자열 XXXXXX가 변경된다.

`mkostemps()` 함수와 `mkstemps()`의 관계는 `mkostemp()`와 `mkstemp()`의 관계와 같다.

## RETURN VALUE

성공 시 이 함수들은 임시 파일의 파일 디스크립터를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EEXIST`
:   유일한 임시 파일명을 만들 수 없다. 이때 `template`의 내용은 규정돼 있지 않다.
`EINVAL`
:   `mkstemp()` 및 `mkostemp()`: `template`의 마지막 여섯 글자가 XXXXXX가 아니다. 이때 `template`은 바뀌지 않는다.

    `mkstemps()` 및 `mkostemps()`: `template`의 길이가 `(6 + suffixlen)` 글자보다 작거나 `template`의 접미부 앞 마지막 6 글자가 XXXXXX가 아니다.

이 함수들이 <tt>[[open(2)]]</tt>에서 기술하는 오류들 때문에 실패할 수도 있다.

## VERSIONS

glibc 2.7부터 `mkostemp()`가 사용 가능하다. glibc 2.11부터 `mkstemps()`와 `mkostemps()`가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mkstemp()`, `mkostemp()`, `mkstemps()`, `mkostemps()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`mkstemp()`: 4.3BSD, POSIX.1-2001.

`mkstemps()`: 표준화 안 됨. 하지만 다른 여러 시스템에 있음.

`mkostemp()` 및 `mkostemps()`: glibc 확장임.

## NOTES

glibc 버전 2.06 및 이전에서는 0666 권한, 즉 모든 사용자 읽기 및 쓰기로 파일을 생성한다. 이 구식 동작 방식이 보안 위험 요소일 수도 있다. 다른 유닉스 계열에서는 0600을 사용하는데 프로그램을 포팅 할 때 이 세부 사항을 간과할 수도 있기 때문이다. 0600 모드로 파일을 생성해야 한다는 요구 사항이 POSIX.1-2008에 추가됐다.

더 일반적으로 `mkstemp()`의 POSIX 명세에서는 파일 모드에 대해 아무것도 언급하지 않으므로 응용에서 `mkstemp()`를 (또는 `mkostemp()`를) 호출하기 전에 파일 모드 생성 마스크(<tt>[[umask(2)]]</tt> 참고)가 적절히 설정돼 있도록 해야 할 것이다.

## SEE ALSO

<tt>[[mkdtemp(3)]]</tt>, <tt>[[mktemp(3)]]</tt>, <tt>[[tempnam(3)]]</tt>, <tt>[[tmpfile(3)]]</tt>, <tt>[[tmpnam(3)]]</tt>

----

2021-03-22
