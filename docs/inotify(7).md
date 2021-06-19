## NAME

inotify - 파일 시스템 이벤트 감시하기

## DESCRIPTION

*inotify* API는 파일 시스템 이벤트를 감시할 수 있는 메커니즘을 제공한다. inotify를 이용해 개별 파일을 감시하거나 디렉터리를 감시할 수 있다. 디렉터리를 감시할 때는 디렉터리 자체와 그 디렉터리 내 파일들에 대한 이벤트를 반환하게 된다.

이 API에서 사용하는 시스템 호출은 다음과 같다.

* <tt>[[inotify_init(2)]]</tt>은 inotify 인스턴스를 만들고 그 inotify 인스턴스를 가리키는 파일 디스크립터를 반환한다. 더 최신인 <tt>[[inotify_init1(2)]]</tt>은 <tt>[[inotify_init(2)]]</tt>과 비슷하되 `flags` 인자가 있어서 몇 가지 추가 기능을 사용할 수 있다.

* <tt>[[inotify_add_watch(2)]]</tt>는 inotify 인스턴스에 연계된 "감시 목록(watch list)"을 조작한다. 감시 목록 내의 각 항목은 파일이나 디렉터리의 경로명과 함께 그 경로명이 가리키는 파일에 대해 커널에서 감시해야 할 어떤 이벤트 집합을 나타낸다. <tt>[[inotify_add_watch(2)]]</tt>는 새 감시 항목을 만들거나 기존 감시 항목을 변경한다. 각 감시 항목에는 고유한 "감시 디스크립터"가 있는데, 그 감시 항목을 생성할 때 <tt>[[inotify_add_watch(2)]]</tt>가 그 정수를 반환한다.

* 감시 대상 파일 및 디렉터리에 대한 이벤트가 발생할 때 inotify 파일 디스크립터에서 <tt>[[read(2)]]</tt>로 그 이벤트를 나타내는 구조화된 데이터를 읽어 들일 수 있다. (아래 참고.)

* <tt>[[inotify_rm_watch(2)]]</tt>는 inotify 감시 목록에서 항목을 제거한다.

* inotify 인스턴스를 가리키는 모든 파일 디스크립터가 (<tt>[[close(2)]]</tt>로) 닫히면 커널에서 기반 객체와 자원을 재사용할 수 있게 해제한다. 연계된 감시 항목 모두가 자동으로 해제된다.

조심스럽게 프로그래밍 한다면 응용에서 inotify를 이용해 파일 시스템 객체들의 상태를 효율적으로 감시하고 캐싱 할 수 있다. 하지만 견고한 응용이라면 감시 로직의 버그나 아래에서 설명할 종류의 경쟁 때문에 캐시가 파일 시스템 상태와 일치하지 않게 될 수도 있다는 점을 감안해야 한다. 아마 어떤 일관성 검사를 해서 불일치 탐지 시 캐시를 재구축하는 게 바람직할 것이다.

## inotify 파일 디스크립터에서 이벤트 읽기

어떤 이벤트가 발생했는지 알아내기 위해 응용에서는 inotify 파일 디스크립터에 <tt>[[read(2)]]</tt>를 한다. 아직 어떤 이벤트도 일어나지 않은 경우 블로킹 파일 디스크립터라고 하면 적어도 한 개 이벤트가 발생할 때까지 <tt>[[read(2)]]</tt>가 블록 하게 된다. (단 시그널에 의해 중단되는 경우에는 호출이 `EINTR` 오류로 실패한다. <tt>[[signal(7)]]</tt> 참고.)

<tt>[[read(2)]]</tt>가 성공하면 다음 구조체를 한 개 이상 담은 버퍼를 반환한다.

```c
struct inotify_event {
    int      wd;       /* 감시 디스크립터 */
    uint32_t mask;     /* 이벤트 기술 마스크 */
    uint32_t cookie;   /* 관련 이벤트들을 연계해 주는
                          유일한 쿠키 값 (rename(2)) */
    uint32_t len;      /* name 필드 크기 */
    char     name[];   /* 선택적인 널 종료 이름 */
};
```

`wd`는 이벤트가 발생한 감시 대상을 식별해 준다. 앞서 <tt>[[inotify_add_watch(2)]]</tt> 호출이 반환한 감시 디스크립터들 중 하나이다.

`mask`는 발생한 이벤트를 기술하는 비트들을 담고 있다. (아래 참고.)

`cookie`는 관련 이벤트들을 연결해 주는 유일 정수이다. 현재는 rename 이벤트에서만 쓰는데 발생한 `IN_MOVED_FROM`와 `IN_MOVED_TO` 이벤트 쌍을 응용에서 연관 지을 수 있게 해 준다. 다른 모든 이벤트들에서는 `cookie`가 0으로 설정돼 있다.

`name` 필드는 감시 대상 디렉터리 내 파일에 대한 이벤트가 반환될 때만 존재하다. 즉 감시 대상 디렉터리 내의 파일명을 나타낸다. 이 파일명은 널로 끝나며 후속 항목을 적절한 주소 경계로 정렬시키기 위해 추가로 널 바이트('\0')들이 있을 수도 있다.

`len` 필드는 널 바이트를 포함해 `name`의 바이트 모두의 개수이다. 따라서 각 `inotify_event` 구조체의 길이는 `sizeof(struct inotify_event)+len`이다.

<tt>[[read(2)]]</tt>에 준 버퍼가 너무 작아서 다음 이벤트에 대한 정보를 담을 수 없는 경우의 동작 방식은 커널 버전에 따라 다르다. 2.6.21 전의 커널에서는 <tt>[[read(2)]]</tt>가 0을 반환하며 커널 2.6.21부터는 <tt>[[read(2)]]</tt>가 `EINVAL` 오류로 실패한다. 다음 크기의 버퍼를 지정하면 적어도 이벤트 한 개는 확실히 읽을 수 있다.

```c
sizeof(struct inotify_event) + NAME_MAX + 1
```

### inotify 이벤트

<tt>[[inotify_add_watch(2)]]</tt>의 `mask` 인자와 inotify 파일 디스크립터에서 <tt>[[read(2)]]</tt> 할 때 반환되는 `inotify_event` 구조체의 `mask` 필드는 inotify 이벤트들을 나타내는 비트 마스크이다. <tt>[[inotify_add_watch(2)]]</tt> 호출 시 `mask` 마스크에 다음 비트들을 지정할 수 있으며 <tt>[[read(2)]]</tt>에서 반환하는 `mask` 필드로 반환될 수 있다.

`IN_ACCESS` (+)
:   파일에 접근했음. (가령 <tt>[[read(2)]]</tt>, <tt>[[execve(2)]]</tt>.)

`IN_ATTRIB` (\*)
:   타데이터 변경됐음. 예를 들어 권한(<tt>[[chmod(2)]]</tt>), 타임스탬프(<tt>[[utimensat(2)]]</tt>), 확장 속성(<tt>[[setxattr(2)]]</tt>), 링크 카운트(리눅스 2.6.25부터, <tt>[[link(2)]]</tt> 및 <tt>[[unlink(2)]]</tt> 대상에 대해), 사용자/그룹 ID(<tt>[[chown(2)]]</tt>).

`IN_CLOSE_WRITE` (+)
:   쓰기용으로 열린 파일이 닫혔음.

`IN_CLOSE_NOWRITE` (\*)
:   쓰기용으로 열리지 않은 파일 내지 디렉터리가 닫혔음.

`IN_CREATE` (+)
:   감시 대상 디렉터리 내에 파일/디렉터리 생성됐음. (가령 <tt>[[open(2)]]</tt> `O_CREAT`, `mkdir(2)`, <tt>[[link(2)]]</tt>, <tt>[[symlink(2)]]</tt>, 유닉스 도메인 소켓 `bind(2)`.)

`IN_DELETE` (+)
:   감시 대상 디렉터리에서 파일/디렉터리가 삭제됐음.

`IN_DELETE_SELF`
:   감시 대상 파일/디렉터리 자체가 삭제됐음. (다른 파일 시스템으로 객체를 옮기는 경우에도 이 이벤트가 발생한다. 그 경우 `mv(1)`는 실질적으로는 다른 파일 시스템으로 파일을 복사한 다음 원래 파일 시스템에서 파일을 삭제하기 때문이다.) 더불어 그 감시 디스크립터로 이어서 `IN_IGNORED` 이벤트가 생성될 것이다.

`IN_MODIFY` (+)
:   파일이 변경됐음. (가령 <tt>[[write(2)]]</tt>, <tt>[[truncate(2)]]</tt>.)

`IN_MOVE_SELF`
:   감시 대상 파일/디렉터리 자체가 이동됐음.

`IN_MOVED_FROM` (+)
:   파일 이름이 바뀔 때 이전 파일명을 담은 디렉터리에서 생성됨.

`IN_MOVED_TO` (+)
:   파일 이름이 바뀔 때 새 파일명을 담은 디렉터리에서 생성됨.

`IN_OPEN` (\*)
:   파일 내지 디렉터리가 열렸음.

inotify 감시는 아이노드를 기준으로 한다. 어떤 파일을 감시할 때 (파일을 담은 디렉터리를 감시하는 게 아닐 때) 그 파일에 대한 (같거나 다른 디렉터리의) 어느 링크의 활동에 의해서도 이벤트가 발생할 수 있다.

디렉터리를 감시할 때,

* 위에 별표(\*)가 된 이벤트는 디렉터리 자체와 디렉터리 내 객체에 대해 생길 수 있다.

* 더하기 표시(+)가 된 이벤트는 디렉터리 내 객체에 대해서만 생길 수 있다. (디렉터리 자체에는 생기지 않는다.)

*참고*: 디렉터리를 감시할 때 그 디렉터리 밖에 위치한 경로명(즉 링크)을 통해 이벤트가 수행될 때는 그 디렉터리 내 파일에 대해 이벤트가 생성되지 않는다.

감시 대상 디렉터리 내 객체에 대해 이벤트가 생성될 때 반환되는 `inotify_event` 구조체의 `name` 필드가 디렉터리 내에서 파일의 이름을 나타낸다.

위 이벤트 모두에 해당하는 비트 마스크로 `IN_ALL_EVENTS` 매크로가 정의돼 있다. <tt>[[inotify_add_watch(2)]]</tt>를 호출할 때 그 매크로를 `mask` 인자로 쓸 수 있다.

편의를 위한 매크로가 두 가지 더 정의돼 있다.

`IN_MOVE`
:   `IN_MOVED_FROM | IN_MOVED_TO`와 같음.

`IN_CLOSE`
:   `IN_CLOSE_WRITE | IN_CLOSE_NOWRITE`와 같음.

<tt>[[inotify_add_watch(2)]]</tt>를 호출할 때 `mask` 인자에 다음 비트들을 추가로 지정할 수 있다.

`IN_DONT_FOLLOW` (리눅스 2.6.16부터)
:   `pathname`이 심볼릭 링크인 경우 따라가지 않는다.

`IN_EXCL_UNLINK` (리눅스 2.6.36부터)
:   기본적으로 디렉터리의 자식들에 대해 이벤트를 감시할 때는 그 디렉터리에서 자식들을 링크 제거(unlink)한 후에도 자식들에 대한 이벤트가 생성된다. 어떤 응용에서는 (가령 여러 응용들이 임시 파일을 만들고 그 이름을 즉시 링크 제거하는 `/tmp`를 감시할 때) 이로 인해 관심 없는 이벤트가 매우 많이 생겨날 수 있다. `IN_EXCL_UNLINK`를 지정해서 기본 동작을 바꾸면 감시 대상 디렉터리에서 자식이 링크 제거된 후에는 이벤트가 생성되지 않는다.

`IN_MASK_ADD`
:   `pathname`에 대응하는 파일 시스템 객체에 대해 감시 인스턴스가 이미 존재하는 경우 (마스크를 교체하는 게 아니라) 감시 마스크에 `mask`의 이벤트들을 추가(OR)한다. `IN_MASK_CREATE`도 지정돼 있으면 `EINVAL` 오류가 발생한다.

`IN_ONESHOT`
:   `pathname`에 대응하는 파일 시스템 객체에서 이벤트를 한 개만 감시한 다음 그 객체를 감시 목록에서 제거한다.

`IN_ONLYDIR` (리눅스 2.6.15부터)
:   `pathname`이 디렉터리인 경우에만 감시한다. `pathname`이 디렉터리가 아니면 `ENOTDIR` 오류가 발생한다. 이 플래그를 이용하면 경쟁 문제가 없는 방식으로 감시 대상이 디렉터리임을 보장할 수 있다.

`IN_MASK_CREATE` (리눅스 4.18부터)
:   `pathname`에 연관된 감시 항목이 없는 경우에만 감시한다. `pathname`을 이미 감시하고 있으면 `EEXIST` 오류가 발생한다.

    이 플래그를 이용하면 응용에서 새 감시 항목이 기존 항목을 변경하지 않게 할 수 있다. 그게 유용한 건 여러 경로가 같은 아이노드를 가리키고 있을 수도 있기 때문이다. 이 플래그 없이 <tt>[[inotify_add_watch(2)]]</tt>를 여러 번 호출하면 기존 감시 마스크를 망가뜨릴 수 있다.

<tt>[[read(2)]]</tt>로 받는 `mask` 필드에 다음 비트들이 설정돼 있을 수 있다.

`IN_IGNORED`
:   감시 대상이 명시적으로(<tt>[[inotify_rm_watch(2)]]</tt>) 또는 자동으로(파일이 삭제되거나 파일 시스템이 마운트 해제됐음) 제거되었다. BUGS도 참고.

`IN_ISDIR`
:   이 이벤트의 대상이 디렉터리다.

`IN_Q_OVERFLOW`
:   이벤트 큐가 넘쳤다. (이 이벤트에선 `wd`가 -1임.)

`IN_UNMOUNT`
:   감시 대상 객체를 담은 파일 시스템이 마운트 해제되었다. 더불어 감시 디스크립터에 대해 `IN_IGNORED` 이벤트가 이어서 생성될 것이다.

### 예시

응용에서 디렉터리 `dir`과 파일 `dir/myfile`의 모든 이벤트를 감시하려 한다고 하자. 아래 예는 그 두 객체에 대해 생성될 몇몇 이벤트를 보여 준다.

`fd = open("dir/myfile", O_RDWR);`
:   `dir`과 `dir/myfile` 모두에 `IN_OPEN` 이벤트 생성.

`read(fd, buf, count);`
:   `dir`과 `dir/myfile` 모두에 `IN_ACCESS` 이벤트 생성.

`write(fd, buf, count);`
:   `dir`과 `dir/myfile` 모두에 `IN_MODIFY` 이벤트 생성.

`fchmod(fd, mode);`
:   `dir`과 `dir/myfile` 모두에 `IN_ATTRIB` 이벤트 생성.

`close(fd);`
:   `dir`과 `dir/myfile` 모두에 `IN_CLOSE_WRITE` 이벤트 생성.

응용에서 디렉터리 `dir1`과 `dir2`, 파일 `dir1/myfile`을 감시하려 한다고 하자. 다음 예는 생성될 수 있는 몇몇 이벤트를 보여 준다.

`link("dir1/myfile", "dir2/new");`
:   `myfile`에 `IN_ATTRIB` 이벤트와 `dir2`에 `IN_CREATE` 이벤트 생성.

`rename("dir1/myfile", "dir2/myfile");`
:   `dir1`에 `IN_MOVED_FROM` 이벤트, `dir2`에 `IN_MOVED_TO` 이벤트, `myfile`에 `IN_MOVE_SELF` 이벤트 생성. `IN_MOVED_FROM` 이벤트와 `IN_MOVED_TO` 이벤트의 `cookie` 값이 같게 된다.

`dir1/xx`와 `dir2/yy`가 같은 파일에 대한 (유이한) 링크이며 응용에서 `dir1`, `dir2`, `dir1/xx`, `dir2/yy`를 감시하려 한다고 하자. 다음 호출들을 차례로 실행하면 다음 이벤트들이 생성된다.

`unlink("dir2/yy");`
:   `xx`에 (링크 카운트가 바뀌므로) `IN_ATTRIB` 이벤트와 `dir2`에 `IN_DELETE` 이벤트 생성.

`unlink("dir1/xx");`
:   `xx`에 `IN_ATTRIB`, `IN_DELETE_SELF`, `IN_IGNORED` 이벤트, 그리고 `dir1`에 `IN_DELETE` 이벤트 생성.

응용에서 디렉터리 `dir`과 (빈) 디렉터리 `dir/subdir`을 감시하려 한다고 하자. 다음 예는 생성될 수 있는 몇몇 이벤트를 보여 준다.

`mkdir("dir/new", mode);`
:   `dir`에 `IN_CREATE | IN_ISDIR` 이벤트 생성.

`rmdir("dir/subdir");`
:   `subdir`에 `IN_DELETE_SELF` 및 `IN_IGNORED` 이벤트, 그리고 `dir`에 `IN_DELETE | IN_ISDIR` 이벤트 생성.

### `/proc` 인터페이스

다음 인터페이스를 이용해 inotify에서 쓰는 커널 메모리의 양을 제한할 수 있다.

`/proc/sys/fs/inotify/max_queued_events`
:   응용에서 <tt>[[inotify_init(2)]]</tt>을 호출할 때 이 파일의 값을 사용해 해당 inotify 인스턴스 큐에 들어갈 수 있는 이벤트 수의 상한을 정한다. 이 제한을 초과하는 이벤트들은 버려지되 항상 `IN_Q_OVERFLOW` 이벤트가 생성된다.

`/proc/sys/fs/inotify/max_user_instances`
:   실제 사용자 ID별로 만들 수 있는 inotify 인스턴스 수의 상한을 지정한다.

`/proc/sys/fs/inotify/max_user_watches`
:   실제 사용자 ID별로 만들 수 있는 감시 항목 수의 상한을 지정한다.

## VERSIONS

리눅스 커널 2.6.13으로 inotify가 병합되었다. glibc 버전 2.4에서 필요한 라이브러리 인터페이스가 추가되었다. (`IN_DONT_FOLLOW`, `IN_MASK_ADD`, `IN_ONLYDIR`은 glibc 버전 2.5에서 추가되었다.)

## CONFORMING TO

inotify API는 리눅스 전용이다.

## NOTES

<tt>[[select(2)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>로 inotify 파일 디스크립터를 감시할 수 있다.

리눅스 2.6.25부터 inotify 파일 디스크립터에 시그널 주도 I/O 알림을 사용할 수 있다. <tt>[[fcntl(2)]]</tt>의 `F_SETFL`(`O_ASYNC` 플래그 설정), `F_SETOWN`, `F_SETSIG` 설명을 보라. 시그널 핸들러에게 전달되는 `siginfo_t` 구조체(<tt>[[sigaction(2)]]</tt>에서 설명)에서 `si_fd`가 inotify 파일 디스크립터 번호로 설정돼 있고, `si_signo`가 시그널 번호로 설정돼 있고, `si_code`가 `POLL_IN`으로 설정돼 있고, `si_band`에 `POLLIN`이 설정돼 있다.

inotify 파일 디스크립터에서 연달아 생겨난 inotify 이벤트들이 동일한 (`wd`, `mask`, `cookie`, `name`이 같은) 경우 이전 이벤트가 아직 읽히지 않았으면 한 이벤트로 병합한다. (하지만 BUGS 참고.) 이렇게 하면 이벤트 큐에 필요한 커널 메모리 양이 줄기는 하지만 응용에서 inotify를 이용해 파일 이벤트 수를 정확히 셀 수 없다는 뜻이기도 하다.

inotify 파일 디스크립터에서 읽어서 반환되는 이벤트들은 순서 있는 큐를 형성한다. 그래서 예를 들어 한 디렉터리에서 다른 디렉터리로 이름을 바꿀 때 이벤트들이 inotify 파일 디스크립터 상에서 정확한 순서로 생겨난다고 보장된다.

inotify 파일 디스크립터를 통해 감시 중인 감시 디스크립터들의 목록을 프로세스의 `/proc/[pid]/fdinfo` 디렉터리 내 inotify 파일 디스크립터에 대한 항목을 통해 볼 수 있다. 자세한 내용은 <tt>[[proc(5)]]</tt> 참고. `FIONREAD` <tt>[[ioctl(2)]]</tt>은 inotify 파일 디스크립터에서 읽을 수 있는 바이트 수를 반환한다.

### 한계과 주의점

inotify API는 inotify 이벤트를 유발한 사용자나 프로세스에 대해선 어떤 정보도 제공하지 않는다. 특히 inotify를 통해 이벤트를 감시하고 있는 프로세스 자체에서 유발한 이벤트와 다른 프로세스가 유발한 이벤트를 구별할 손쉬운 방법이 없다.

inotify는 사용자 공간 프로그램이 파일 시스템 API를 통해 유발한 이벤트만을 보고한다. 따라서 네트워크 파일 시스템 상에서 일어나는 원격 이벤트는 잡지 못한다. (그런 이벤트를 잡으려면 파일 시스템 폴링으로 되돌아가야 한다.) 또한 `/proc`, `/sys`, `/dev/pts` 같은 여러 가상 파일 시스템들은 inotify로 감시할 수 없다.

inotify API는 <tt>[[mmap(2)]]</tt>, <tt>[[msync(2)]]</tt>, <tt>[[munmap(2)]]</tt> 때문에 일어날 수 있는 파일 접근 및 변경에 대해 알려 주지 않는다.

inotify API에서는 영향 받는 파일들을 파일명으로 구별해 준다. 하지만 응용에서 inotify 이벤트를 처리하는 시점에 그 파일명이 이미 삭제 내지 변경되었을 수도 있다.

inotify API에서는 감시 디스크립터를 통해 이벤트를 식별한다. (필요한 경우) 감시 디스크립터와 경로명 사이 매핑을 캐싱 하는 건 응용의 몫이다. 디렉터리 이름이 바뀌면 캐싱 된 여러 경로명이 영향을 받을 수 있다는 점에 유의하라.

inotify에서의 디렉터리 감시는 재귀적이지 않다. 디렉터리 아래 서브디렉터리를 감시하려면 감시 항목을 추가로 만들어야 한다. 커다란 디렉터리 트리에서는 상당한 시간이 걸릴 수 있다.

디렉터리 서브트리 전체를 감시 중이며 그 트리 내에 새 서브디렉터리가 생기거나 기존 디렉터리가 그 트리 안으로 이름이 바뀌는 경우에 그 새 서브디렉터리에 대한 감시 항목을 만드는 시점에 그 안에 이미 새 파일들이 (그리고 서브디렉터리들이) 존재할 수도 있다는 점에 유의해야 한다. 따라서 감시 항목을 추가한 직후에 그 서브디렉터리 내용을 훑어봐야 할 수도 (그리고 원한다면 그 안의 서브디렉터리들에 대해 재귀적으로 감시 항목을 추가할 수도) 있다.

이벤트 큐가 넘칠 수 있음에 유의하라. 이 경우 이벤트가 유실된다. 견고한 응용이라면 이벤트 유실 가능성을 우아하게 다뤄야 할 것이다. 예를 들어 응용 캐시의 일부 내지 전체를 재구축해야 할 수도 있다. (간단하지만 비용이 클 수 있는 방식 하나는 inotify 파일 디스크립터를 닫고, 캐시를 비우고, 새 inotify 파일 디스크립터를 만들고 나서 감시할 객체들에 대한 감시 항목과 캐시 항목을 다시 생성하는 것이다.)

감시 대상 디렉터리 위에 어떤 파일 시스템을 마운트 하는 경우 어떤 이벤트도 생성되지 않으며 그 새 마운트 지점 바로 아래 객체들에 대해 어떤 이벤트도 생성되지 않는다. 이후에 그 파일 시스템이 마운트 해제되면 그 뒤에 거기 담긴 디렉터리와 객체들에 대한 이벤트가 생성될 것이다.

### `rename()` 이벤트 다루기

위에서 언급한 것처럼 <tt>[[rename(2)]]</tt>이 생성하는 `IN_MOVED_FROM` 및 `IN_MOVED_TO` 이벤트를 공유 쿠키 값을 통해 짝을 맞출 수 있다. 하지만 짝을 맞추는 작업에 몇 가지 어려운 점이 있다.

inotify 파일 디스크립터에서 이벤트 스트림을 읽어 들일 때 이 두 이벤트는 일반적으로 연속으로 나온다. 하지만 그게 보장돼 있는 건 아니다. 어떤 감시 대상 객체에 여러 프로세스가 이벤트를 유발하는 경우에는 (드물지만) `IN_MOVED_FROM` 및 `IN_MOVED_TO` 이벤트 사이에 임의의 다른 이벤트들이 등장할 수 있다. 또한 그 이벤트 쌍이 큐에 원자적으로 삽입된다고 보장되지 않는다. 즉 `IN_MOVED_FROM`은 등장했지만 `IN_MOVED_TO`는 없는 짧은 기간이 있을 수 있다.

그래서 <tt>[[rename(2)]]</tt>으로 생성되는 `IN_MOVED_FROM` 및 `IN_MOVED_TO` 이벤트의 짝을 맞추는 데는 기본적으로 경쟁 요소가 있다. (게다가 객체의 이름이 감시 대상 디렉터리 밖으로 바뀌면 `IN_MOVED_TO` 이벤트가 없을 수도 있다.) 경험적 접근 방식(가령 이벤트가 항상 연속돼 있다고 가정하기)을 쓰면 대부분 경우에서 짝을 맞출 수 있지만 불가피하게 일부 경우를 놓치게 되어 응용에서 어떤 `IN_MOVED_FROM` 및 `IN_MOVED_TO` 이벤트 쌍이 서로 무관하다고 인식하게 된다. 그래서 감시 디스크립터를 파기하고 다시 만들면 그 감시 디스크립터가 미처리 이벤트의 감시 디스크립터와 일치하지 않게 된다. (이런 경우에는 inotify 파일 디스크립터를 다시 만들고 캐시를 재구축하는 게 도움이 될 수 있다.)

또한 응용에서는 이번 <tt>[[read(2)]]</tt> 호출이 반환하는 버퍼에 `IN_MOVED_FROM` 이벤트까지만 들어갈 수 있어서 딸린 `IN_MOVED_TO` 이벤트를 다음 <tt>[[read(2)]]</tt>로만 읽을 수 있을 가능성을 감안해야 한다. 이때 (작은) 타임아웃을 주는 게 좋은데, `IN_MOVED_FROM`+`IN_MOVED_TO` 이벤트 쌍 삽입이 원자적이지 않으며 `IN_MOVED_TO` 이벤트가 없을 가능성도 있기 때문이다.

## BUGS

리눅스 3.19 전에서 <tt>[[fallocate(2)]]</tt>가 inotify 이벤트를 생성하지 않았다. 리눅스 3.19부터 <tt>[[fallocate(2)]]</tt> 호출이 `IN_MODIFY` 이벤트를 생성한다.

커널 2.6.16 전에서 `mask` 플래그 `IN_ONESHOT`이 동작하지 않는다.

원래 설계와 구현에서는 `IN_ONESHOT` 플래그를 사용해서 한 이벤트 후에 감시 항목이 없어질 때 `IN_IGNORED` 이벤트가 생성되지 않았다. 하지만 다른 변경 내용들의 의도치 않은 효과 때문에 리눅스 2.6.36부터는 이 경우에 `IN_IGNORED` 이벤트가 생성된다.

커널 2.6.25 전에서 연속한 동일 이벤트(즉 오래된 쪽을 아직 읽지 않았으면 합쳐질 수도 있는 가장 최근의 두 이벤트)를 합치기 위한 커널 코드에서 가장 최근 이벤트와 *가장 오래된* 안 읽은 이벤트를 합칠 수 있는지 확인했다.

<tt>[[inotify_rm_watch(2)]]</tt> 호출로 (또는 감시 대상 파일이 삭제되거나 파일을 담은 파일 시스템이 마운트 해제돼서) 감시 디스크립터가 제거될 때 그 감시 디스크립터에 대한 안 읽은 미처리 이벤트가 있으면 계속 읽을 수 있는 상태로 남는다. 이어서 <tt>[[inotify_add_watch(2)]]</tt>로 감시 디스크립터를 할당하면 커널에서는 가능한 감시 디스크립터 범위(0에서 `INT_MAX`까지)를 차례로 돈다. 그런데 유휴 감시 디스크립터를 할당할 때 그 디스크립터에 대한 안 읽은 미처리 이벤트가 inotify 큐에 있는지 확인하지 않는다. 그래서 감시 디스크립터 번호의 이전 사용 때 안 읽은 미처리 이벤트가 존재하는데도 감시 디스크립터가 재할당되는 경우가 생길 수 있으며, 그 결과로 응용에서 그 이벤트들을 읽어서는 새로 재사용하는 감시 디스크립터에 연계된 파일에 속한 이벤트로 해석하게 될 수 있다. 실질적으로 이 버그가 발생할 가능성은 극히 낮은데, 응용에서 `INT_MAX` 개의 감시 디스크립터를 거쳐야 하며 큐에 안 읽은 이벤트를 남겨둔 채로 감시 디스크립터를 해제했다가 그 감시 디스크립터를 재사용해야 하기 때문이다. 그런 이유로, 그리고 실환경 응용에서 그 버그가 발생했다는 보고가 없었기 때문에 리눅스 3.15 기준으로 이 잠재적 버그를 제거하기 위한 어떤 커널 변경 작업도 이뤄지지 않았다.

## EXAMPLES

다음 프로그램은 inotify API 사용 방식을 보여 준다. 명령행 인자로 받은 디렉터리들에 표시를 하고 `IN_OPEN`, `IN_CLOSE_NOWRITE`, `IN_CLOSE_WRITE` 이벤트를 기다린다.

다음 출력은 파일 `/home/user/temp/foo`를 편집하고 디렉터리 `/tmp`를 나열하면서 기록한 것이다. 파일과 디렉터리를 열기 전에 `IN_OPEN` 이벤트가 일어난다. 파일을 닫은 후에 `IN_CLOSE_WRITE` 이벤트가 일어난다. 디렉터리를 닫은 후에 `IN_CLOSE_NOWRITE` 이벤트가 일어난다. 사용자가 엔터 키를 누르면 프로그램 실행이 끝난다.

### 예시 출력

```text
$ ./a.out /tmp /home/user/temp
Press enter key to terminate.
Listening for events.
IN_OPEN: /home/user/temp/foo [file]
IN_CLOSE_WRITE: /home/user/temp/foo [file]
IN_OPEN: /tmp/ [directory]
IN_CLOSE_NOWRITE: /tmp/ [directory]

Listening for events stopped.
```

### 프로그램 소스

```c
#include <errno.h>
#include <poll.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/inotify.h>
#include <unistd.h>
#include <string.h>

/* 파일 디스크립터 'fd'에서 가용 inotify 이벤트를 모두 읽어 들인다.
   wd는 argv 내 디렉터리들에 대한 감시 디스크립터들의 테이블이다.
   argc는 wd 및 argv의 길이이다.
   argv는 감시 대상 디렉터리들의 목록이다.
   wd 및 argv의 0번 항목은 쓰지 않는다. */

static void
handle_events(int fd, int *wd, int argc, char *argv[])
{
    /* 어떤 시스템에서는 올바로 정렬돼 있지 않으면 정수 변수를
       읽을 수 없다. 다른 시스템에서는 정렬이 안 맞으면 성능이
       떨어질 수 있다. 따라서 inotify 파일 디스크립터에서 읽을 때
       쓰는 버퍼의 정렬이 struct inotify_event와 같아야 한다. */

    char buf[4096]
        __attribute__ ((aligned(__alignof__(struct inotify_event))));
    const struct inotify_event *event;
    ssize_t len;

    /* inotify 파일 디스크립터에서 이벤트를 읽을 수 있는 동안 반복하기. */

    for (;;) {

        /* 이벤트 읽기 */

        len = read(fd, buf, sizeof(buf));
        if (len == -1 && errno != EAGAIN) {
            perror("read");
            exit(EXIT_FAILURE);
        }

        /* 논블로킹 read()에서 읽을 이벤트가 없으면 -1을 반환하며
           errno를 EAGAIN으로 설정한다. 그러면 루프에서 빠져나간다. */

        if (len <= 0)
            break;

        /* 버퍼 내 모든 이벤트 돌기. */

        for (char *ptr = buf; ptr < buf + len;
                ptr += sizeof(struct inotify_event) + event->len) {

            event = (const struct inotify_event *) ptr;

            /* 이벤트 타입 찍기. */

            if (event->mask & IN_OPEN)
                printf("IN_OPEN: ");
            if (event->mask & IN_CLOSE_NOWRITE)
                printf("IN_CLOSE_NOWRITE: ");
            if (event->mask & IN_CLOSE_WRITE)
                printf("IN_CLOSE_WRITE: ");

            /* 감시 대상 디렉터리 이름 찍기. */

            for (int i = 1; i < argc; ++i) {
                if (wd[i] == event->wd) {
                    printf("%s/", argv[i]);
                    break;
                }
            }

            /* 파일 이름 찍기. */

            if (event->len)
                printf("%s", event->name);

            /* 파일 시스템 객체 종류 찍기. */

            if (event->mask & IN_ISDIR)
                printf(" [directory]\n");
            else
                printf(" [file]\n");
        }
    }
}

int
main(int argc, char *argv[])
{
    char buf;
    int fd, i, poll_num;
    int *wd;
    nfds_t nfds;
    struct pollfd fds[2];

    if (argc < 2) {
        printf("Usage: %s PATH [PATH ...]\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    printf("Press ENTER key to terminate.\n");

    /* inotify API 접근을 위한 파일 디스크립터 만들기. */

    fd = inotify_init1(IN_NONBLOCK);
    if (fd == -1) {
        perror("inotify_init1");
        exit(EXIT_FAILURE);
    }

    /* 감시 디스크립터를 위한 메모리 할당하기. */

    wd = calloc(argc, sizeof(int));
    if (wd == NULL) {
        perror("calloc");
        exit(EXIT_FAILURE);
    }

    /* 디렉터리에 다음 이벤트 표시:
       - 파일 열림
       - 파일 닫힘 */

    for (i = 1; i < argc; i++) {
        wd[i] = inotify_add_watch(fd, argv[i],
                                  IN_OPEN | IN_CLOSE);
        if (wd[i] == -1) {
            fprintf(stderr, "Cannot watch '%s': %s\n",
                    argv[i], strerror(errno));
            exit(EXIT_FAILURE);
        }
    }

    /* 폴링 준비하기. */

    nfds = 2;

    fds[0].fd = STDIN_FILENO;       /* 콘솔 입력 */
    fds[0].events = POLLIN;

    fds[1].fd = fd;                 /* inotify 입력 */
    fds[1].events = POLLIN;

    /* 이벤트 및/또는 터미널 입력 기다리기. */

    printf("Listening for events.\n");
    while (1) {
        poll_num = poll(fds, nfds, -1);
        if (poll_num == -1) {
            if (errno == EINTR)
                continue;
            perror("poll");
            exit(EXIT_FAILURE);
        }

        if (poll_num > 0) {

            if (fds[0].revents & POLLIN) {

                /* 콘솔 입력 있음. stdin 비우고 끝내기. */

                while (read(STDIN_FILENO, &buf, 1) > 0 && buf != '\n')
                    continue;
                break;
            }

            if (fds[1].revents & POLLIN) {

                /* inotify 이벤트 있음. */

                handle_events(fd, wd, argc, argv);
            }
        }
    }

    printf("Listening for events stopped.\n");

    /* inotify 파일 디스크립터 닫기. */

    close(fd);

    free(wd);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`inotifywait(1)`, `inotifywatch(1)`, <tt>[[inotify_add_watch(2)]]</tt>, <tt>[[inotify_init(2)]]</tt>, <tt>[[inotify_init1(2)]]</tt>, <tt>[[inotify_rm_watch(2)]]</tt>, <tt>[[read(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[fanotify(7)]]</tt>

리눅스 커널 소스 트리의 `Documentation/filesystems/inotify.txt`

----

2021-03-22
