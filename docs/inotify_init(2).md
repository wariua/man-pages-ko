## NAME

inotify_init, inotify_init1 - inotify 인스턴스 초기화 하기

## SYNOPSIS

```c
#include <sys/inotify.h>

int inotify_init(void);
int inotify_init1(int flags);
```

## DESCRIPTION

inotify API에 대한 소개는 <tt>[[inotify(7)]]</tt>를 보라.

`inotify_init()`은 새 inotify 인스턴스를 초기화 하고 그 새 inotify 이벤트 큐에 연계된 파일 디스크립터를 반환한다.

`inotify_init1()`은 `flags`가 0이면 `inotify_init()`과 동일하다. `flags`에 다음 값들을 비트 OR 해서 다른 동작 방식을 얻을 수 있다.

`IN_NONBLOCK`
:   새 파일 디스크립터가 가리키는 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)에 `O_NONBLOCK` 파일 상태 플래그를 설정한다. 이 플래그를 사용하면 같은 결과를 얻기 위해 <tt>[[fcntl(2)]]</tt>을 추가로 호출하지 않아도 된다.

`IN_CLOEXEC`
:   새 파일 디스크립터에 exec에서 닫기(`FD_CLOEXEC`) 플래그를 설정한다. 이게 유용할 수 있는 이유에 대해선 <tt>[[open(2)]]</tt>의 `O_CLOEXEC` 플래그 설명을 보라.

## RETURN VALUE

성공 시 이 시스템 호출들은 새 파일 디스크립터를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   (`inotify_init1()`) `flags`에 유효하지 않은 플래그를 지정했다.

`EMFILE`
:   inotify 인스턴스에 대한 사용자별 제한에 도달했다.

`EMFILE`
:   열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다.

`ENFILE`
:   열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.

`ENOMEM`
:   사용 가능한 커널 메모리가 충분하지 않다.

## VERSIONS

리눅스 2.6.13에서 `inotify_init()`이 처음 등장했다. glibc 버전 2.4에서 라이브러리 지원이 추가되었다. 리눅스 2.6.27에서 `inotify_init1()`이 추가되었다. glibc 버전 2.9에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

이 시스템 호출들은 리눅스 전용이다.

## SEE ALSO

<tt>[[inotify_add_watch(2)]]</tt>, <tt>[[inotify_rm_watch(2)]]</tt>, <tt>[[inotify(7)]]</tt>

----

2020-04-11
