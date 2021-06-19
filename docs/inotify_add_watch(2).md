## NAME

inotify_add_watch - 초기화 된 inotify 인스턴스에 감시 항목 추가하기

## SYNOPSIS

```c
#include <sys/inotify.h>

int inotify_add_watch(int fd, const char *pathname, uint32_t mask);
```

## DESCRIPTION

`inotify_add_watch()`는 `pathname`에 지정한 위치의 파일에 대해 새 감시 항목을 추가하거나 기존 감시 항목을 변경한다. 호출자가 그 파일에 대해 읽기 권한을 가지고 있어야 한다. `fd` 인자는 감시 목록을 변경할 inotify 인스턴스를 가리키는 파일 디스크립터이다. `pathname`에 대해 감시할 이벤트들을 비트 마스크 인자 `mask`에 지정한다. `mask`에 설정할 수 있는 비트들에 대한 설명은 <tt>[[inotify(7)]]</tt>를 보라.

`inotify_add_watch()` 호출이 성공하면 `pathname`에 해당하는 파일 시스템 객체(아이노드)에 대한, 이 inotify 인스턴스에서 유일한 감시 디스크립터를 반환한다. 그 파일 시스템 객체를 이 inotify 인스턴스로 이전부터 감시하고 있던 게 아니면 감시 디스크립터를 새로 할당한다. 그 파일 시스템 객체가 (어쩌면 동일 객체에 대한 다른 링크를 통해서) 이미 감시 중이었으면 기존 감시 항목에 대한 디스크립터를 반환한다.

이후 inotify 파일 디스크립터에서 <tt>[[read(2)]]</tt> 하면 그 감시 디스크립터가 반환된다. 읽기를 하면 파일 시스템 이벤트를 나타내는 `inotify_event` 구조체(<tt>[[inotify(7)]]</tt> 참고)를 가져오는데 그 구조체 내의 감시 디스크립터로 이벤트 발생 객체를 식별할 수 있다.

## RETURN VALUE

성공 시 `inotify_add_watch()`는 감시 디스크립터(음수 아닌 정수)를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   지정한 파일에 대해 읽기 권한이 거부되었다.

`EBADF`
:   지정한 파일 디스크립터가 유효하지 않다.

`EEXIST`
:   `mask`에 `IN_MASK_CREATE`가 들어 있는데 `pathname`이 같은 `fd`로 이미 감시 중인 파일을 가리키고 있다.

`EFAULT`
:   `pathname`이 프로세스의 접근 가능한 주소 공간 밖을 가리키고 있다.

`EINVAL`
:   지정한 이벤트 마스크에 유효한 이벤트가 들어 있지 않다. 또는 `mask`에 `IF_MASK_ADD`와 `IN_MASK_CREATE`가 같이 들어 있다. 또는 `fd`가 inotify 파일 디스크립터가 아니다.

`ENAMETOOLONG`
:   `pathname`이 너무 길다.

`ENOENT`
:   `pathname`의 어느 디렉터리 요소가 존재하지 않거나 깨진 심볼릭 링크이다.

`ENOMEM`
:   사용 가능한 커널 메모리가 충분하지 않다.

`ENOSPC`
:   inotify 감시 항목 총수에 대한 사용자별 제한에 도달했거나 커널에서 필요한 자원을 할당하는 데 실패했다.

`ENOTDIR`
:   `mask`에 `IN_ONLYDIR`이 들어 있는데 `pathname`이 디렉터리가 아니다.

## VERSIONS

리눅스 커널 2.6.13으로 inotify가 병합되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## EXAMPLES

<tt>[[inotify(7)]]</tt> 참고.

## SEE ALSO

<tt>[[inotify_init(2)]]</tt>, <tt>[[inotify_rm_watch(2)]]</tt>, <tt>[[inotify(7)]]</tt>

----

2021-03-22
