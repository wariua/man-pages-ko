## NAME

getdtablesize - 파일 디스크립터 테이블 크기 얻기

## SYNOPSIS

```c
#include <unistd.h>

int getdtablesize(void);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getdtablesize()`:
:   glibc 2.20부터:
    :   `_DEFAULT_SOURCE || ! (_POSIX_C_SOURCE >= 200112L)`

    glibc 2.12에서 2.19까지:
    :   `_BSD_SOURCE || ! (_POSIX_C_SOURCE >= 200112L)`

    glibc 2.12 전:
    :   `_BSD_SOURCE || _XOPEN_SOURCE >= 500`

## DESCRIPTION

`getdtablesize()`는 프로세스가 열 수 있는 파일 최대 개수, 즉 가능한 가장 큰 파일 디스크립터 값에 1을 더한 값을 반환한다.

## RETURN VALUE

현재의 프로세스별 열린 파일 개수 제한.

## ERRORS

리눅스에서는 `getdtablesize()`가 <tt>[[getrlimit(2)]]</tt>에 기술된 어떤 오류든 반환할 수 있다. 아래 NOTES 참고.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getdtablesize()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

SVr4, 4.4BSD. (4.2BSD에서 `getdtablesize()` 함수가 처음 등장했다.) POSIX.1에 명세되어 있지 않다. 이식 가능한 응용에서는 이 호출보다 `sysconf(_SC_OPEN_MAX)`를 이용하는 게 좋다.

## NOTES

`getdtablesize()`의 glibc 버전에서는 <tt>[[getrlimit(2)]]</tt>를 호출해서 현재 `RLIMIT_NOFILE` 제한치를 반환하며, 실패 시 `OPEN_MAX`를 반환한다.

## SEE ALSO

<tt>[[close(2)]]</tt>, <tt>[[dup(2)]]</tt>, <tt>[[getrlimit(2)]]</tt>, <tt>[[open(2)]]</tt>

----

2021-03-22
