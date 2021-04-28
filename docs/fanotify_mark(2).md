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

`FAN_MARK_ADD`
:   표시 마스크에 (또는 무시 마스크에) `mask`의 이벤트들을 추가한다. `mask`가 비어 있지 않아야 하며, 안 그러면 `EINVAL` 오류가 발생한다.

`FAN_MARK_REMOVE`
:   표시 마스크에서 (또는 무시 마스크에서) `mask` 인자의 이벤트들을 제거한다. `mask`가 비어 있지 않아야 하며, 안 그러면 `EINVAL` 오류가 발생한다.

`FAN_MARK_FLUSH`
:   fanotify 그룹에서 모든 파일 시스템 표시, 또는 모든 마운트 표시, 또는 모든 디렉터리 및 파일 표시를 제거한다. `flags`에 `FAN_MARK_MOUNT`가 있으면 그룹에서 마운트에 대한 표시들을 모두 제거한다. `flags`에 `FAN_MARK_FILESYSTEM`이 있으면 그룹에서 파일 시스템에 대한 표시들을 모두 제거한다. 그렇지 않으면 디렉터리 및 파일에 대한 표시들을 모두 제거한다. `FAN_MARK_MOUNT`와 `FAN_MARK_FILESYSTEM` 외의 플래그나 두 플래그 모두를 `FAN_MARK_FLUSH`와 함께 사용할 수 없다. `mask`는 무시한다.

위 값들 중 아무것도 지정하지 않거나 여러 가지를 지정하면 호출이 `EINVAL` 오류로 실패한다.

추가로 `flags`에 다음 값들을 0개 이상 OR 할 수 있다.

`FAN_MARK_DONT_FOLLOW`
:   `pathname`이 심볼릭 링크면 가리키는 파일이 아니라 링크 자체에 표시를 한다. (기본적으로 `fanotify_mark()`는 `pathname`이 심볼릭 링크면 따라간다.)

`FAN_MARK_ONLYDIR`
:   표시할 파일 시스템 객체가 디렉터리가 아니면 `ENOTDIR` 오류를 제기한다.

`FAN_MARK_MOUNT`
:   `pathname`으로 지정한 마운트 지점에 표시를 한다. `pathname` 자체가 마운트 지점이 아니면 `pathname`을 담은 마운트 지점에 표시를 하게 된다. 마운트 지점의 모든 디렉터리와 서브디렉터리, 그리고 거기 담긴 파일들을 감시한다. `flags`에 `FAN_MARK_MOUNT`가 있을 때는 `FAN_CREATE`, `FAN_ATTRIB`, `FAN_MOVE`, `FAN_DELETE_SELF`처럼 파일 핸들로 파일 시스템 객체를 식별해야 하는 이벤트를 줄 수 없다. 그렇게 해서 시도하면 오류 `EINVAL`이 반환된다.

`FAN_MARK_FILESYSTEM` (리눅스 4.20부터)
:   `pathname`으로 지정한 파일 시스템에 표시를 한다. `pathname`을 담은 파일 시스템에 표시를 하게 된다. 마운트 지점이 어디든 그 파일 시스템에 담긴 모든 파일과 디렉터리를 감시한다.

`FAN_MARK_IGNORED_MASK`
:   무시 마스크에서 `mask`의 이벤트들을 추가하거나 제거한다.

`FAN_MARK_IGNORED_SURV_MODIFY`
:   무시 마스크가 변경 이벤트를 거치며 유지된다. 이 플래그가 설정돼 있지 않으면 무시하는 파일이나 디렉터리에 변경 이벤트가 발생할 때 그 무시 마스크를 없앤다.

`mask`는 어떤 이벤트를 청취할지 (또는 무시할지) 지정한다. 다음 값들로 이뤄진 비트 마스크이다.

`FAN_ACCESS`
:   파일이나 디렉터리에 (BUGS 참고) 접근(읽기)이 이뤄지면 이벤트 생성.

`FAN_MODIFY`
:   파일이 변경(쓰기)되면 이벤트 생성.

`FAN_CLOSE_WRITE`
:   쓰기 가능 파일이 닫히면 이벤트 생성.

`FAN_CLOSE_NOWRITE`
:   읽기 전용 파일이나 디렉터리가 닫히면 이벤트 생성.

`FAN_OPEN`
:   파일이나 디렉터리가 열리면 이벤트 생성.

`FAN_OPEN_EXEC` (리눅스 5.0부터)
:   파일이 실행하려는 의도로 열리면 이벤트 생성. NOTES의 추가 설명 참고.

`FAN_ATTRIB` (리눅스 5.1부터)
:   파일이나 디렉터리의 메타데이터가 바뀌었을 때 이벤트 생성. 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹이 필요하다.

`FAN_CREATE` (리눅스 5.1부터)
:   표시된 부모 디렉터리 내에서 파일이나 디렉터리가 생성됐을 때 이벤트 생성. 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹이 필요하다.

`FAN_DELETE` (리눅스 5.1부터)
:   표시된 부모 디렉터리 내에서 파일이나 디렉터리가 삭제됐을 때 이벤트 생성. 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹이 필요하다.

`FAN_DELETE_SELF` (리눅스 5.1부터)
:   표시된 파일이나 디렉터리 자체가 삭제됐을 때 이벤트 생성. 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹이 필요하다.

`FAN_MOVED_FROM` (리눅스 5.1부터)
:   표시된 부모 디렉터리에 있던 파일이나 디렉터리가 이동됐을 때 이벤트 생성. 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹이 필요하다.

`FAN_MOVED_TO` (리눅스 5.1부터)
:   표시된 부모 디렉터리로 파일이나 디렉터리가 이동됐을 때 이벤트 생성. 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹이 필요하다.

`FAN_MOVE_SELF` (리눅스 5.1부터)
:   표시된 파일이나 디렉터리 자체가 이동됐을 때 이벤트 생성. 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹이 필요하다.

`FAN_OPEN_PERM`
:   파일이나 디렉터리 열기 요청이 있으면 이벤트 생성. `FAN_CLASS_PRE_CONTENT`나 `FAN_CLASS_CONTENT`로 생성한 fanotify 파일 디스크립터가 필요하다.

`FAN_OPEN_EXEC_PERM` (리눅스 5.0부터)
:   실행을 위한 파일 열기 요청이 있으면 이벤트 생성. `FAN_CLASS_PRE_CONTENT`나 `FAN_CLASS_CONTENT`로 생성한 fanotify 파일 디스크립터가 필요하다. NOTES의 추가 설명 참고.

`FAN_ACCESS_PERM`
:   파일이나 디렉터리 읽기 요청이 있으면 이벤트 생성. `FAN_CLASS_PRE_CONTENT`나 `FAN_CLASS_CONTENT`로 생성한 fanotify 파일 디스크립터가 필요하다.

`FAN_ONDIR`
:   디렉터리에 대해 (가령 <tt>[[opendir(3)]]</tt>, <tt>[[readdir(3)]]</tt> (단 BUGS 참고), <tt>[[closedir(3)]]</tt>이 호출될 때) 이벤트 생성. 이 플래그가 없으면 파일에 대해서만 이벤트가 생긴다. `FAN_CREATE`, `FAN_DELETE`, `FAN_MOVED_FROM`, `FAN_MOVED_TO` 같은 디렉터리 항목 이벤트 맥락에서 서브디렉터리 항목들이 변경(즉 <tt>[[mkdir(2)]]</tt>/<tt>[[rmdir(2)]]</tt>)될 때 이벤트가 생성되게 하려면 `FAN_ONDIR` 플래그를 지정해야 한다.

`FAN_EVENT_ON_CHILD`
:   표시한 디렉터리의 직접 자식들에 대한 이벤트를 생성한다. 마운트 및 파일 시스템에 표시할 때는 이 플래그에 아무 효과가 없다. 표시된 디렉터리의 서브디렉터리의 자식들에 대해 이벤트가 생성되지 않는다는 점에 유의하라. 구체적으로 표시된 디렉터리의 서브디렉터리 내에서 수행된 항목 변경에 대해 디렉터리 항목 변경 이벤트인 `FAN_CREATE`, `FAN_DELETE`, `FAN_MOVED_FROM`, `FAN_MOVED_TO`가 생성되지 않는다. 표시된 디렉터리의 자식들에 대해 `FAN_DELETE_SELF` 및 `FAN_MOVE_SELF` 이벤트가 생성되지 않는다는 점에 유의하라. 디렉터리 트리 전체를 감시하려면 적절한 마운트 내지 파일 시스템에 표시를 해야 한다.

다음 조합 값이 정의돼 있다.

`FAN_CLOSE`
:   파일이 닫혔음. (`FAN_CLOSE_WRITE|FAN_CLOSE_NOWRITE`)

`FAN_MOVE`
:   파일이나 디렉터리가 이동됐음. (`FAN_MOVED_FROM|FAN_MOVED_TO`)

파일 디스크립터 `dirfd`와 `pathname`에 지정한 경로명을 써서 표시를 할 파일 시스템 객체를 결정한다.

* `pathname`이 NULL이면 `dirfd`가 표시할 파일 시스템 객체를 결정한다.

* `pathname`이 NULL이고 `dirfd`가 특수 값 `AT_FDCWD`이면 현재 작업 디렉터리에 표시를 한다.

* `pathname`이 절대 경로이면 표시할 파일 시스템 객체를 결정하며, `dirfd`는 무시된다.

* `pathname`이 상대 경로이고 `dirfd`의 값이 `AT_FDCWD`가 아니면 `dirfd`가 가리키는 디렉터리를 기준으로 `pathname`을 해석해서 표시할 파일 시스템 객체를 결정한다.

* `pathname`이 상대 경로이고 `dirfd`의 값이 `AT_FDCWD`이면 현재 작업 디렉터리를 기준으로 `pathname`을 해석해서 표시할 파일 시스템 객체를 결정한다.

## RETURN VALUE

성공 시 `fanotify_mark()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   `fanotify_fd`로 유효하지 않은 파일 디스크립터를 줬다.

`EINVAL`
:   `flags`나 `mask`에 유효하지 않은 값을 줬다. 또는 `fanotify_fd`가 fanotify 파일 디스크립터가 아니다.

`EINVAL`
:   `FAN_CLASS_NOTIF`로, 또는 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹으로 fanotify 파일 디스크립터를 열였는데 `mask`에 허가 이벤트를 위한 플래그(`FAN_OPEN_PERM`이나 `FAN_ACCESS_PERM`)가 있다.

`ENODEV`
:   `pathname`이 가리키는 파일 시스템 객체가 `fsid`를 지원하는 파일 시스템에 연계돼 있지 않다. (예를 들면 <tt>[[tmpfs(5)]]</tt>.) 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹에서만 이 오류가 반환될 수 있다.

`ENOENT`
:   `dirfd`와 `pathname`으로 나타낸 파일 시스템 객체가 존재하지 않는다. 표시 안 된 객체에서 표시를 제거하려 할 때도 이 오류가 생긴다.

`ENOMEM`
:   필요한 메모리를 할당할 수 없다.

`ENOSPC`
:   표시 개수가 제한값인 8192개를 초과했으며 <tt>[[fanotify_init(2)]]</tt>으로 fanotify 파일 디스크립터를 생성할 때 `FAN_UNLIMITED_MARKS` 플래그를 지정하지 않았다.

`ENOSYS`
:   커널에서 `fanotify_mark()`를 구현하고 있지 않다. 커널을 `CONFIG_FANOTIFY`로 구성한 경우에만 fanotify API를 쓸 수 있다.

`ENOTDIR`
:   `flags`에 `FAN_MARK_ONLYDIR`이 있는데 `dirfd`와 `pathname`이 디렉터리를 나타내지 않는다.

`EOPNOTSUPP`
:   `pathname`이 가리키는 객체가 파일 핸들 인코딩을 지원하지 않는 파일 시스템에 연계돼 있다. 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹에서만 이 오류가 반환될 수 있다.

`EXDEV`
:   `pathname`이 가리키는 파일 시스템 객체가 루트 수퍼블록과 다른 `fsid`를 쓰는 파일 시스템 서브볼륨에 위치해 있다. (예를 들면 `btrfs(5)`.) 파일 핸들로 파일 시스템 객체를 식별하는 fanotify 그룹에서만 이 오류가 반환될 수 있다.

## VERSIONS

리눅스 커널 버전 2.6.36에서 `fanotify_mark()`가 도입되었고 버전 2.6.37에서 활성화되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

### `FAN_OPEN_EXEC`와 `FAN_OPEN_EXEC_PERM`

`mask`에 `FAN_OPEN_EXEC`나 `FAN_OPEN_EXEC_PERM`를 사용 시에는 프로그램 직접 실행이 이뤄질 때만 그 이벤트들이 반환된다. 구체적으로 말해 <tt>[[execve(2)]]</tt>나 <tt>[[execveat(2)]]</tt>, <tt>[[uselib(2)]]</tt>로 열리는 파일에 대해 그 이벤트들이 생성된다. 인터프리터가 파일을 전달받는 (또는 읽어 들이는) 경우에는 그 이벤트들이 생기지 않는다.

그리고 리눅스 동적 링커에도 표시가 돼 있다면 <tt>[[execve(2)]]</tt>나 <tt>[[execveat(2)]]</tt>을 이용해 성공적으로 ELF 오브젝트가 열릴 때도 이벤트를 받을 걸 예상해야 한다.

예를 들어 다음 ELF 바이너리를 호출하려 하고 `/`에 `FAN_OPEN_EXEC` 표시를 해 뒀다고 하자.

```text
$ /bin/echo foo
```

이 경우 이벤트 수신 응용은 그 ELF 바이너리와 인터프리터 각각에 대해 `FAN_OPEN_EXEC` 이벤트를 받게 된다.

```text
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

2021-03-22
