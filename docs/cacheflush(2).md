## NAME

cacheflush - 인스트럭션 캐시 및/또는 데이터 캐시의 내용물 비우기

## SYNOPSIS

```c
#include <sys/cachectl.h>

int cacheflush(void *addr, int nbytes, int cache);
```

*주의*: 일부 아키텍처에는 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`cacheflush()`는 `addr`에서 `(addr+nbytes-1)`까지 범위에 대해 표시한 캐시(들)의 내용물을 비운다. `cache`는 다음 중 하나일 수 있다.

`ICACHE`
:   인스트럭션 캐시를 비운다.

`DCACHE`
:   변경을 메모리로 기록하고 영향 받는 유효한 캐시 라인들을 무효화한다.

`BCACHE`
:   `(ICACHE|DCACHE)`와 같다.

## RETURN VALUE

`cacheflush()`는 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `addr`에서 `(addr+nbytes-1)`까지의 주소 범위 일부 내지 전부가 접근 가능하지 않다.

`EINVAL`
:   `cache`가 `ICACHE`, `DCACHE`, `BCACHE` 중 하나가 아니다. (하지만 BUGS 참고.)

## CONFORMING TO

역사적으로 이 시스템 호출은 RISC/os, IRIX, Ultrix, NetBSD, OpenBSD, FreeBSD를 포함한 모든 MIPS 유닉스 변종들에서 (그리고 일부 비유닉스 MIPS 운영 체제들에서도) 사용 가능했다. 그래서 MIPS 운영 체제들에서는 이 호출의 존재가 사실상 표준이다.

### 경고

이식 가능해야 하는 프로그램에서는 `cacheflush()`를 사용하지 말아야 한다. 리눅스에서 이 호출은 MIPS 아키텍처에서 처음 등장했다. 요즘은 일부 다른 아키텍처들에서도 `cacheflush()` 시스템 호출을 제공하지만 인자가 다르다.

## NOTES

### 아키텍처별 차이

glibc에서 ARC, CSKY, MIPS, NIOS2 아키텍처에 대해 SYNOPSIS에 있는 원형으로 이 시스템 호출의 래퍼를 제공한다.

몇몇 다른 아키텍처에서는 리눅스에서 이 시스템 호출을 다른 인자로 제공한다.

M68K:
:   

        int cacheflush(unsigned long addr, int scope, int cache,
                       unsigned long len);

SH:
:   

        int cacheflush(unsigned long addr, unsigned long len, int op);

NDS32:
:   

        int cacheflush(unsigned int start, unsigned int end, int cache);

위 아키텍처들에선 glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

### GCC의 대안

이 시스템 호출에서 제공하는 세밀한 제어가 필요하지 않다면 GCC 내장 함수인 `__builtin___clear_cache()`를 이용하는 게 좋을 수도 있다. GCC 및 호환 컴파일러에서 지원하는 플랫폼 전반에서 이식성 있는 인터페이스를 제공해 준다.

```c
void __builtin___clear_cache(void *begin, void *end);
```

인스트럭션 캐시 비우기가 필요치 않은 플랫폼에선 `__builtin___clear_cache()`가 아무 효력이 없다.

*주의*: 일부 GCC 호환 컴파일러에선 이 내장 함수의 원형에서 매개변수에 `void *` 대신 `char *`를 쓴다.

## BUGS

2.6.11보다 오래된 리눅스 커널에서는 `addr`과 `nbytes` 인자를 무시하며, 그래서 이 함수 비용이 꽤 비싸진다. 그 때문에 항상 캐시 전체를 비운다.

이 함수는 항상 `cache` 인자로 `BCACHE`를 전달받은 것처럼 동작하며 `cache` 인자에 대해 어떤 오류 검사도 하지 않는다.

----

2021-03-22
