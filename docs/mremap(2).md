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

`flags` 비트 마스크 인자는 0이거나 다음 플래그를 포함할 수 있다.

`MREMAP_MAYMOVE`
:   기본적으로 현재 위치에서 매핑을 확장할 충분한 공간이 없으면 `mremap()`이 실패한다. 이 플래그를 지정하면 필요시 커널에서 매핑을 재배치하는 것을 허용한다. 매핑이 재배치되면 이전 매핑 위치에 대한 절대 포인터가 유효하지 않게 된다. (매핑 시작 주소를 기준으로 한 오프셋을 쓰는 게 좋다.)

`MREMAP_FIXED` (리눅스 2.3.31부터)
:   이 플래그는 <tt>[[mmap(2)]]</tt>의 `MAP_FIXED` 플래그와 비슷한 역할을 한다. 이 플래그를 지정하면 `mremap()`이 다섯 번째 인자 `void *new_address`를 받는데 그 인자는 매핑을 이동시킬 페이지 정렬 주소를 나타낸다. `new_address`와 `new_size`로 지정한 주소 범위에 기존 매핑이 있으면 맵을 해제한다.

    `MREMAP_FIXED`를 지정하는 경우 `MREMAP_MAYMOVE`도 함께 지정해야 한다.

`MREMAP_DONTUPMAP` (리눅스 5.7부터)
:   이 플래그는 `MREMAP_MAYMOVE`와 함께 사용해야 하며, 매핑을 새 주소로 재사상하되 `old_address`의 맵을 해제하지 않는다.

    `MREMAP_DONTUNMAP` 플래그는 비공유 익명 매핑에만 쓸 수 있다. (<tt>[[mmap(2)]]</tt>의 `MAP_PRIVATE` 및 `MAP_ANONYMOUS` 설명 참고.)

    완료 후에는 `old_address`와 `old_size`로 지정한 범위에 접근 시 페이지 폴트가 발생한다. 그 주소가 앞서 <tt>[[userfaultfd(2)]]</tt>로 등록해 둔 범위 안에 있다면 <tt>[[userfaultfd(2)]]</tt>로 그 페이지 폴트를 처리하게 된다. 아니라면 커널에서 0으로 채운 페이지를 할당해서 그 폴트를 처리한다.

    `MREMAP_DONTUNMAP` 플래그를 사용하면 원본의 매핑을 유지한 채 원자적으로 매핑을 옮길 수 있다. `MREMAP_DONTUNMAP`을 이용할 만한 곳은 NOTES 참고.

`old_address`와 `old_size`로 지정한 메모리 세그먼트가 (<tt>[[mlock(2)]]</tt> 등으로) 잠겨 있으면 세그먼트 크기 조정 및/또는 재배치 과정에서 그 잠김이 유지된다. 그에 따라 프로세스에 의해 잠긴 메모리 양이 바뀔 수 있다.

## RETURN VALUE

성공 시 `mremap()`은 새 가상 메모리 영역에 대한 포인터를 반환한다. 오류 시 `MAP_FAILED`(즉 `(void *) -1`) 값을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EAGAIN`
:   호출자가 잠긴 메모리 세그먼트를 확장하려 했는데 `RLIMIT_MEMLOCK` 자원 제한을 초과하지 않고는 불가능하다.

`EFAULT`
:   `old_address`에서 `old_address+old_size`까지 범위 내의 어떤 주소가 이 프로세스에게 유효하지 않은 가상 메모리 주소이다. 요청한 주소 공간 전체를 포괄하는 매핑들이 존재하지만 그 매핑들이 서로 다른 유형인 경우에도 `EFAULT`를 받을 수 있다.

`EINVAL`
:   유효하지 않은 인자를 주었다. 가능한 원인은 다음과 같다.

    * `old_address`가 페이지에 정렬되어 있지 않다.
    * `flags`에 `MREMAP_MAYMOVE`와 `MREMAP_FIXED`와 `MREMAP_DONTUNMAP` 외의 값을 지정했다.
    * `new_size`가 0이다.
    * `new_size`나 `new_address`가 유효하지 않다.
    * `new_address`와 `new_size`로 지정한 새 주소 범위가 `old_address`와 `old_size`로 지정한 이전 주소 범위와 겹친다.
    * `MREMAP_FIXED`나 `MREMAP_DONTUNMAP`을 지정하면서 `MREMAP_MAYMOVE`를 함께 지정하지 않았다.
    * `MREMAP_DONTUNMAP`을 지정했는데 `old_address`와 `old_size`로 지정한 범위에 있는 한 개 이상 페이지가 비공유 익명이 아니다.
    * `MREMAP_DONTUNMAP`을 지정했는데 `old_size`가 `new_size`와 같지 않다.
    * `old_size`가 0인데 `old_address`가 공유 가능 매핑을 가리키고 있지 않다. (하지만 BUGS 참고.)
    * `old_size`가 0인데 `MREMAP_MAYMOVE` 플래그를 지정하지 않았다.

`ENOMEM`
:   메모리가 부족해서 동작을 완료할 수 없다. 가능한 원인은 다음과 같다.

    * 현재 가상 주소에서 메모리 영역을 확장할 수 없으며 `flags`에 `MREMAP_MAYMOVE`가 설정돼 있지 않다. 또는 쓸 수 있는 (가상) 메모리가 충분치 않다.

    * `MREMAP_DONTUNMAP`을 써서 새 매핑을 생성해야 하는데 가용 (가상) 메모리를 초과하게 된다. 또는 허용되는 최대 매핑 수를 초과하게 된다.

## CONFORMING TO

이 호출은 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

`mremap()`은 가상 주소와 메모리 페이지 간의 매핑을 바꾼다. 이를 이용하면 아주 효율적인 <tt>[[realloc(3)]]</tt>을 구현할 수 있다.

리눅스에서 메모리는 페이지들로 나눠져 있다. 프로세스에는 (한 개 또는) 여러 개의 선형 가상 메모리 세그먼트가 있다. 각 가상 메모리 세그먼트에는 실제 메모리 페이지에 대한 한 개 또는 그 이상의 매핑이 (페이지 테이블에) 있다. 가상 메모리 세그먼트에는 각자의 보호 방식(접근권)이 있어서 그 메모리에 잘못된 방식으로 접근하면 (가령 읽기 전용 세그먼트에 쓰기) 세그먼테이션 위반이 발생할 수 있다. 세그먼트 밖의 가상 메모리에 접근해도 세그먼테이션 위반이 발생하게 된다.

<tt>[[mlock(2)]]</tt> 등으로 잠근 영역을 `mremap()`으로 이동 내지 확장하는 경우에 `mremap()` 호출에서는 메모리에 새 영역을 채우려고 최선을 다하되 채우는 데 실패한 경우에 `ENOMEM`으로 실패하지 않는다.

glibc 버전 2.4 전에서는 `MREMAP_FIXED` 정의를 노출하지 않았으며 `mremap()` 원형에서 `new_address` 인자를 허용하지 않았다.

### `MREMAP_DONTUNMAP` 사용례

`MREMAP_DONTUNMAP`을 이용할 만한 곳은 다음과 같다.

* 비협력적 <tt>[[userfaultfd(2)]]</tt>: 응용에서 `MREMAP_DONTUNMAP`으로 어떤 가상 주소 범위를 빼낸 다음 <tt>[[userfaultfd(2)]]</tt> 핸들러를 사용해서 프로세스의 다른 스레드가 그 빠진 범위의 페이지를 건드려서 발생하는 페이지 폴트를 처리할 수 있다.

* 쓰레기 수집: `MREMAP_DONTUNMAP`을 <tt>[[userfaultfd(2)]]</tt>와 함께 써서 (자바 가상 머신에 있는 것 같은) 쓰레기 수집 알고리즘을 구현할 수 있다. 그런 구현 방식은 페이지에 `PROT_NONE` 보호 표시를 하고 `SIGSEGV` 핸들러로 그 페이지에 대한 접근을 잡는 전통적인 쓰레기 수집 기법들보다 비용이 낮을 (그리고 단순할) 수 있다.

## BUGS

리눅스 4.14 전에서는 `old_size`가 0이고 `old_address`가 가리키는 매핑이 비공유 매핑(<tt>[[mmap(2)]]</tt> `MAP_PRIVATE`)이면 `mremap()`이 원래 매핑과 무관한 새 비공유 매핑을 생성했다. 그 동작 방식은 의도하지 않은 것이며 아마 사용자 공간 응용에게도 (`mremap()`의 목적이 원래 매핑을 바탕으로 새 매핑을 만드는 것이므로) 예상치 못한 동작이었을 것이다. 리눅스 4.14부터는 그 경우에 `mremap()`이 `EINVAL` 오류로 실패한다.

## SEE ALSO

<tt>[[brk(2)]]</tt>, <tt>[[getpagesize(2)]]</tt>, <tt>[[getrlimit(2)]]</tt>, <tt>[[mlock(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[sbrk(2)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[realloc(3)]]</tt>

메모리 페이징에 대한 자세한 내용은 좋아하는 운영 체제 교과서(가령 Andrew S. Tanenbaum의 *Modern Operating System*, Randolph Bentson의 *Inside Linux*, Maurice J. Bach의 *The Design of the UNIX Operating System*)를 보라.

----

2021-03-22
