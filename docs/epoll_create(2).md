## NAME

epoll_create, epoll_create1 - epoll 파일 디스크립터 열기

## SYNOPSIS

```c
#include <sys/epoll.h>

int epoll_create(int size);
int epoll_create1(int flags);
```

## DESCRIPTION

`epoll_create()`는 새 <tt>[[epoll(7)]]</tt> 인스턴스를 만든다. 리눅스 2.6.8부터는 `size` 인자를 무시하되 0보다는 커야 한다. NOTES 참고.

`epoll_create()`는 새 epoll 인스턴스를 가리키는 파일 디스크립터를 반환한다. 이후의 **epoll** 인터페이스 호출 모두에 이 파일 디스크립터가 쓰인다. `epoll_create()`가 반환한 파일 디스크립터가 더이상 필요치 않으면 <tt>[[close(2)]]</tt>로 닫아야 한다. epoll 인스턴스를 가리키는 모든 파일 디스크립터가 닫혔을 때 커널에서 그 인스턴스를 파기하고 관련 자원을 재사용할 수 있게 해제한다.

### `epoll_create1()`

`flags`가 0인 경우에는 구식 `size` 인자가 없어졌다는 점을 빼고 `epoll_create1()`이 `epoll_create()`와 동일하다. `flags`에 다음 값을 포함시켜서 다른 동작 방식을 얻을 수 있다.

`EPOLL_CLOEXEC`
:   새 파일 디스크립터에 'exec에서 닫기'(`FD_CLOEXEC`) 플래그를 설정한다. 이게 유용할 수 있는 이유에 대해선 <tt>[[open(2)]]</tt>의 `O_CLOEXEC` 플래그 설명을 보라.

## RETURN VALUE

성공 시 이 시스템 호출들은 파일 디스크립터(음수 아닌 정수)를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `size`가 양수가 아니다.

`EINVAL`
:   (`epoll_create1()`) `flags`에 유효하지 않은 값을 지정했다.

`EMFILE`
:   `/proc/sys/fs/epoll/max_user_instances`에 따른 epoll 인스턴스 개수에 대한 사용자별 제한에 걸렸다. 자세한 내용은 <tt>[[epoll(7)]]</tt> 참고.

`EMFILE`
:   열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다.

`ENFILE`
:   열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.

`ENOMEM`
:   커널 객체를 생성하기에 메모리가 충분하지 않았다.

## VERSIONS

커널 버전 2.6에서 `epoll_create()`이 추가되었다. glibc 버전 2.3.2부터 라이브러리 지원을 제공한다.

커널 버전 2.6.27에서 `epoll_create1()`이 추가되었다. glibc 버전 2.9부터 라이브러리 지원을 제공한다.

## CONFORMING TO

`epoll_create()`와 `epoll_create1()`은 리눅스 전용이다.

## NOTES

초기 `epoll_create()` 구현에서 `size` 인자는 호출자가 그 **epoll** 인스턴스에 추가하리라 예상하는 파일 디스크립터 수를 커널에게 알려주기 위한 것이었다. 커널에서는 내부의 이벤트 집합 자료 구조를 처음 할당할 때 크기에 대한 힌트로 그 정보를 사용했다. (호출자가 실제 사용에서 `size`로 준 힌트를 초과하면 커널에서 추가로 공간을 할당했다.) 요즘은 (힌트 없이도 커널에서 필요한 자료 구조 크기를 동적으로 조정하므로) 그 힌트가 더이상 필요치 않다. 하지만 `size`가 여전히 0보다 커야 하는데, 새 **epoll** 응용이 이전 커널에서 돌 때 하위 호환성을 보장하기 위해서이다.

## SEE ALSO

<tt>[[close(2)]]</tt>, <tt>[[epoll_ctl(2)]]</tt>, <tt>[[epoll_wait(2)]]</tt>, <tt>[[epoll(7)]]</tt>

----

2021-03-22
