## NAME

numa - 불균일 기억 장치 접근 개요

## DESCRIPTION

불균일 기억 장치 접근(Non-Uniform Memory Access; NUMA)은 메모리가 여러 메모리 노드들로 나눠져 있는 다중 프로세서 시스템을 가리킨다. 메모리 노드의 접근 시간이 접근하려는 CPU와 접근 대상 노드의 상대적 위치에 따라 달라진다. (이는 메모리 전체에 대한 접근 시간이 모든 CPU에게 동일한 대칭형 다중 프로세서 시스템과 대비된다.) 보통 NUMA 시스템의 각 CPU에는 로컬 메모리 노드가 있어서 다른 CPU에 로컬인 노드의 메모리나 모든 CPU가 공유하는 버스 상의 메모리보다 빠르게 그 내용물에 접근할 수 있다.

### NUMA 시스템 호출

리눅스 커널에서 다음의 NUMA 관련 시스템 호출을 구현하고 있다: <tt>[[get_mempolicy(2)]]</tt>, <tt>[[mbind(2)]]</tt>, <tt>[[migrate_pages(2)]]</tt>, <tt>[[move_pages(2)]]</tt>, <tt>[[set_mempolicy(2)]]</tt>. 하지만 응용에서는 보통 `libnuma`가 제공하는 인터페이스를 사용하는 게 좋다. 아래의 "라이브러리 지원"을 보라.

### `/proc/[number]/numa_maps` (리눅스 2.6.14부터)

이 파일은 프로세스의 NUMA 메모리 정책과 할당에 대한 정보를 표시한다.

각 행은 프로세스가 사용하는 메모리 범위에 대한 정보를 담고 있다. 여러 정보들에 더해서 그 메모리 범위에 대한 실효 메모리 정책, 그리고 그 페이지들이 어느 노드 상에서 할당되었는지를 보여 준다.

`numa_maps`는 읽기 전용 파일이다. `/proc/<pid>/numa_maps`를 읽을 때 커널이 그 프로세스의 가상 주소 공간을 훑어서 메모리가 어떻게 쓰이고 있는지 알려 주게 된다. 프로세스의 고유한 메모리 범위마다 한 줄씩 표시된다.

각 행의 첫 번째 필드는 메모리 범위의 시작 주소이다. 이 필드를 범위의 끝 주소와 접근 권한 및 공유 같은 여타 정보를 담고 있는 `/proc/<pid>/maps` 파일의 내용과 연관시킬 수 있다.

두 번째 필드는 현재 그 메모리 범위에 효력이 있는 메모리 정책을 보여 준다. 참고로 그 실효 정책이 반드시 프로세스가 그 메모리 범위에 대해 설치했던 정책인 것은 아니다. 구체적으로 프로세스가 그 범위에 "default" 정책을 설치했다면 그 범위의 실효 정책은 프로세스 정책이 되는데, 이는 "default"일 수도 있고 아닐 수도 있다.

행의 나머지 부분에는 메모리 범위 내에서 할당된 페이지들에 대한 다음과 같은 정보가 들어있다.

<dl>
<dt><code>N&lt;node&gt;=&lt;nr_pages&gt;</code></dt>
<dd><code>&lt;node&gt;</code>에 할당된 페이지 개수. <code>&lt;nr_pages&gt;</code>는 현재 프로세스가 매핑 한 페이지들만 포함한다. 페이지 이전과 메모리 회수 때문에 이 메모리 범위에 연계된 페이지들이 일시적으로 매핑이 해제되었을 수도 있다. 그런 페이지들은 프로세스에서 그 페이지들에 접근하려 시도한 후에만 나타날 수도 있다. 메모리 범위가 공유 메모리 영역이나 파일 매핑을 나타내는 경우에는 다른 프로세스들이 동시에 해당 메모리 범위 내에 추가 페이지들을 매핑 하고 있을 수도 있다.</dd>

<dt><code>file=&lt;filename&gt;</code></dt>
<dd>메모리 범위의 기반이 되는 파일. 파일이 private으로 매핑 되어 있다면 쓰기 접근이 이 메모리 범위에서 COW (Copy-On-Write) 페이지들이 생성했을 수도 있다. 그런 페이지들은 익명 페이지로 표시된다.</dd>

<dt><code>heap</code></dt>
<dd>메모리 범위가 힙에 쓰인다.</dd>

<dt><code>stack</code></dt>
<dd>메모리 범위가 스택에 쓰인다.</dd>

<dt><code>huge</code></dt>
<dd>거대 메모리 범위. 표시된 페이지 개수는 거대 페이지들과 비정규 크기 페이지들이다.</dd>

<dt><code>anon=&lt;pages&gt;</code></dt>
<dd>범위 내의 익명 페이지 개수.</dd>

<dt><code>dirty=&lt;pages&gt;</code></dt>
<dd>더러워진 페이지 개수.</dd>

<dt><code>mapped=&lt;pages&gt;</code></dt>
<dd>매핑 된 페이지 총개수. <code>dirty</code> 및 <code>anon</code> 페이지들과 다른 경우.</dd>

<dt><code>mapmax=&lt;count&gt;</code></dt>
<dd>훑는 동안 만난 최대 mapcount (한 페이지를 매핑 하고 있는 프로세스들의 수). 해당 메모리 범위 내에서 이뤄지고 있는 공유의 정도를 나타내는 지표로 사용할 수 있다.</dd>

<dt><code>swapcache=&lt;count&gt;</code></dt>
<dd>스왑 장치에 연계 항목이 있는 페이지들의 개수.</dd>

<dt><code>active=&lt;pages&gt;</code></dt>
<dd>활동 목록의 페이지 개수. 이 범위의 페이지 개수와 다른 경우에만 이 필드가 보인다. 조만간 스와퍼에 의해 메모리에서 제거될 수도 있는 비활동 페이지들이 메모리 범위 내에 좀 존재한다는 뜻이다.</dd>

<dt><code>writeback=&lt;pages&gt;</code></dt>
<dd>현재 디스크로 기록 중인 페이지들의 개수.</dd>
</dl>

## CONFORMING TO

NUMA 인터페이스를 관할하는 표준이 없다.

## NOTES

리눅스의 NUMA 시스템 호출들과 `/proc` 인터페이스는 커널을 `CONFIG_NUMA` 옵션으로 구성하여 빌드 한 경우에만 사용 가능하다.

### 라이브러리 지원

시스템 호출 정의들이 필요하면 `-lnuma`로 링크 하라. `libnuma`와 필수적인 `<numaif.h>` 헤더를 `numactl` 패키지에서 얻을 수 있다.

하지만 응용에서 이 시스템 호출들을 직접 사용하지 않는 게 좋다. 그보다는 `numactl` 패키지의 <tt>[[numa(3)]]</tt> 함수들이 제공하는 고수준 인터페이스를 권장한다. `numactl` 패키지를 (ftp://oss.sgi.com/www/projects/libnuma/download/ )에서 얻을 수 있다. 또한 일부 리눅스 배포판에도 이 패키지가 포함되어 있다. 일부 배포판에서는 별도의 `numactl-devel` 패키지에 개발용 라이브러리와 헤더가 있다.

## SEE ALSO

<tt>[[get_mempolicy(2)]]</tt>, <tt>[[mbind(2)]]</tt>, <tt>[[move_pages(2)]]</tt>, <tt>[[set_mempolicy(2)]]</tt>, <tt>[[numa(3)]]</tt>, <tt>[[cpuset(7)]]</tt>, `numactl(8)`

----

2012-08-05
