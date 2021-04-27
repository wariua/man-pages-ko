## NAME

migrate_pages - 프로세스의 모든 페이지들을 다른 노드들로 옮기기

## SYNOPSIS

```c
#include <numaif.h>

long migrate_pages(int pid, unsigned long maxnode,
                   const unsigned long *old_nodes,
                   const unsigned long *new_nodes);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

`-lnuma`로 링크.

## DESCRIPTION

`migrate_pages()`는 프로세스 `pid`의 페이지들 중 메모리 노드 `old_nodes`에 있는 것들을 모두 메모리 노드 `new_nodes`로 옮기려고 시도한다. `old_nodes`의 어느 노드에도 위치해 있지 않은 페이지는 이동하지 않는다. 커널에서는 `new_nodes`로 이동시키는 동안 `old_nodes` 내의 상대적 위상 관계를 가급적 유지한다.

`old_nodes` 및 `new_nodes` 인자는 노드 번호 비트 마스크의 포인터이며, 각 마스크에 최대 `maxnode` 개 비트가 있다. 그 마스크들은 부호 없는 `long` 정수들의 배열 형태로 유지한다. (마지막 `long` 정수에서 `maxnode`로 지정한 것 너머의 비트들은 무시한다.) `maxnode` 인자는 비트 마스크 내의 가장 큰 노드 번호에 1을 더한 것이다. (<tt>[[mbind(2)]]</tt>와는 같지만 <tt>[[select(2)]]</tt>와는 다르다.)

`pid` 인자는 이동할 페이지들을 소유한 프로세스의 ID이다. 다른 프로세스의 페이지들을 옮기려면 호출자에게 특권(`CAP_SYS_NICE`)이 있거나 호출 프로세스의 실제 내지 실효 사용자 ID가 대상 프로세스의 실제 내지 saved-set 사용자 ID와 일치해야 한다. `pid`가 0이면 `migrate_pages()`는 호출 프로세스의 페이지를 옮긴다.

다른 프로세스와 공유하는 페이지들은 개시 프로세스에게 `CAP_SYS_NICE` 특권이 있는 경우에만 옮긴다.

## RETURN VALUE

성공 시 `migrate_pages()`는 옮길 수 없었던 페이지 개수를 반환한다. (즉 0이 반환되면 모든 페이지를 성공적으로 옮겼다는 뜻이다.) 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `old_nodes`/`new_nodes`와 `maxnode`로 지정한 메모리 범위의 일부 내지 전체가 접근 가능한 주소 공간 밖을 가리킨다.

`EINVAL`
:   `maxnode`에 지정한 값이 커널에서 두는 제한치를 초과한다. 또는 지원하는 가장 큰 노드보다 큰 노드 ID를 `old_nodes`나 `new_nodes`에 한 개 이상 지정했다. 또는 `new_nodes`로 지정한 노드 ID들 중에서 온라인이고 프로세스의 현재 cpuset 문맥에서 허용되는 게 없거나, 지정한 노드들 중에서 메모리를 담고 있는 게 없다.

`EPERM`
:   `pid`로 지정한 프로세스의 페이지를 옮길 특권(`CAP_SYS_NICE`)이 부족하거나, 지정한 대상 노드에 접근하기 위한 특권(`CAP_SYS_NICE`)이 부족하다.

`ESRCH`
:   `pid`에 일치하는 프로세스를 찾을 수 없다.

## VERSIONS

리눅스 버전 2.6.16에서 `migrate_pages()` 시스템 호출이 처음 등장했다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. 라이브러리 지원에 대한 정보는 <tt>[[numa(7)]]</tt>를 보라.

<tt>[[get_mempolicy(2)]]</tt>를 `MPOL_F_MEMS_ALLOWED` 플래그로 사용하면 호출 프로세스의 cpuset에서 허용하는 노드들의 집합을 얻을 수 있다. 참고로 그 정보는 수동 내지 자동으로 이뤄지는 cpuset 재구성으로 인해 언제든 바뀔 수 있다.

`migrate_pages()`를 사용하면 페이지들의 위치(노드)가 지정 주소(<tt>[[mbind(2)]]</tt>) 및/또는 지정 프로세스(<tt>[[set_mempolicy(2)]]</tt>)에 대해 설정한 메모리 정책과 어긋나게 될 수도 있다. 다시 말해 메모리 정책이 `migrate_pages()`에서 쓰는 목적지 노드를 제약하지 않는다.

`<numaif.h>` 헤더는 glibc에 포함되어 있지 않으며 `libnuma-devel` 내지 그와 비슷한 패키지를 설치해야 한다.

## SEE ALSO

<tt>[[get_mempolicy(2)]]</tt>, <tt>[[mbind(2)]]</tt>, <tt>[[set_mempolicy(2)]]</tt>, <tt>[[numa(3)]]</tt>, <tt>[[numa_maps(5)]]</tt>, <tt>[[cpuset(7)]]</tt>, <tt>[[numa(7)]]</tt>, `migratepages(8)`, `numastat(8)`

리눅스 커널 소스 트리의 `Documentation/vm/page_migration.rst`

----

2021-03-22
