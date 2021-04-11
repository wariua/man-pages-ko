## NAME

confstr - 구성에 따라 달라지는 문자열 변수 얻기

## SYNOPSIS

```c
#include <unistd.h>

size_t confstr(int name, char *buf, size_t len);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`confstr()`:
:   `_POSIX_C_SOURCE >= 2 || _XOPEN_SOURCE`

## DESCRIPTION

`confstr()`은 구성에 따라 달라지는 문자열 변수의 값을 얻는다.

`name` 인자는 질의하려는 시스템 변수이다. 다음 변수들을 지원한다.

`_CS_GNU_LIBC_VERSION` (GNU C 라이브러리 전용. glibc 2.3.2부터)
:   이 시스템의 GNU C 라이브러리 버전을 나타내는 문자열. (가령 "glibc 2.3.4")

`_CS_GNU_LIBPTHREAD_VERSION` (GNU C 라이브러리 전용. glibc 2.3.2부터)
:   이 C 라이브러리에서 제공하는 POSIX 구현을 나타내는 문자열. (가령 "NPTL 2.3.4"나 "linuxthreads-0.10")

`_CS_PATH`
:   모든 POSIX.2 표준 유틸리티들을 찾을 수 있는 곳을 나타내는 `PATH` 변수를 위한 값.

`buf`가 NULL이 아니고 `len`이 0이 아니면 `confstr()`이 문자열 값을 `buf`로 복사하는데, 필요시 `len - 1` 바이트로 자르고 널 바이트(`'\0'`)로 끝낸다. `confstr()` 반환 값을 `len`과 비교하여 이 경우를 탐지할 수 있다.

`len`이 0이고 `buf`가 NULL이면 `confstr()`이 아래 정의된 대로 값을 반환하기만 한다.

## RETURN VALUE

`name`이 유효한 구성 변수이면 `confstr()`이 그 변수의 값 전체를 담기 위해 필요한 (종료용 널 바이트를 포함한) 바이트 수를 반환한다. 이 값이 `len`보다 클 수도 있는데, 이는 `buf`의 값이 잘려진 것임을 뜻한다.

`name`이 유효한 구성 변수이지만 그 변수에 값이 없으면 `confstr()`이 0을 반환한다. `name`이 유효한 구성 변수에 해당하지 않으면 `confstr()`이 0을 반환하며 `errno`를 `EINVAL`로 설정한다.

## ERRORS

`EINVAL`
:   `name`의 값이 유효하지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `confstr()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## EXAMPLES

다음 코드 조각은 POSIX.2 시스템 유틸리티들을 찾을 경로를 알아낸다.

```c
char *pathbuf;
size_t n;

n = confstr(_CS_PATH, NULL, (size_t) 0);
pathbuf = malloc(n);
if (pathbuf == NULL)
    abort();
confstr(_CS_PATH, pathbuf, n);
```

## SEE ALSO

`getconf(1)`, `sh(1)`, <tt>[[exec(3)]]</tt>, <tt>[[fpathconf(3)]]</tt>, <tt>[[pathconf(3)]]</tt>, <tt>[[sysconf(3)]]</tt>, <tt>[[system(3)]]</tt>

----

2021-03-22
