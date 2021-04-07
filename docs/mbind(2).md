## NAME

mbind - 메모리 범위에 메모리 정책 설정하기

## SYNOPSIS

```c
#include <numaif.h>

long mbind(void *addr, unsigned long len, int mode,
           const unsigned long *nodemask, unsigned long maxnode,
           unsigned flags);
```

`-lnuma`로 링크.

## DESCRIPTION

`mbind()`는 `addr`에서 시작해서 `len` 바이트만큼 이어지는 메모리 범위에 대해 NUMA 메모리 정책을 설정한다. 메모리 정책은 정책 모드와 0개 이상의 노드로 이뤄지며, 메모리를 어느 노드로부터 할당하는지 규정한다.

`addr` 및 `len` 인자로 지정한 메모리 범위에 "익명" 메모리 영역(즉 <tt>[[mmap(2)]]</tt> 시스템 호출을 `MAP_ANONYMOUS`로 써서 만든 메모리 영역)이나 (<tt>[[mmap(2)]]</tt> 시스템 호출을 `MAP_PRIVATE` 플래그로 써서 맵 한) 메모리 맵 파일이 포함돼 있으면 응용에서 그 페이지에 쓰기(저장)를 할 때 지정 정책에 따라 페이지를 할당하게 된다. 익명 영역에서 처음 읽기 접근을 하면 0으로 채워진 커널 내 공유 페이지를 이용하게 된다. `MAP_PRIVATE`으로 맵 한 파일에서 처음 읽기 접근을 하면 페이지 할당을 유발한 스레드의 메모리 정책에 따라 페이지를 할당하게 된다. 그 스레드가 `mbind()`를 호출한 스레드가 아닐 수도 있다.

지정한 메모리 범위 내의 `MAP_SHARED` 매핑에서는 지정 정책이 무시된다. 페이지 할당을 유발한 스레드의 메모리 정책에 따라 페이지를 할당하게 된다. 마찬가지로 `mbind()`를 호출한 스레드가 아닐 수도 있다.

지정한 메모리 범위에 <tt>[[shmget(2)]]</tt> 시스템 호출로 만들어서 <tt>[[shmat(2)]]</tt> 시스템 호출로 붙인 공유 메모리 영역이 포함돼 있으면 그 공유 메모리 세그먼트에 붙은 어느 프로세스가 할당을 유발했는지와 상관없이 지정 정책에 따라 그 익명 내지 공유 메모리 영역을 위한 페이지들을 할당하게 된다. 하지만 그 공유 메모리 영역이 `SHM_HUGETLB` 플래그를 써서 만든 것이라면 그 영역에 `mbind()`를 호출한 프로세스가 페이지 할당을 유발한 경우에만 지정 정책에 따라 거대 페이지를 할당하게 된다.

기본적으로 `mbind()`는 새 할당에만 효력이 있다. 정책을 설정하기 전에 그 범위 내의 페이지들을 이미 건드렸다면 정책에 아무 효과가 없다. 아래에서 설명하는 `MPOL_MF_MOVE` 및 `MPOL_MF_MOVE_ALL` 플래그로 이 기본 동작 방식을 바꿀 수 있다.

`mode` 인자에는 `MPOL_DEFAULT`, `MPOL_BIND`, `MPOL_INTERLEAVE`, `MPOL_PREFERRED`, `MPOL_LOCAL` (아래에서 자세히 설명함) 중 하나를 지정해야 한다. `MPOL_DEFAULT`를 제외한 모든 정책 모드에서 호출자가 `nodemask` 인자를 통해 모드를 적용할 노드를 지정해야 한다.

`mode` 인자에 선택적으로 <em>모드 플래그</em>가 포함될 수 있다. 지원하는 <em>모드 플래그</em>는 다음과 같다.

<dl>
<dt><code>MPOL_F_STATIC_NODES</code> (리눅스 2.6.26부터)</dt>
<dd>비어 있지 않은 <code>nodemask</code>가 물리적 노드 ID들을 나타낸다. 스레드가 다른 cpuset 문맥으로 이동하거나 스레드의 현재 cpuset 문맥이 허용하는 노드 집합이 바뀔 때 리눅스에서 <code>nodemask</code>를 재사상 하지 않는다.</dd>

<dt><code>MPOL_F_RELATIVE_NODES</code> (리눅스 2.6.26부터)</dt>
<dd>비어 있지 않은 <code>nodemask</code>가 스레드의 현재 cpuset이 허용하는 노드 ID 집합에 대한 상대적인 노드 ID들을 나타낸다.</dd>
</dl>

`nodemask`는 노드들의 비트 마스크를 가리키며 그 비트 마스크에는 최대 `maxnode` 개까지의 비트가 담겨 있다. 비트 마스크 크기를 `sizeof(unsigned long)`의 다음 배수로 올림 하지만 커널에서는 `maxnode` 개까지의 비트만 사용한다. `nodemask` 값이 NULL이거나 `maxnode` 값이 0이면 빈 노드 집합을 나타낸다. `maxnode`의 값이 0이면 `nodemask` 인자를 무시한다. `nodemask`가 필수인 경우에 그 인자는 온라인이고, (`MPOL_F_STATIC_NODES` 모드 플래그를 지정하지 않았으면) 스레드의 현재 cpuset 문맥이 허용하고, 메모리를 담고 있는 노드를 적어도 한 개는 포함해야 한다.

`mode` 인자에 다음 값들 중 하나가 포함돼야 한다.

<dl>
<dt><code>MPOL_DEFAULT</code></dt>
<dd>이 모드는 기본과 다른 정책을 모두 제거해서 기본 동작 방식을 복원하도록 요청한다. <code>mbind()</code>를 통해 메모리 범위에 적용 시 이는 <tt>[[set_mempolicy(2)]]</tt>로 설정한 것일 수도 있는 스레드 메모리 정책을 사용하라는 뜻이다. 스레드 메모리 정책의 모드도 <code>MPOL_DEFAULT</code>이면 시스템 전역 기본 정책을 쓰게 된다. 시스템 전역 기본 정책은 할당을 촉발한 CPU의 노드에서 페이지를 할당하는 것이다. <code>MPOL_DEFAULT</code>에서 <code>nodemask</code> 및 <code>maxnode</code> 인자는 빈 노드 집합을 나타내야 한다.</dd>

<dt><code>MPOL_BIND</code></dt>
<dd>이 모드는 메모리 할당을 <code>nodemask</code>에 지정한 노드들로 제약하는 엄격한 정책을 지정한다. <code>nodemask</code>에 노드가 여러 개 있으면 충분한 유휴 메모리를 가지고 있으면서 할당이 이뤄지는 노드에 가장 가까운 노드에서 페이지 할당을 한다. <code>nodemask</code>에 지정하지 않은 어떤 노드에서도 페이지를 할당하지 않는다. (리눅스 2.6.26 전에서는 노드 ID의 숫자 값이 가장 낮은 노드에서부터 페이지 할당을 했다. 그 노드에 유휴 메모리가 없어지면 <code>nodemask</code>에 지정한 다음으로 낮은 ID의 노드에서 할당을 하고, 그런 식으로 지정한 노드들 어디에도 유휴 메모리가 없을 때까지 진행했다.)</dd>

<dt><code>MPOL_INTERLEAVE</code></dt>
<dd>이 모드에서는 <code>nodemask</code>에 지정한 노드 집합 내에서 교대로 페이지 할당을 한다. 페이지들과 그에 대한 메모리 접근을 여러 노드로 분산시킴으로써 지연보다는 대역폭을 최적화 한다. 효과가 있으려면 메모리 영역이 꽤 커야 한다. 최소 1MB는 돼야 꽤 균일한 접근 패턴이 나온다. 단일 페이지 영역에 대한 접근은 여전히 단일 노드 메모리 대역폭으로 제한된다.</dd>

<dt><code>MPOL_PREFERRED</code></dt>
<dd>이 모드는 할당 선호 노드를 설정한다. 커널이 먼저 그 노드에서 페이지를 할당하려고 시도하지만 그 선호 노드에 유휴 메모리가 부족하면 다른 노드로 후퇴한다. <code>nodemask</code>에 노드 ID를 여러 개 지정하면 마스크의 첫 번째 노드를 선호 노드로 선택하게 된다. <code>nodemask</code> 및 <code>maxnode</code> 인자가 빈 집합을 나타내는 경우에는 할당을 촉발한 CPU의 노드에서 메모리를 할당한다.</dd>

<dt><code>MPOL_LOCAL</code> (리눅스 3.8부터)</dt>
<dd>이 모드는 "지역 할당"을 나타낸다. 즉 할당을 촉발한 CPU의 노드("지역 노드")에서 메모리를 할당한다. <code>nodemask</code> 및 <code>maxnode</code> 인자가 빈 집합을 나타내야 한다. "지역 노드"에 유휴 메모리가 부족한 경우 커널은 다른 노드에서 메모리 할당을 시도하게 된다. 사용 가능한 메모리가 있는 경우에는 커널이 항상 "지역 노드"에서 메모리를 할당하게 된다. 스레드의 현재 cpuset 문맥이 "지역 노드"를 허용하지 않으면 커널이 다른 노드에서 메모리 할당을 시도하게 된다. 스레드의 현재 cpuset 문맥에서 허용하게 되면 커널이 항상 "지역 노드"에서 메모리를 할당하게 된다. 이와 달리 <code>MPOL_DEFAULT</code>는 (<tt>[[set_mempolicy(2)]]</tt>를 통해 설정한 것일 수도 있는) 스레드의 메모리 정책으로 되돌아간다. 그 정책은 "지역 할당" 아닌 어떤 정책일 수도 있다.</dd>
</dl>

`flags`에 `MPOL_MF_STRICT`를 주고 `mode`가 `MPOL_DEFAULT`가 아니면 메모리 범위 내의 기존 페이지들이 정책을 따르지 않는 경우 호출이 `EIO` 오류로 실패한다.

`flags`에 `MPOL_MF_MOVE`를 지정하면 메모리 범위 내의 기존 페이지들이 모두 정책을 따르게 되도록 페이지를 옮기려고 커널이 시도하게 된다. 다른 프로세스와 공유하는 페이지는 옮기지 않는다. `MPOL_MF_STRICT`를 함께 지정하면 일부 페이지를 옮길 수 없는 경우 호출이 `EIO` 오류로 실패한다.

`flags`에 `MPOL_MF_MOVE_ALL`을 주면 다른 프로세스가 페이지를 쓰고 있는지 여부와 상관없이 메모리 범위 내의 기본 페이지 모두를 옮기려고 커널이 시도하게 된다. 이 플래그를 쓰려면 호출 스레드가 특권(`CAP_SYS_NICE`)을 가지고 있어야 한다. `MPOL_MF_STRICT`를 함께 지정하면 일부 페이지를 옮길 수 없는 경우 호출이 `EIO` 오류로 실패한다.

## RETURN VALUE

성공 시 `mbind()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EFAULT</code></dt>
<dd><code>nodemask</code>와 <code>maxnode</code>로 지정한 메모리 범위의 일부 내지 전체가 접근 가능한 주소 공간 밖을 가리킨다. 또는 <code>addr</code>과 <code>len</code>으로 지정한 메모리 범위에 맵 안 된 구멍이 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>flags</code>나 <code>mode</code>에 유효하지 않은 값을 지정했다. 또는 <code>addr + len</code>이 <code>addr</code>보다 작다. 또는 <code>addr</code>이 시스템 페이지 크기의 배수가 아니다. 또는 <code>mode</code>가 <code>MPOL_DEFAULT</code>인데 <code>nodemask</code>가 비어 있지 않은 집합을 나타낸다. 또는 <code>mode</code>가 <code>MPOL_BIND</code>나 <code>MPOL_INTERLEAVE</code>이고 <code>nodemask</code>가 비어 있다. 또는 <code>maxnode</code>가 커널에서 두는 제한치를 초과한다. 또는 지원하는 가장 큰 노드 ID보다 큰 노드 ID를 <code>nodemask</code>에 한 개 이상 지정했다. 또는 <code>nodemask</code>로 지정한 노드 ID들 중에서 온라인이고 스레드의 현재 cpuset 문맥에서 허용되는 게 없거나, 지정한 노드들 중에서 메모리를 담고 있는 게 없다. 또는 <code>mode</code> 인자에 <code>MPOL_F_STATIC_NODES</code>와 <code>MPOL_F_RELATIVE_NODES</code>를 함께 지정했다.
</dd>
<dt><code>EIO</code></dt>
<dd><code>MPOL_MF_STRICT</code>를 지정했으며 그 노드에 정책을 따르지 않는 기존 페이지가 이미 있다. 또는 <code>MPOL_MF_MOVE</code>나 <code>MPOL_MF_MOVE_ALL</code>을 지정했으며 커널에서 그 범위 내의 기존 페이지 전체를 옮길 수 없었다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>사용 가능한 커널 메모리가 충분하지 않다.</dd>
<dt><code>EPERM</code></dt>
<dd><code>flags</code> 인자에 <code>MPOL_MF_MOVE_ALL</code> 플래그가 포함돼 있는데 호출자가 <code>CAP_SYS_NICE</code> 특권을 가지고 있지 않다.</dd>
</dl>

## VERSIONS

리눅스 커널 버전 2.6.7에서 `mbind()` 시스템 호출이 추가되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

라이브러리 지원에 대한 정보는 <tt>[[numa(7)]]</tt>를 보라.

`MAP_SHARED` 플래그로 맵 한 메모리 맵 파일 범위에서는 NUMA 정책을 지원하지 않는다.

`MPOL_DEFAULT` 모드의 효과가 `mbind()`와 <tt>[[set_mempolicy(2)]]</tt>에서 다를 수 있다. <tt>[[set_mempolicy(2)]]</tt>에서 `MPOL_DEFAULT`를 지정하면 스레드의 메모리 정책이 시스템 기본 정책인 지역 할당으로 되돌아간다. 반면 `mbind()`를 이용해 메모리 범위에 `MPOL_DEFAULT`를 지정하면 그 범위에서 이후 할당하는 페이지들이 <tt>[[set_mempolicy(2)]]</tt>로 설정한 스레드 메모리 정책을 사용하게 된다. 지정한 범위에서 명시적 정책을 없애서 기본과는 다를 수도 있는 정책으로 "후퇴"시키는 효과가 있다. 메모리 범위에서 명시적으로 "지역 할당"을 선택하려면 빈 노드 집합으로 `mode`에 `MPOL_LOCAL`이나 `MPOL_PREFERRED`를 지정하면 된다. <tt>[[set_mempolicy(2)]]</tt>에서도 마찬가지로 이 방법이 통한다.

거대 페이지 정책 지원은 2.6.16에서 추가되었다. 거대 페이지 매핑에서 교대 정책이 효과가 있으려면 정책 대상 메모리가 수십 메가바이트 이상이 되어야 한다.

거대 페이지 매핑에서는 `MPOL_MF_STRICT`를 무시한다.

`MPOL_MF_MOVE`와 `MPOL_MF_MOVE_ALL`은 리눅스 2.6.16 및 이후에서만 사용 가능하다.

## SEE ALSO

<tt>[[get_mempolicy(2)]]</tt>, <tt>[[getcpu(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[set_mempolicy(2)]]</tt>, <tt>[[shmat(2)]]</tt>, <tt>[[shmget(2)]]</tt>, <tt>[[numa(3)]]</tt>, <tt>[[cpuset(7)]]</tt>, <tt>[[numa(7)]]</tt>, `numactl(8)`

----

2017-09-15
