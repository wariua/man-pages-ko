## NAME

brk, sbrk - 데이터 세그먼트 크기 바꾸기

## SYNOPSIS

```c
#include <unistd.h>

int brk(void *addr);
void *sbrk(intptr_t increment);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`brk()`, `sbrk()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`<br>
        `    || ((_XOPEN_SOURCE >= 500) &&`<br>
        `        ! (_POSIX_C_SOURCE >= 200112L))`

    glibc 2.12부터 2.19까지:
    :   `_BSD_SOURCE || _SVID_SOURCE`<br>
        `    || ((_XOPEN_SOURCE >= 500) &&`<br>
        `        ! (_POSIX_C_SOURCE >= 200112L))`

    glibc 2.12 전:
    :   `_BSD_SOURCE || _SVID_SOURCE || _XOPEN_SOURCE >= 500`

## DESCRIPTION

`brk()`와 `sbrk()`는 *프로그램 단절점(program break)*의 위치를 바꾼다. 프로그램 단절점은 프로세스의 데이터 세그먼트 끝을 규정한다. (즉, 프로그램 단절점은 비초기화 데이터 세그먼트의 끝 다음의 첫 위치다.) 프로그램 단절점을 올리면 프로세스에게 메모리를 할당하는 효과가 있고 단절점을 내리면 메모리를 해제하는 것이다.

`brk()`는 데이터 세그먼트의 끝을 `addr`에 지정한 값으로 설정한다. 단, 그 값이 적당하고, 시스템에 충분한 메모리가 있고, 프로세스의 최대 데이터 크기(<tt>[[setrlimit(2)]]</tt> 참고)를 초과하지 않아야 한다.

`sbrk()`는 프로그램의 데이터 공간을 `increment` 바이트만큼 올린다. `increment`를 0으로 해서 `sbrk()`를 호출하면 프로그램 단절점의 현재 위치를 알아낼 수 있다.

## RETURN VALUE

`brk()`는 성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 `ENOMEM`으로 설정한다.

`sbrk()`는 성공 시 이전 프로그램 단절점을 반환한다. (단절점이 올라갔다면 이 값이 새로 할당된 메모리 시작점에 대한 포인터다.) 오류 시 `(void *) -1`을 반환하며 `errno`를 `ENOMEM`으로 설정한다.

## CONFORMING TO

4.3BSD, SUSv1. SUSv2에서 LEGACY로 표시됨. POSIX.1-2001에서 제거됨.

## NOTES

`brk()`와 `sbrk()` 사용을 피하라. <tt>[[malloc(3)]]</tt> 메모리 할당 패키지가 메모리를 할당하는 이식성 있고 편리한 방식이다.

시스템마다 `sbrk()` 인자에 다양한 타입을 쓴다. 흔하게는 `int`, `ssize_t`, `ptrdiff_t`, `intptr_t`가 있다.

### C 라이브러리/커널 차이

위에서 기술한 `brk()` 반환 값은 리눅스 `brk()` 시스템 호출을 위한 glibc 래퍼 함수가 제공하는 동작 방식이다. (다른 대부분의 구현에서도 `brk()` 반환 값이 그와 같다. 이 반환 값이 SUSv2에 명세되기도 했다.) 하지만 실제 리눅스 시스템 호출은 성공 시 새로운 프로그램 단절점을 반환한다. 그리고 실패 시 현재 단절점을 반환한다. glibc 래퍼 함수에서 약간의 작업을 해서 (즉 새 단절점이 `addr`보다 작은지 확인해서) 위에 기술한 0과 -1 반환 값을 제공한다.

리눅스에서 `sbrk()`는 `brk()` 시스템 호출을 사용하는 라이브러리 함수로 구현돼 있으며, 내부적으로 어떤 상태 관리를 해서 이전 단절점 값을 반환할 수 있다.

## SEE ALSO

<tt>[[execve(2)]]</tt>, <tt>[[getrlimit(2)]]</tt>, <tt>[[end(3)]]</tt>, <tt>[[malloc(3)]]</tt>

----

2021-03-22
