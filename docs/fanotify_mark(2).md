## NAME

fanotify_mark - 파일 시스템 객체에 대한 fanotify 표시 추가, 제거, 변경하기

## SYNOPSIS

```c
#include <sys/fanotify.h>

int fanotify_mark(int fanotify_fd, unsigned int flags,
                  uint64_t mask, int dirfd, const char *pathname);
```

## DESCRIPTION

fanotify API에 대한 소개는 <tt>[[fanotify(7)]]</tt>를 보라.

`fanotify_mark()`는 파일 시스템 객체에 fanotify 표시를 추가하거나 제거하거나 변경한다. 표시를 하는 파일 시스템 객체에 대해 호출자가 읽기 권한을 가지고 있어야 한다.

`fanotify_fd` 인자는 <tt>[[fanotify_init(2)]]</tt>이 반환한 파일 디스크립터이다.

`flags`는 수행할 변경 동작을 기술하는 비트 마스크이다. 다음 값들 중 정확히 하나를 포함해야 한다.

<dl>
<dt><code>FAN_MARK_ADD</code></dt>
<dd>표시 마스크에 (또는 무시 마스크에) <code>mask</code>의 이벤트들을 추가한다. <code>mask</code>가 비어 있지 않아야 하며, 안 그러면 <code>EINVAL</code> 오류가 발생한다.</dd>

<dt><code>FAN_MARK_REMOVE</code></dt>
<dd>표시 마스크에서 (또는 무시 마스크에서) <code>mask</code> 인자의 이벤트들을 제거한다. <code>mask</code>가 비어 있지 않아야 하며, 안 그러면 <code>EINVAL</code> 오류가 발생한다.</dd>

<dt><code>FAN_MARK_FLUSH</code></dt>
<dd>fanotify 그룹에서 모든 파일 시스템 표시, 또는 모든 마운트 표시, 또는 모든 디렉터리 및 파일 표시를 제거한다. <code>flags</code>에 <code>FAN_MARK_MOUNT</code>가 있으면 그룹에서 마운트에 대한 표시들을 모두 제거한다. <code>flags</code>에 <code>FAN_MARK_FILESYSTEM</code>이 있으면 그룹에서 파일 시스템에 대한 표시들을 모두 제거한다. 그렇지 않으면 디렉터리 및 파일에 대한 표시들을 모두 제거한다. <code>FAN_MARK_MOUNT</code>와 <code>FAN_MARK_FILESYSTEM</code> 외의 플래그나 두 플래그 모두를 <code>FAN_MARK_FLUSH</code>와 함께 사용할 수 없다. <code>mask</code>는 무시한다.</dd>
</dl>

위 값들 중 아무것도 지정하지 않거나 여러 가지를 지정하면 호출이 `EINVAL` 오류로 실패한다.

추가로 `flags`에 다음 값들을 0개 이상 OR 할 수 있다.

<dl>
<dt><code>FAN_MARK_DONT_FOLLOW</code></dt>
<dd><code>pathname</code>이 심볼릭 링크이면 가리키는 파일이 아니라 링크 자체에 표시를 한다. (기본적으로 <code>fanotify_mark()</code>는 <code>pathname</code>이 심볼릭 링크이면 역참조를 한다.)</dd>

<dt><code>FAN_MARK_ONLYDIR</code></dt>
<dd>표시할 파일 시스템 객체가 디렉터리가 아니면 <code>ENOTDIR</code> 오류를 제기한다.</dd>

<dt><code>FAN_MARK_MOUNT</code></dt>
<dd><code>pathname</code>으로 지정한 마운트 지점에 표시를 한다. <code>pathname</code> 자체가 마운트 지점이 아니면 <code>pathname</code>을 담은 마운트 지점에 표시를 하게 된다. 마운트 지점의 모든 디렉터리와 서브디렉터리, 그리고 거기 담긴 파일들을 감시한다. 파일 디스크립터 <code>fanotify_fd</code>를 <code>FAN_REPORT_FID</code> 플래그로 초기화 했거나 <code>mask</code>에 새 디렉터리 변경 이벤트를 하나라도 준 경우에는 이 값을 쓸 수 없다. 그렇게 해서 시도하면 오류 <code>EINVAL</code>이 반환된다.</dd>

<dt><code>FAN_MARK_FILESYSTEM</code> (리눅스 4.20부터)</dt>
<dd><code>pathname</code>으로 지정한 파일 시스템에 표시를 한다. <code>pathname</code>을 담은 파일 시스템에 표시를 하게 된다. 마운트 지점이 어디든 그 파일 시스템에 담긴 모든 파일과 디렉터리를 감시한다.</dd>

<dt><code>FAN_MARK_IGNORED_MASK</code></dt>
<dd>무시 마스크에서 <code>mask</code>의 이벤트들을 추가하거나 제거한다.</dd>

<dt><code>FAN_MARK_IGNORED_SURV_MODIFY</code></dt>
<dd>무시 마스크가 변경 이벤트를 거치며 유지된다. 이 플래그가 설정돼 있지 않으면 무시하는 파일이나 디렉터리에 변경 이벤트가 발생할 때 그 무시 마스크를 없앤다.</dd>
</dl>

`mask`는 어떤 이벤트를 청취할지 (또는 무시할지) 지정한다. 다음 값들로 이뤄진 비트 마스크이다.

<dl>
<dt><code>FAN_ACCESS</code></dt>
<dd>파일이나 디렉터리에 (BUGS 참고) 접근(읽기)이 이뤄지면 이벤트 생성.</dd>

<dt><code>FAN_MODIFY</code></dt>
<dd>파일이 변경(쓰기)되면 이벤트 생성.</dd>

<dt><code>FAN_CLOSE_WRITE</code></dt>
<dd>쓰기 가능 파일이 닫히면 이벤트 생성.</dd>

<dt><code>FAN_CLOSE_NOWRITE</code></dt>
<dd>읽기 전용 파일이나 디렉터리가 닫히면 이벤트 생성.</dd>

<dt><code>FAN_OPEN</code></dt>
<dd>파일이나 디렉터리가 열리면 이벤트 생성.</dd>

<dt><code>FAN_OPEN_EXEC</code> (리눅스 5.0부터)</dt>
<dd>파일이 실행하려는 의도로 열리면 이벤트 생성. NOTES의 추가 설명 참고.</dd>

<dt><code>FAN_ATTRIB</code></dt>
<dd>파일이나 디렉터리의 메타데이터가 바뀌었을 때 이벤트 생성.</dd>

<dt><code>FAN_CREATE</code></dt>
<dd>표시된 부모 디렉터리 내에서 파일이나 디렉터리가 생성됐을 때 이벤트 생성.</dd>

<dt><code>FAN_DELETE</code></dt>
<dd>표시된 부모 디렉터리 내에서 파일이나 디렉터리가 삭제됐을 때 이벤트 생성.</dd>

<dt><code>FAN_DELETE_SELF</code></dt>
<dd>표시된 파일이나 디렉터리 자체가 삭제됐을 때 이벤트 생성.</dd>

<dt><code>FAN_MOVED_FROM</code></dt>
<dd>표시된 부모 디렉터리에 있던 파일이나 디렉터리가 이동됐을 때 이벤트 생성.</dd>

<dt><code>FAN_MOVED_TO</code></dt>
<dd>표시된 부모 디렉터리로 파일이나 디렉터리가 이동됐을 때 이벤트 생성.</dd>

<dt><code>FAN_Q_OVERFLOW</code></dt>
<dd>이벤트 큐 넘침이 발생하면 이벤트 생성. <tt>[[fanotify_init(2)]]</tt>에서 <code>FAN_UNLIMITED_QUEUE</code>를 설정하지 않으면 이벤트 큐 크기가 16384개 항목으로 제한된다.</dd>

<dt><code>FAN_OPEN_PERM</code></dt>
<dd>파일이나 디렉터리 열기 요청이 있으면 이벤트 생성. <code>FAN_CLASS_PRE_CONTENT</code>나 <code>FAN_CLASS_CONTENT</code>로 생성한 fanotify 파일 디스크립터가 필요하다.</dd>

<dt><code>FAN_OPEN_EXEC_PERM</code> (리눅스 5.0부터)</dt>
<dd>실행을 위한 파일 열기 요청이 있으면 이벤트 생성. <code>FAN_CLASS_PRE_CONTENT</code>나 <code>FAN_CLASS_CONTENT</code>로 생성한 fanotify 파일 디스크립터가 필요하다. NOTES의 추가 설명 참고.</dd>

<dt><code>FAN_ACCESS_PERM</code></dt>
<dd>파일이나 디렉터리 읽기 요청이 있으면 이벤트 생성. <code>FAN_CLASS_PRE_CONTENT</code>나 <code>FAN_CLASS_CONTENT</code>로 생성한 fanotify 파일 디스크립터가 필요하다.</dd>

<dt><code>FAN_ONDIR</code></dt>
<dd>디렉터리에 대해 (가령 <tt>[[opendir(3)]]</tt>, <tt>[[readdir(3)]]</tt> (단 BUGS 참고), <tt>[[closedir(3)]]</tt>이 호출될 때) 이벤트 생성. 이 플래그가 없으면 파일에 대한 이벤트만 생긴다. 파일 디스크립터 <code>fanotify_fd</code>를 <code>FAN_REPORT_FID</code> 플래그로 초기화 했을 때만 <code>FAN_ONDIR</code> 플래그를 보고한다. <code>FAN_CREATE</code>, <code>FAN_DELETE</code>, <code>FAN_MOVED_FROM</code>, <code>FAN_MOVED_TO</code> 같은 디렉터리 항목 이벤트 맥락에서 서브디렉터리 항목들이 변경(즉 <tt>[[mkdir(2)]]</tt>/<tt>[[rmdir(2)]]</tt>)될 때 이벤트가 생성되게 하려면 <code>FAN_ONDIR</code> 플래그를 지정해야 한다. 이 플래그는 이벤트에서 단독으로 보고되는 일이 절대 없으며 항상 다른 종류의 이벤트와 함께 제공된다.</dd>

<dt><code>FAN_EVENT_ON_CHILD</code></dt>
<dd>표시한 디렉터리의 직접 자식들에 대한 이벤트를 생성한다. 마운트 및 파일 시스템에 표시할 때는 이 플래그에 아무 효과가 없다. 참고로 표시한 디렉터리의 서브디렉터리의 자식들에 대해선 이벤트가 생성되지 않는다. 디렉터리 트리 전체를 감시하려면 적절한 마운트에 표시를 해야 한다.</dd>
</dl>

다음 조합 값이 정의돼 있다.

<dl>
<dt><code>FAN_CLOSE</code></dt>
<dd>파일이 닫혔음. (<code>FAN_CLOSE_WRITE|FAN_CLOSE_NOWRITE</code>)</dd>

<dt><code>FAN_MOVE</code></dt>
<dd>파일이나 디렉터리가 이동됐음. (<code>FAN_MOVED_FROM|FAN_MOVED_TO</code>)</dd>
</dl>

파일 디스크립터 `dirfd`와 `pathname`에 지정한 경로명을 써서 표시를 할 파일 시스템 객체를 결정한다.

* `pathname`이 NULL이면 `dirfd`가 표시할 파일 시스템 객체를 결정한다.

* `pathname`이 NULL이고 `dirfd`가 특수 값 `AT_FDCWD`이면 현재 작업 디렉터리에 표시를 한다.

* `pathname`이 절대 경로이면 표시할 파일 시스템 객체를 결정하며, `dirfd`는 무시된다.

* `pathname`이 상대 경로이고 `dirfd`의 값이 `AT_FDCWD`가 아니면 `dirfd`가 가리키는 디렉터리를 기준으로 `pathname`을 해석해서 표시할 파일 시스템 객체를 결정한다.

* `pathname`이 상대 경로이고 `dirfd`의 값이 `AT_FDCWD`이면 현재 작업 디렉터리를 기준으로 `pathname`을 해석해서 표시할 파일 시스템 객체를 결정한다.

## RETURN VALUE

성공 시 `fanotify_mark()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EBADF</code></dt>
<dd><code>fanotify_fd</code>로 유효하지 않은 파일 디스크립터를 줬다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>flags</code>나 <code>mask</code>에 유효하지 않은 값을 줬다. 또는 <code>fanotify_fd</code>가 fanotify 파일 디스크립터가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd>fanotify 파일 디스크립터를 <code>FAN_CLASS_NOTIF</code>나 <code>FAN_REPORT_FID</code>로 열였는데 <code>mask</code>에 허가 이벤트를 위한 플래그(<code>FAN_OPEN_PERM</code>이나 <code>FAN_ACCESS_PERM</code>)가 있다.</dd>
<dt><code>ENOENT</code></dt>
<dd><code>dirfd</code>와 <code>pathname</code>으로 나타낸 파일 시스템 객체가 존재하지 않는다. 표시 안 된 객체에서 표시를 제거하려 할 때도 이 오류가 생긴다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>필요한 메모리를 할당할 수 없다.</dd>
<dt><code>ENOSPC</code></dt>
<dd>표시 개수가 제한값인 8192개를 초과했으며 <tt>[[fanotify_init(2)]]</tt>으로 fanotify 파일 디스크립터를 생성할 때 <code>FAN_UNLIMITED_MARKS</code> 플래그를 지정하지 않았다.</dd>
<dt><code>ENOSYS</code></dt>
<dd>커널에서 <code>fanotify_mark()</code>를 구현하고 있지 않다. 커널을 <code>CONFIG_FANOTIFY</code>로 구성한 경우에만 fanotify API를 쓸 수 있다.</dd>
<dt><code>ENOTDIR</code></dt>
<dd><code>flags</code>에 <code>FAN_MARK_ONLYDIR</code>이 있는데 <code>dirfd</code>와 <code>pathname</code>이 디렉터리를 나타내지 않는다.</dd>
<dt><code>EXDEV</code></dt>
<dd><code>pathname</code>이 가리키는 파일 시스템 객체가 루트 수퍼블록과 다른 <code>fsid</code>를 쓰는 파일 시스템 서브볼륨에 위치해 있다. (예를 들면 <code>btrfs(5)</code>.) <tt>[[fanotify_init(2)]]</tt>이 반환한 fanotify 파일 디스크립터를 <code>FAN_REPORT_FID</code>로 생성했을 때만 이 오류가 반환될 수 있다.</dd>
<dt><code>ENODEV</code></dt>
<dd><code>pathname</code>이 가리키는 파일 시스템 객체가 <code>fsid</code>를 지원하는 파일 시스템에 연계돼 있지 않다. (예를 들면 <tt>[[tmpfs(5)]]</tt>.) <tt>[[fanotify_init(2)]]</tt>이 반환한 fanotify 파일 디스크립터를 <code>FAN_REPORT_FID</code>로 생성했을 때만 이 오류가 반환될 수 있다.</dd>
<dt><code>EOPNOTSUPP</code></dt>
<dd><code>pathname</code>이 가리키는 객체가 파일 핸들 인코딩을 지원하지 않는 파일 시스템에 연계돼 있다. <tt>[[fanotify_init(2)]]</tt>이 반환한 fanotify 파일 디스크립터를 <code>FAN_REPORT_FID</code>로 생성했을 때만 이 오류가 반환될 수 있다.</dd>
</dl>

## VERSIONS

리눅스 커널 버전 2.6.36에서 `fanotify_mark()`가 도입되었고 버전 2.6.37에서 활성화되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

### `FAN_OPEN_EXEC`와 `FAN_OPEN_EXEC_PERM`

`mask`에 `FAN_OPEN_EXEC`나 `FAN_OPEN_EXEC_PERM`를 사용 시에는 프로그램 직접 실행이 이뤄질 때만 그 이벤트들이 반환된다. 구체적으로 말해 <tt>[[execve(2)]]</tt>나 <tt>[[execveat(2)]]</tt>, <tt>[[uselib(2)]]</tt>로 열리는 파일에 대해 그 이벤트들이 생성된다. 인터프리터가 스크립트 파일을 전달받는 (또는 읽어 들이는) 경우에는 그 이벤트들이 생기지 않는다.

그리고 리눅스 동적 링커에도 표시가 돼 있다면 <tt>[[execve(2)]]</tt>나 <tt>[[execveat(2)]]</tt>을 이용해 성공적으로 ELF 오브젝트가 열릴 때도 이벤트를 받을 걸 예상해야 한다.

예를 들어 다음 ELF 바이너리를 호출하려 하고 `/`에 `FAN_OPEN_EXEC` 표시를 해 뒀다고 하자.

```
$ /bin/echo foo
```

이 경우 이벤트 수신 응용은 그 ELF 바이너리와 인터프리터 각각에 대해 `FAN_OPEN_EXEC` 이벤트를 받게 된다.

```
/bin/echo
/lib64/ld-linux-x86-64.so.2
```

## BUGS

리눅스 커널 버전 3.16 전에 다음 버그들이 있었다.

* `flags`에 `FAN_MARK_FLUSH`가 있는 경우 쓰지도 않는데 `dirfd`와 `pathname`이 유효한 파일 시스템 객체를 나타내야 한다.

* <tt>[[readdir(2)]]</tt>이 `FAN_ACCESS` 이벤트를 생성하지 않는다.

* `fanotify_mark()`를 `FAN_MARK_FLUSH`로 호출하는 경우 `flags` 값이 유효한지 확인하지 않는다.

## SEE ALSO

<tt>[[fanotify_init(2)]]</tt>, <tt>[[fanotify(7)]]</tt>

----

2019-08-02
