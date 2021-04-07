## NAME

inotify_rm_watch - inotify 인스턴스에서 기존 감시 항목 제거하기

## SYNOPSIS

```c
#include <sys/inotify.h>

int inotify_rm_watch(int fd, int wd);
```

## DESCRIPTION

`inotify_rm_watch()`는 파일 디스크립터 `fd`에 연계된 inotify 인스턴스에서 감시 디스크립터 `wd`에 연계된 감시 항목을 제거한다.

감시 항목을 제거하면 이 감시 디스크립터에 대해 `IN_IGNORED` 이벤트가 생성된다. (<tt>[[inotify(7)]]</tt> 참고.)

## RETURN VALUE

성공 시 `inotify_rm_watch()`는 0을 반환한다. 오류 시 -1을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EBADF</code></dt>
<dd><code>fd</code>가 유효한 파일 디스크립터가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd>감시 디스크립터 <code>wd</code>가 유효하지 않다. 또는 <code>fd</code>가 inotify 파일 디스크립터가 아니다.</dd>
</dl>

## VERSIONS

리눅스 커널 2.6.13으로 inotify가 병합되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## SEE ALSO

<tt>[[inotify_add_watch(2)]]</tt>, <tt>[[inotify_init(2)]]</tt>, <tt>[[inotify(7)]]</tt>

----

2017-09-15
