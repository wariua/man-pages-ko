## NAME

posix_memalign, aligned_alloc, memalign, valloc, pvalloc - 정렬된 메모리 할당하기

## SYNOPSIS

```c
#include <stdlib.h>

int posix_memalign(void **memptr, size_t alignment, size_t size);
void *aligned_alloc(size_t alignment, size_t size);
void *valloc(size_t size);

#include <malloc.h>

void *memalign(size_t alignment, size_t size);
void *pvalloc(size_t size);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>posix_memalign()</code>:</dt>
<dd><code>_POSIX_C_SOURCE >= 200112L</code></dd>

<dt><code>aligned_alloc()</code>:</dt>
<dd><code>_ISOC11_SOURCE</code></dd>

<dt><code>valloc()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.12부터:</dt>
 <dd>
<code>(_XOPEN_SOURCE >= 500) && !(_POSIX_C_SOURCE >= 200112L)</code><br>
<code>    || /* glibc 2.19부터: */ _DEFAULT_SOURCE</code><br>
<code>    || /* glibc 버전 <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE</code>
 </dd>
 <dt>glibc 2.12 전:</dt>
 <dd>
<code>_BSD_SOURCE || _XOPEN_SOURCE >= 500</code>>><br>
((비표준) 헤더 파일 <code>&lt;malloc.h&gt;</code>도 <code>valloc()</code> 선언을 드러낸다. 어떤 기능 확인 매크로도 필요하지 않다.)
 </dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`posix_memalign()` 함수는 `size` 바이트를 할당하여 할당한 메모리의 주소를 `*memptr`에 넣는다. 할당한 메모리의 주소가 `alignment`의 배수가 된다. `alignment`는 2의 거듭제곱이어야 하며 `sizeof(void *)`의 배수여야 한다. `size`가 0인 경우 `*memptr`에 넣는 값은 NULL이거나, 이후 <tt>[[free(3)]]</tt>에 무사히 전달할 수 있는 고유한 포인터 값이다.

구식 함수 `memalign()`은 `size` 바이트를 할당하여 할당된 메모리에 대한 포인터를 반환한다. 메모리 주소가 `alignment`의 배수가 된다. `alignment`는 2의 거듭제곱이어야 한다.

`aligned_alloc()` 함수는 `memalign()`과 동일하되 `size`가 `alignment`의 배수여야 한다는 제약이 추가된다.

구식 함수 `valloc()`은 `size` 바이트를 할당하여 할당된 메모리에 대한 포인터를 반환한다. 메모리 주소가 페이지 크기의 배수가 된다. `memalign(sysconf(_SC_PAGESIZE),size)`와 동등하다.

구식 함수 `pvalloc()`은 `valloc()`과 유사하되 할당 크기를 시스템 페이지 크기의 배수로 올림 한다.

이 함수들 모두 메모리를 0으로 채우지 않는다.

## RETURN VALUE

성공 시 `aligned_alloc()`, `memalign()`, `valloc()`, `pvalloc()`은 할당한 메모리에 대한 포인터를 반환한다. 오류 시 NULL을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다.

`posix_memalign()`은 성공 시 0을 반환하고 실패 시 다음 절에 나열한 오류 값들 중 하나를 반환한다. `errno` 값은 설정하지 않는다. 리눅스에서는 (그리고 다른 시스템들에서는) 실패 시 `posix_memalign()`이 `memptr`을 변경하지 않는다. 이런 동작 방식을 표준화한 요구 사항이 POSIX.1-2016에 추가되었다.

## ERRORS

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>alignment</code> 인자가 2의 거듭제곱이 아니거나 <code>sizeof(void *)</code>의 배수가 아니다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>할당 요청을 충족할 충분한 메모리가 없다.</dd>
</dl>

## VERSIONS

리눅스의 모든 libc 라이브러리들에서 `memalign()`, `valloc()`, `pvalloc()` 함수가 늘 사용 가능했다.

glibc 버전 2.16에서 `aligned_alloc()` 함수가 추가되었다.

glibc 2.1.91부터 `posix_memalign()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `aligned_alloc()`,<br>`memalign()`,<br>`posix_memalign()` | 스레드 안전성 | MT-Safe |
| `valloc()`,<br>`pvalloc()` | 스레드 안전성 | MT-Unsafe init |

## CONFORMING TO

`valloc()` 함수는 3.0BSD에서 등장했다. 4.3BSD에서 구식으로, 그리고 SUSv2에서 legacy로 적고 있다. POSIX.1에는 등장하지 않는다.

`pvalloc()` 함수는 GNU 확장이다.

`memalign()` 함수는 SunOS 4.1.3에서 등장하지만 4.4BSD에는 없다.

`posix_memalign()`은 POSIX.1d에서 온 것이며 POSIX.1-2001 및 POSIX.1-2008에 명세되어 있다.

`aligned_alloc()` 함수는 C11 표준에 명세되어 있다.

### 헤더

`posix_memalign()`이 `<stdlib.h>`에 선언되어 있다는 것에는 모두가 동의한다.

일부 시스템에서는 `memalign()`이 `<malloc.h>`가 아니라 `<stdlib.h>`에 선언되어 있다.

SUSv2에 따르면 `valloc()`은 `<stdlib.h>`에 선언되어 있다. libc 4와 5, 그리고 glibc에서는 `<malloc.h>`에서 선언하며, 적절한 기능 확인 매크로가 정의되어 있으면 (위 참고) `<stdlib.h>`에서도 선언한다.

## NOTES

여러 시스템에서 가령 블록 장치 직접 I/O에 쓰는 버퍼들에 정렬 제약이 있다. POSIX에서는 어떤 정렬이 필요한지 알려 주는 `pathconf(path,_PC_REC_XFER_ALIGN)` 호출을 명세한다. 그러면 `posix_memalign()`을 사용해 이런 요구 사항을 충족시킬 수 있다.

`posix_memalign()`은 `alignment`가 위에 상술한 요구 사항에 맞는지 검사한다. `memalign()`은 `alignment` 인자가 올바른지 검사하지 않을 수도 있다.

POSIX에서는 `posix_memalign()`에게서 얻은 메모리를 <tt>[[free(3)]]</tt>로 해제할 수 있어야 한다고 요구한다. 일부 시스템에서는 `memalign()`이나 `valloc()`으로 할당한 메모리를 회수하기 위한 방법을 제공하지 않는다. (<tt>[[malloc(3)]]</tt>에게서 얻은 포인터만 <tt>[[free(3)]]</tt>에 줄 수 있는데 가령 `memalign()`에서 <tt>[[malloc(3)]]</tt>을 호출해서 얻은 값을 정렬하게 되기 때문이다.) glibc 구현에서는 위 함수들 중 무엇으로 얻은 메모리든지 <tt>[[free(3)]]</tt>로 회수할 수 있다.

glibc의 <tt>[[malloc(3)]]</tt>은 항상 8바이트에 정렬된 메모리 주소를 반환한다. 따라서 더 큰 정렬 값이 필요할 때만 이 함수들을 쓰면 된다.

## SEE ALSO

<tt>[[brk(2)]]</tt>, <tt>[[getpagesize(2)]]</tt>, <tt>[[free(3)]]</tt>, <tt>[[malloc(3)]]</tt>

----

2019-05-09
