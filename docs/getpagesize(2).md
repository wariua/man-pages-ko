## NAME

getpagesize - 메모리 페이지 크기 얻기

## SYNOPSIS

```c
#include <unistd.h>

int getpagesize(void);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getpagesize()`:
:   glibc 2.20부터:
    :   `_DEFAULT_SOURCE || ! (_POSIX_C_SOURCE >= 200112L)`

    glibc 2.12부터 2.19까지:
    :   `_BSD_SOURCE || ! (_POSIX_C_SOURCE >= 200112L)`

    glibc 2.12 전:
    :   `_BSD_SOURCE || _XOPEN_SOURCE >= 500`

## DESCRIPTION

`getpagesize()` 함수는 메모리 페이지의 바이트 수를 반환한다. 여기서 "페이지"는 고정 길이의 블록으로 메모리 할당과 <tt>[[mmap(2)]]</tt>이 수행하는 파일 매핑의 단위이다.

## CONFORMING TO

SVr4, 4.4BSD, SUSv2. SUSv2에서 `getpagesize()` 호출이 LEGACY로 표시되었으며, POSIX.1-2001에서 빠졌다. HP-UX에는 이 호출이 없다.

## NOTES

이식 가능한 응용에서는 `getpagesize()` 대신 `sysconf(_SC_PAGESIZE)`를 사용해야 한다.

```c
#include <unistd.h>
long sz = sysconf(_SC_PAGESIZE);
```

(대부분의 시스템에서 `_SC_PAGESIZE`와 같은 의미로 `_SC_PAGE_SIZE`를 허용한다.)

`getpagesize()`가 리눅스 시스템 호출로 존재하는지 여부는 아키텍처에 따라 다르다. 존재하는 경우 커널 심볼 `PAGE_SIZE`를 반환하는데, 이 값은 아키텍처와 머신 모델에 따라 다르다. 일반적으로 아키텍처별로 한 가지 바이너리 배포본만 두기 위해 머신 모델이 아니라 아키텍처에 따라 바이너리를 다르게 사용한다. 이게 뜻하는 바는 적어도 머신 모델 의존성이 존재하는 (sun4 같은) 아키텍처에서는 사용자 프로그램에서 컴파일 시점에 헤더 파일에서 `PAGE_SIZE`를 찾지 말고 대신 실제 시스템 호출을 사용해야 한다는 것이다. 이 점에서 glibc 2.0에는 문제가 있는데, `getpagesize()`에서 시스템 호출을 사용하지 않고 정적으로 얻은 값을 반환하기 때문이다. glibc 2.1에서는 문제가 없다.

## SEE ALSO

<tt>[[mmap(2)]]</tt>, <tt>[[sysconf(3)]]</tt>

----

2021-03-22
