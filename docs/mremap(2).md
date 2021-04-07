## NAME

mremap - 가상 메모리 주소 재사상하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <sys/mman.h>

void *mremap(void *old_address, size_t old_size,
             size_t new_size, int flags, ... /* void *new_address */);
```

## DESCRIPTION

`mremap()`은 기존 메모리 매핑을 확장(또는 축소)하며, 동시에 옮기는 것도 가능하다. (`flags` 인자와 사용 가능한 가상 주소 공간에 따라 정해진다.)

`old_address`는 확장(또는 축소)하려는 가상 메모리 블록의 이전 주소이다. `old_address`가 페이지에 정렬되어 있어야 함에 유의해야 한다. `old_size`는 그 가상 메모리 블록의 이전 크기이다. `new_size`는 그 가상 메모리 블록의 조정 후의 요청 크기이다. 선택적인 다섯 번째 인자 `new_address`를 줄 수도 있다. 아래 `MREMAP_FIXED` 설명을 보라.

`old_size`의 값이 0이고 `old_address`가 공유 가능 매핑(<tt>[[mmap(2)]]</tt> `MAP_SHARED` 참고)을 가리키는 경우에는 `mremap()`이 동일 페이지들로 새 매핑을 만들게 된다. `new_size`가 새 매핑의 크기가 되며 새 매핑의 위치를 `new_address`로 지정할 수도 있다. 아래 `MREMAP_FIXED` 설명을 보라. 이 방식으로 새 매핑을 요청하는 경우에는 `MREMAP_MAYMOVE` 플래그도 함께 지정해야 한다.

리눅스에서 메모리는 페이지들로 나눠져 있다. 사용자 프로세스에는 (한 개 또는) 여러 개의 선형 가상 메모리 세그먼트가 있다. 각 가상 메모리 세그먼트에는 실제 메모리 페이지에 대한 한 개 또는 그 이상의 매핑이 (페이지 테이블 내에) 있다. 가상 메모리 세그먼트에는 각자의 보호 방식(접근권)이 있어서 그 메모리에 잘못된 방식으로 접근하면 (가령 읽기 전용 세그먼트에 쓰기) 세그멘테이션 위반이 발생할 수 있다. 세그먼트 밖의 가상 메모리에 접근해도 세그멘테이션 위반이 발생하게 된다.

`mremap()`은 리눅스의 페이지 테이블 체계를 이용한다. `mremap()`은 가상 주소와 메모리 페이지 간의 매핑을 바꾼다. 이를 이용하면 아주 효율적인 <tt>[[realloc(3)]]</tt>을 구현할 수 있다.

`flags` 비트 마스크 인자는 0이거나 다음 플래그를 포함할 수 있다.

<dl>
<dt><code>MREMAP_MAYMOVE</code></dt>
<dd>기본적으로 현재 위치에서 매핑을 확장할 충분한 공간이 없으면 <code>mremap()</code>이 실패한다. 이 플래그를 지정하면 필요시 커널에서 매핑을 재배치하는 것을 허용한다. 매핑이 재배치되면 이전 매핑 위치에 대한 절대 포인터가 유효하지 않게 된다. (매핑 시작 주소를 기준으로 한 오프셋을 쓰는 게 좋다.)</dd>

<dt><code>MREMAP_FIXED</code> (리눅스 2.3.31부터)</dt>
<dd>이 플래그는 <tt>[[mmap(2)]]</tt>의 <code>MAP_FIXED</code> 플래그와 비슷한 역할을 한다. 이 플래그를 지정하면 <code>mremap()</code>이 다섯 번째 인자 <code>void *new_address</code>를 받는데 그 인자는 매핑을 이동시킬 페이지 정렬 주소를 나타낸다. <code>new_address</code>와 <code>new_size</code>로 지정한 주소 범위에 기존 매핑이 있으면 맵을 해제한다. <code>MREMAP_FIXED</code>를 지정하는 경우 <code>MREMAP_MAYMOVE</code>도 함께 지정해야 한다.</dd>
</dl>

`old_address`와 `old_size`로 지정한 메모리 세그먼트가 (<tt>[[mlock(2)]]</tt> 등으로) 잠겨 있으면 세그먼트 크기 조정 및/또는 재배치 과정에서 그 잠김이 유지된다. 그에 따라 프로세스에 의해 잠긴 메모리 양이 바뀔 수 있다.

## RETURN VALUE

성공 시 `mremap()`은 새 가상 메모리 영역에 대한 포인터를 반환한다. 오류 시 `MAP_FAILED`(즉 `(void *) -1`) 값을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EAGAIN</code></dt>
<dd>호출자가 잠긴 메모리 세그먼트를 확장하려 했는데 <code>RLIMIT_MEMLOCK</code> 자원 제한을 초과하지 않고는 불가능하다.</dd>
<dt><code>EFAULT</code></dt>
<dd>"세그멘테이션 폴트". <code>old_address</code>에서 <code>old_address+old_size</code>까지 범위 내의 어떤 주소가 이 프로세스에게 유효하지 않은 가상 메모리 주소이다. 요청한 주소 공간 전체를 포괄하는 매핑들이 존재하지만 그 매핑들이 서로 다른 유형인 경우에도 <code>EFAULT</code>를 받을 수 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd>

유효하지 않은 인자를 주었다. 가능한 원인은 다음과 같다.

* <code>old_address</code>가 페이지에 정렬되어 있지 않다.
* <code>flags</code>에 <code>MREMAP_MAYMOVE</code>와 <code>MREMAP_FIXED</code> 외의 값을 지정했다.
* <code>new_size</code>가 0이다.
* <code>new_size</code>나 <code>new_address</code>가 유효하지 않다.
* <code>new_address</code>와 <code>new_size</code>로 지정한 새 주소 범위가 <code>old_address</code>와 <code>old_size</code>로 지정한 이전 주소 범위와 겹친다.
* <code>MREMAP_FIXED</code>를 지정하면서 <code>MREMAP_MAYMOVE</code>를 함께 지정하지 않았다.
* <code>old_size</code>가 0인데 <code>old_address</code>가 공유 가능 매핑을 가리키고 있지 않다. (하지만 BUGS 참고.)
* <code>old_size</code>가 0인데 <code>MREMAP_MAYMOVE</code> 플래그를 지정하지 않았다.
</dd>
<dt><code>ENOMEM</code></dt>
<dd>현재 가상 주소에서 메모리 영역을 확장할 수 없으며 <code>flags</code>에 <code>MREMAP_MAYMOVE</code>가 설정돼 있지 않다. 또는 쓸 수 있는 (가상) 메모리가 충분치 않다.</dd>
</dl>

## CONFORMING TO

이 호출은 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

glibc 버전 2.4 전에서는 `MREMAP_FIXED` 정의를 노출하지 않았으며 `mremap()` 원형에서 `new_address` 인자를 허용하지 않았다.

<tt>[[mlock(2)]]</tt> 등으로 잠근 영역을 `mremap()`으로 이동 내지 확장하는 경우에 `mremap()` 호출에서는 메모리에 새 영역을 채우려고 최선을 다하되 채우는 데 실패한 경우에 `ENOMEM`으로 실패하지 않는다.

## BUGS

리눅스 4.14 전에서는 `old_size`가 0이고 `old_address`가 가리키는 매핑이 비공유 매핑(<tt>[[mmap(2)]]</tt> `MAP_PRIVATE`)이면 `mremap()`이 원래 매핑과 무관한 새 비공유 매핑을 생성했다. 그 동작 방식은 의도하지 않은 것이며 아마 사용자 공간 응용에게도 (`mremap()`의 목적이 원래 매핑을 바탕으로 새 매핑을 만드는 것이므로) 예상치 못한 동작이었을 것이다. 리눅스 4.14부터는 그 경우에 `mremap()`이 `EINVAL` 오류로 실패한다.

## SEE ALSO

<tt>[[brk(2)]]</tt>, <tt>[[getpagesize(2)]]</tt>, <tt>[[getrlimit(2)]]</tt>, <tt>[[mlock(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[sbrk(2)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[realloc(3)]]</tt>

메모리 페이징에 대한 자세한 내용은 좋아하는 운영 체제 교과서(가령 Andrew S. Tanenbaum의 <em>Modern Operating System</em>, Randolf Bentson의 <em>Inside Linux</em>, Maurice J. Bach의 <em>The Design of the UNIX Operating System</em>)를 보라.

----

2019-03-06
