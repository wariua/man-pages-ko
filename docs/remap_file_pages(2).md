## NAME

remap_file_pages - 비선형 파일 매핑 만들기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <sys/mman.h>

int remap_file_pages(void *addr, size_t size, int prot,
                     size_t pgoff, int flags);
```

## DESCRIPTION

**주의**: 이 시스템 호출은 리눅스 3.16부터 제거 예정으로 표시되었다. 리눅스 4.0에서는 그 구현이 느린 커널 내 에뮬레이션으로 대체되었다. 이 시스템 호출을 쓰는 얼마 안되는 응용들은 다른 대안으로 이전하는 걸 검토하는 게 좋다. 변경 이유는 이 시스템 호출을 위한 커널 코드가 복잡했으며 이 시스템 호출이 완전히는 아니더라도 거의 쓰이지 않는다고 보았기 때문이다. 32비트 시스템 상의 데이터베이스 응용에 몇 가지 사용 사례가 있었지만 64비트 시스템에는 그런 사용 사례가 존재하지 않는다.

`remap_file_pages()` 시스템 호출을 사용해 비선형 매핑, 즉 파일의 페이지들이 순차적이지 않은 순서로 메모리에 맵 되는 매핑을 만든다. <tt>[[mmap(2)]]</tt>을 반복 호출하는 대신 `remap_file_pages()`를 쓰는 것의 장점은 커널에서 VMA(Virtual Memory Area) 자료 구조를 추가로 만들 필요가 없다는 점이다.

비선형 매핑을 만들려면 다음 단계를 수행한다.

1. <tt>[[mmap(2)]]</tt>을 써서 (처음에는 선형인) 매핑을 만든다. `MAP_SHARED` 플래그를 줘서 매핑을 만들어야 한다.

2. `remap_file_pages()` 호출을 한 번 또는 그 이상 사용해서 매핑 페이지와 파일 페이지 간의 대응 관계를 재조정한다. 파일의 동일 페이지를 맵 영역 내의 여러 위치로 맵 하는 게 가능하다.

`pgoff` 및 `size` 인자는 매핑 내에서 재배치하려는 파일 영역을 지정한다. `pgoff`는 시스템 페이지 크기 단위의 파일 오프셋이고 `size`는 바이트 단위의 영역 길이이다.

`addr` 인자는 두 가지 역할을 한다. 첫째로, 페이지를 재조정하려는 매핑을 식별한다. 따라서 `addr`은 앞서 <tt>[[mmap(2)]]</tt> 호출로 맵 한 영역 내에 포함되는 주소여야 한다. 둘째로, `addr`은 `pgoff` 및 `size`로 지정한 파일 페이지들이 위치하게 될 주소를 나타낸다.

`addr`과 `size`에 지정하는 값을 시스템 페이지 크기의 배수로 하는 게 좋다. 배수가 아니면 커널에서 두 값 모두를 가장 가까운 페이지 크기 배수로 *내림* 한다.

`prot` 인자는 0으로 지정해야 한다.

`flags` 인자의 의미는 <tt>[[mmap(2)]]</tt>과 같되, `MAP_NONBLOCK` 외의 플래그는 모두 무시한다.

## RETURN VALUE

성공 시 `remap_file_pages()`는 0을 반환한다. 오류시 -1을 반환한며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>addr</code>이 <code>MAP_SHARED</code> 플래그로 생성한 유효한 매핑을 가리키고 있지 않다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>addr</code>, <code>size</code>, <code>prot</code>, <code>pgoff</code>가 유효하지 않다.</dd>
</dl>

## VERSIONS

리눅스 2.5.46에서 `remap_file_pages()` 시스템 호출이 등장했다. glibc 버전 2.3.3에서 지원이 추가되었다.

## CONFORMING TO

`remap_file_pages()` 시스템 호출은 리눅스 전용이다.

## NOTES

리눅스 2.6.23부터는 <tt>[[tmpfs(5)]]</tt>, hugetlbfs, ramfs 같은 메모리 내 파일 시스템에서만 `remap_file_pages()`가 비선형 매핑을 만든다. 기반 저장소가 있는 파일 시스템에서는 파일의 어느 부분을 어느 주소로 맵 할지 조정하는 데 있어서 `remap_file_pages()`가 <tt>[[mmap(2)]]</tt>보다 크게 효율적이지 않다.

## SEE ALSO

<tt>[[getpagesize(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[mmap2(2)]]</tt>, <tt>[[mprotect(2)]]</tt>, <tt>[[mremap(2)]]</tt>, <tt>[[msync(2)]]</tt>

----

2017-09-15
