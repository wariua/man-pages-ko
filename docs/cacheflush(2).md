## NAME

cacheflush - 인스트럭션 캐시 및/또는 데이터 캐시의 내용물 비우기

## SYNOPSIS

```c
#include <asm/cachectl.h>

int cacheflush(char *addr, int nbytes, int cache);
```

## DESCRIPTION

`cacheflush()`는 `addr`에서 `(addr+nbytes-1)`까지 범위에 대해 표시한 캐시(들)의 내용물을 비운다. `cache`는 다음 중 하나일 수 있다.

<dl>
<dt><code>ICACHE</code></dt>
<dd>인스트럭션 캐시를 비운다.</dd>

<dt><code>DCACHE</code></dt>
<dd>변경을 메모리로 기록하고 영향 받는 유효한 캐시 라인들을 무효화한다.</dd>

<dt><code>BCACHE</code></dt>
<dd><code>(ICACHE|DCACHE)</code>와 같다.</dd>
</dl>

## RETURN VALUE

`cacheflush()`는 성공 시 0을 반환하거나 오류 시 -1을 반환한다. 오류를 탐지한 경우 `errno`가 오류를 나타내게 된다.

## ERRORS

<dl>
<dt><code>EFAULT</code></dt>
<dd><code>addr</code>에서 <code>(addr+nbytes-1)</code>까지의 주소 범위 일부 내지 전부가 접근 가능하지 않다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>cache</code>가 <code>ICACHE</code>, <code>DCACHE</code>, <code>BCACHE</code> 중 하나가 아니다. (하지만 BUGS 참고.)</dd>
</dl>

## CONFORMING TO

역사적으로 이 시스템 호출은 RISC/os, IRIX, Ultrix, NetBSD, OpenBSD, FreeBSD를 포함한 모든 MIPS 유닉스 변종들에서 (그리고 일부 비유닉스 MIPS 운영 체제들에서도) 사용 가능했다. 그래서 MIPS 운영 체제들에서는 이 호출의 존재가 사실상 표준이다.

### 경고

이식 가능해야 하는 프로그램에서는 `cacheflush()`를 사용하지 말아야 한다. 리눅스에서 이 호출은 MIPS 아키텍처에서 처음 등장했다. 요즘은 일부 다른 아키텍처들에서도 `cacheflush()` 시스템 호출을 제공하지만 인자가 다르다.

## BUGS

2.6.11보다 오래된 리눅스 커널에서는 `addr`과 `nbytes` 인자를 무시하며, 그래서 이 함수 비용이 꽤 비싸진다. 그 때문에 항상 캐시 전체를 비운다.

이 함수는 항상 `cache` 인자로 `BCACHE`를 전달받은 것처럼 동작하며 `cache` 인자에 대해 어떤 오류 검사도 하지 않는다.

----

2017-09-15
