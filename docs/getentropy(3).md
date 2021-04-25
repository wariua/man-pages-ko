## NAME

getentropy - 버퍼를 난수 바이트로 채우기

## SYNOPSIS

```c
#include <unistd.h>

int getentropy(void *buffer, size_t length);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getentropy()`:
:   `_DEFAULT_SOURCE`

## DESCRIPTION

`getentropy()` 함수는 `buffer`가 가리키는 위치에서 시작하는 버퍼에 `length` 개 바이트의 고품질 난수 데이터를 써넣는다. `length`에 허용되는 가장 큰 값은 256이다.

성공적인 `getentropy()` 호출은 항상 요청한 바이트 수만큼의 엔트로피를 제공한다.

## RETURN VALUE

성공 시 이 함수는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `buffer` 및 `length`로 지정한 버퍼의 일부 내지 전체가 유효한 접근 가능 메모리 내에 있지 않다.

`EIO`
:   `length`가 256보다 크다.

`EIO`
:   `buffer`를 난수 데이터로 덮어쓰려는 동안 명세되어 있지 않은 오류가 발생했다.

`ENOSYS`
:   이 함수 구현에 필요한 <tt>[[getrandom(2)]]</tt> 시스템 호출을 이 커널 버전에서 구현하고 있지 않다.

## VERSIONS

glibc 2.25에서 `getentropy()` 함수가 처음 등장했다.

## CONFORMING TO

이 함수는 비표준이다. OpenBSD에도 이 함수가 있다.

## NOTES

`getentropy()` 함수는 <tt>[[getrandom(2)]]</tt>을 이용해 구현되어 있다.

glibc 래퍼가 <tt>[[getrandom(2)]]</tt>를 취소점으로 만들지만 `getentropy()`는 취소점이 아니다.

`getentropy()`는 `<sys/random.h>`에도 선언되어 있다. (이 헤더 파일에서 선언을 얻는 데에는 어떤 기능 확인 매크로도 정의되어 있을 필요가 없다.)

시스템이 방금 부팅하여 커널이 엔트로피 풀을 초기화 할 난수성을 아직 충분히 수집하지 못했다면 `getentropy()` 호출이 블록 할 수도 있다. 이 경우에는 시그널을 처리하여도 계속 블록 하게 되며 엔트로피 풀이 초기화 되고 나서야 반환하게 된다.

## SEE ALSO

<tt>[[getrandom(2)]]</tt>, <tt>[[urandom(4)]]</tt>, <tt>[[random(7)]]</tt>

----

2021-03-22
