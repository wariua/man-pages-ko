## NAME

fanotify - 파일 시스템 이벤트 감시하기

## DESCRIPTION

fanotify API를 통해 파일 시스템 이벤트 알림을 받고 이벤트를 가로챌 수 있다. 사용례로는 바이러스 검사나 계층적 저장소 관리가 있을 수 있다. 현재는 제한된 종류의 이벤트들만 지원한다. 특히 생성, 삭제, 이동 이벤트를 지원하지 않는다. (이 이벤트들을 알려 주는 API에 대한 내용은 <tt>[[inotify(7)]]</tt>를 보라.)

<tt>[[inotify(7)]]</tt> API와 비교하자면 마운트 한 파일 시스템 내의 모든 객체들을 감시할 수 있고, 접근 허용 여부를 결정할 수 있으며, 다른 응용에서 접근하기 전에 파일을 읽거나 변경하는 게 가능하다.

이 API에서 쓰는 시스템 호출은 <tt>[[fanotify_init(2)]]</tt>, <tt>[[fanotify_mark(2)]]</tt>, `read(2)`, `write(2)`, <tt>[[close(2)]]</tt>이다.

### `fanotify_init()`, `fanotify_mark()`, 알림 그룹

<tt>[[fanotify_init(2)]]</tt> 시스템 호출은 fanotify 알림 그룹을 만들어서 초기화 하며 그 알림 그룹을 가리키는 파일 디스크립터를 반환한다.

fanotify 알림 그룹이란 커널 내부 객체로서 이벤트가 생성될 파일, 디렉터리, 파일 시스템, 마운트 지점들의 목록을 가지고 있다.

fanotify 알림 그룹 내의 각 항목마다 <em>표시(mark)</em> 마스크와 <em>무시(ignore)</em> 마스크라는 두 가지 비트 마스크가 있다. 표시 마스크는 이벤트가 생성될 파일 활동들을 규정한다. 무시 마스크는 이벤트가 생성되지 않을 활동들을 규정한다. 이렇게 두 가지 마스크가 있어서 어떤 파일 시스템이나 마운트 지점, 디렉터리에 이벤트를 받는다고 표시하면서 동시에 어떤 마운트 지점 내지 디렉터리 아래의 특정 객체들에 대한 이벤트를 무시할 수 있다.

<tt>[[fanotify_mark(2)]]</tt> 시스템 호출은 알림 그룹에 파일이나 디렉터리, 파일 시스템, 마운트를 추가하면서 어떤 이벤트를 보고해야 (또는 무시해야) 하는지 지정한다. 또는 그런 항목을 제거하거나 변경한다.

무시 마스크를 쓸 수 있는 경우로 파일 캐시가 있다. 파일 캐시에서 관심 있는 이벤트는 파일이 변경되고 닫히는 것이다. 따라서 캐싱 대상 디렉터리 내지 마운트 지점에 그 이벤트들을 받겠다고 표시한다. 파일이 변경됐음을 알리는 첫 번째 이벤트를 수신하면 대응하는 캐시 항목을 무효화하게 된다. 그런데 그 파일이 닫힐 때까지 이후의 변경 이벤트는 중요치 않다. 따라서 무시 마스크에 변경 이벤트를 추가할 수 있다. 닫힘 이벤트를 수신하면 무시 마스크에서 변경 이벤트를 제거하고서 파일 캐시 항목을 갱신할 수 있다.

fanotify 알림 그룹 안의 항목들은 아이노드를 통해 파일과 디렉터리를 가리키고 마운트 ID를 통해 마운트를 가리킨다. 파일이나 디렉터리가 동일 마운트 내에서 이름이 바뀌거나 이동하는 경우에는 대응 항목이 유지된다. 파일이나 디렉터리가 삭제되거나 다른 마운트로 이동하는 경우, 또는 파일 시스템이나 마운트가 언마운트 되는 경우에는 대응 항목이 삭제된다.

### 이벤트 큐

알림 그룹으로 감시하는 파일 시스템 객체들에서 이벤트가 생기면 fanotify 시스템에서 이벤트를 만들어서 큐에 모아 둔다. 그러면 <tt>[[fanotify_init(2)]]</tt>이 반환한 fanotify 파일 디스크립터에서 (`read(2)` 등으로) 그 이벤트들을 읽을 수 있다.

생성되는 이벤트에는 두 가지 종류가 있는데 *알림* 이벤트와 *허가* 이벤트이다. 알림 이벤트는 정보를 줄 뿐이므로 범용 이벤트로 전달된 파일 디스크립터를 닫아 줘야 한다는 것 외에는 수신 측 응용에서 어떤 행동도 취할 필요가 없다. 이벤트마다 파일 디스크립터를 닫는 건 `FAN_REPORT_FID`(아래 참고)를 쓰지 않고 fanotify를 초기화 한 응용에만 해당된다. 허가 이벤트는 어떤 파일 접근을 승인할지 여부를 수신 측 응용에서 판단해 달라는 요청이다. 이 이벤트를 수신한 쪽에선 접근을 승인할지 여부를 결정하는 응답을 줘야 한다.

이벤트를 읽어 들이면 fanotify 그룹의 이벤트 큐에서 그 이벤트가 제거된다. 읽기가 이뤄진 허가 이벤트는 fanotify 파일 디스크립터로 쓰기를 해서 허가 판단을 내리거나 fanotify 파일 디스크립터가 닫힐 때까지 fanotify 그룹의 내부 목록에 유지된다.

### fanotify 이벤트 읽기

<tt>[[fanotify_init(2)]]</tt>이 반환한 파일 디스크립터에 `read(2)`를 호출하면 (<tt>[[fanotify_init(2)]]</tt> 호출에서 `FAN_NONBLOCK` 플래그를 지정하지 않았다면) 파일 이벤트가 일어나거나 시그널에 의해 호출이 중단(<tt>[[signal(7)]]</tt> 참고)될 때까지 블록 한다.

<tt>[[fanotify_init(2)]]</tt>에 `FAN_REPORT_FID` 플래그를 사용했는지 여부가 이벤트마다 이벤트 리스너로 반환되는 자료 구조에 영향을 준다. `read(2)` 성공 후에 읽기 버퍼에는 다음 구조체가 한 개 이상 담긴다.

```c
struct fanotify_event_metadata {
    __u32 event_len;
    __u8 vers;
    __u8 reserved;
    __u16 metadata_len;
    __aligned_u64 mask;
    __s32 fd;
    __s32 pid;
};
```

<tt>[[fanotify_init(2)]]</tt>의 플래그 중 하나로 `FAN_REPORT_FID`를 준 경우에는 읽기 버퍼에서 각 범용 `fanotify_event_metadata` 구조체 다음에 아래 설명하는 구조체가 따라온다.

```c
struct fanotify_event_info_fid {
    struct fanotify_event_info_header hdr;
    __kernel_fsid_t fsid;
    unsigned char file_handle[0];
};
```

성능을 고려하면 큰 버퍼(가령 4096바이트)를 써서 `read(2)` 한 번에 여러 이벤트를 가져올 수 있게 하는 걸 권장한다.

`read(2)`의 반환 값은 버퍼로 들어간 바이트 수이며 오류 시에는 -1이다. (하지만 BUGS 참고.)

`fanotify_event_metadata` 구조체의 필드들은 다음과 같다.

<dl>
<dt><code>event_len</code></dt>
<dd>이 이벤트에 대한 데이터 길이이자 버퍼 내 다음 이벤트로의 오프셋이다. <code>FAN_REPORT_FID</code>를 안 쓰면 <code>event_len</code>의 값은 항상 <code>FAN_EVENT_METADATA_LEN</code>이다. <code>FAN_REPORT_FID</code>를 쓰면 <code>event_len</code>에 가변 길이 파일 식별자 크기까지 더해진다.</dd>

<dt><code>vers</code></dt>
<dd>이 필드는 구조체의 버전 번호를 담는다. 그 번호를 <code>FANOTIFY_METADATA_VERSION</code>과 비교해서 런타임에 반환된 구조체가 컴파일 타임에 정의돼 있던 구조체와 일치하는지 검증해야 한다. 일치하지 않는 경우 응용에서 fanotify 파일 디스크립터 사용 시도를 포기하는 게 좋다.</dd>

<dt><code>reserved</code></dt>
<dd>이 필드는 사용하지 않는다.</dd>

<dt><code>metadata_len</code></dt>
<dd>구조체의 길이이다. 이 필드는 이벤트 유형에 따른 선택적 헤더를 구현할 수 있도록 도입된 것이다. 현재 구현에서는 그런 선택적 헤더가 없다.</dd>

<dt><code>mask</code></dt>
<dd>이벤트를 기술하는 비트 마스크이다. (아래 참고.)</dd>

<dt><code>fd</code></dt>
<dd>

접근이 이뤄지고 있는 객체에 대한 열린 파일 디스크립터이다. 큐 넘침이 일어난 경우에는 <code>FAN_NOFD</code>이다. 응용에서 <code>FAN_REPORT_FID</code>를 써서 fanotify 파일 디스크립터를 초기화 했다면 수신하는 이벤트마다 이 값이 <code>FAN_NOFD</code>로 설정돼 있게 된다. 이 파일 디스크립터를 사용해 감시하는 파일 내지 디렉터리의 내용에 접근할 수 있다. 읽는 쪽 응용에서 파일 디스크립터를 닫을 책임이 있다.

<tt>[[fanotify_init(2)]]</tt> 호출 시에 (<code>event_f_flags</code> 인자를 통해) 이 파일 디스크립터에 대응하는 열린 파일 기술 항목에 설정할 다양한 파일 상태 플래그들을 지정할 수 있다. 더불어 그 열린 파일 기술 항목에는 (커널 내부용인) <code>FMODE_NONOTIFY</code> 파일 상태 플래그가 설정된다. 이 플래그는 fanotify 이벤트 생성을 막는다. 그래서 fanotify 이벤트 수신자가 이 파일 디스크립터를 이용해 알림 대상 파일 내지 디렉터리에 접근할 때 이벤트가 추가로 생기지 않게 된다.
</dd>

<dt><code>pid</code></dt>
<dd>

<tt>[[fanotify_init(2)]]</tt>에서 <code>FAN_REPORT_FID</code> 플래그를 설정했다면 이벤트를 유발한 스레드의 TID다. 아니라면 이벤트를 유발한 프로세스의 PID다.

fanotify 이벤트 청취 프로그램에서 이 PID를 <tt>[[getpid(2)]]</tt>가 반환하는 PID와 비교해서 이벤트를 유발한 것이 자기 자신인지 아니면 다른 프로세스의 파일 접근 때문인지 판단할 수 있다.
</dd>
</dl>

`mask`의 비트 마스크는 한 파일 시스템 객체에 어떤 이벤트들이 일어났는지를 나타낸다. 감시 중인 파일 시스템 객체에 여러 이벤트가 일어나면 이 마스크에 여러 비트가 설정될 수 있다. 특히 같은 파일 시스템 객체에 대한 동일 프로세스에서 유래한 연속된 이벤트들이 한 이벤트로 합쳐질 수도 있다. 단, 절대로 허가 이벤트 두 개가 질의 항목 하나로 합쳐지지는 않는다.

`mask`에 등장할 수 있는 비트들은 다음과 같다.

<dl>
<dt><code>FAN_ACCESS</code></dt>
<dd>파일이나 디렉터리에 (BUGS 참고) 접근(읽기)이 이뤄졌다.</dd>

<dt><code>FAN_OPEN</code></dt>
<dd>파일이나 디렉터리가 열렸다.</dd>

<dt><code>FAN_OPEN_EXEC</code></dt>
<dd>파일이 실행하려는 의도로 열렸다. 자세한 내용은 <tt>[[fanotify_mark(2)]]</tt>의 NOTES 참고.</dd>

<dt><code>FAN_ATTRIB</code></dt>
<dd>파일이나 디렉터리의 메타데이터가 변경됐다.</dd>

<dt><code>FAN_CREATE</code></dt>
<dd>감시하는 부모 내에서 파일이나 디렉터리가 생성됐다.</dd>

<dt><code>FAN_DELETE</code></dt>
<dd>감시하는 부모 내에서 파일이나 디렉터리가 삭제됐다.</dd>

<dt><code>FAN_DELETE_SELF</code></dt>
<dd>감시하는 파일이나 디렉터리 자체가 삭제됐다.</dd>

<dt><code>FAN_MOVED_FROM</code></dt>
<dd>감시하는 부모 디렉터리에 있던 파일이나 디렉터리가 이동됐다.</dd>

<dt><code>FAN_MOVED_TO</code></dt>
<dd>감시하는 부모 디렉터리로 파일이나 디렉터리가 이동됐다.</dd>

<dt><code>FAN_MOVE_SELF</code></dt>
<dd>감시하는 파일이나 디렉터리가 이동됐다.</dd>

<dt><code>FAN_MODIFY</code></dt>
<dd>파일이 변경됐다.</dd>

<dt><code>FAN_CLOSE_WRITE</code></dt>
<dd>쓰기용(<code>O_WRONLY</code>나 <code>O_RDWR</code>)으로 열린 파일이 닫혔다.</dd>

<dt><code>FAN_CLOSE_NOWRITE</code></dt>
<dd>읽기 전용(<code>O_RDONLY</code>)으로 열린 파일이나 디렉터리가 닫혔다.</dd>

<dt><code>FAN_Q_OVERFLOW</code></dt>
<dd>이벤트 큐가 제한치인 16384개 항목을 초과했다. <tt>[[fanotify_init(2)]]</tt>을 호출할 때 <code>FAN_UNLIMITED_QUEUE</code> 플래그를 지정하면 그 제한을 무시할 수 있다.</dd>

<dt><code>FAN_ACCESS_PERM</code></dt>
<dd>어느 응용에서 가령 <code>read(2)</code>나 <tt>[[readdir(2)]]</tt>을 이용해 파일이나 디렉터리를 읽고 싶어 한다. 그 파일 시스템 객체 접근을 인가해야 할지 결정하는 응답을 (아래 설명하는 대로) 줘야 한다.</dd>

<dt><code>FAN_OPEN_PERM</code></dt>
<dd>어느 응용에서 파일이나 디렉터리를 열고 싶어 한다. 그 파일 시스템 객체 열기를 인가해야 할지 결정하는 응답을 줘야 한다.</dd>

<dt><code>FAN_OPEN_EXEC_PERM</code></dt>
<dd>어느 응용에서 실행을 위해 파일을 열고 싶어 한다. 그 파일 시스템 객체의 실행 목적 열기를 인가해야 할지 결정하는 응답을 줘야 한다. 자세한 내용은 <tt>[[fanotify_mark(2)]]</tt>의 NOTES 참고.</dd>
</dl>

닫기 이벤트 확인에 다음 비트 마스크를 사용할 수도 있다.

<dl>
<dt><code>FAN_CLOSE</code></dt>
<dd>

파일이 닫혔다. 다음과 같다.

```c
FAN_CLOSE_WRITE | FAN_CLOSE_NOWRITE
```
</dd>
</dl>

이동 이벤트 확인에 다음 비트 마스크를 사용할 수도 있다.

<dl>
<dt><code>FAN_MOVE</code></dt>
<dd>

파일이나 디렉터리가 이동됐다. 다음과 같다.

```c
FAN_MOVED_FROM | FAN_MOVED_TO
```
</dd>
</dl>

`fanotify_event_info_fid` 구조체의 필드들은 다음과 같다.

<dl>
<dt><code>hdr</code></dt>
<dd><code>fanotify_event_info_header</code> 타입 구조체다. 이벤트에 덧붙은 추가 정보를 기술하는 정보를 담은 범용 헤더다. 예를 들어 <code>FAN_REPORT_FID</code>를 써서 fanotify 파일 디스크립터를 생성했을 때 이 헤더의 <code>info_type</code> 필드가 <code>FAN_EVENT_INFO_TYPE_FID</code>로 설정된다. 이벤트 리스너에서는 이 필드를 이용해 이벤트에 대해 수신한 추가 정보가 올바른 종류인지 확인할 수 있다. 그리고 <code>fanotify_event_info_header</code>에는 <code>len</code> 필드가 있다. 현재 구현에서 <code>len</code>의 값은 항상 <code>(event_len - FAN_EVENT_METADATA_LEN)</code>이다.</dd>

<dt><code>fsid</code></dt>
<dd>이벤트 연관 객체를 담은 파일 시스템의 고유 식별자다. <code>__kernel_fsid_t</code> 타입의 구조체이고 <tt>[[statfs(2)]]</tt> 호출 시의 <code>f_fsid</code>와 같은 값을 담고 있다.</dd>

<dt><code>file_handle</code></dt>
<dd><code>file_handle</code> 타입의 가변 길이 구조체다. 파일 시스템 상의 특정 객체에 대응하는 불투명한 핸들이며 <tt>[[name_to_handle_at(2)]]</tt>이 반환하는 것과 같은 것이다. 이를 이용해 파일 시스템 상의 파일을 유일하게 식별할 수 있으며 <tt>[[open_by_handle_at(2)]]</tt>에 인자로 줄 수 있다. 참고로 <code>FAN_CREATE</code>, <code>FAN_DELETE</code>, <code>FAN_MOVE</code> 같은 디렉터리 항목 이벤트에서 <code>file_handle</code>은 생성/삭제/이동된 자식 객체가 아니라 변경된 디렉터리를 나타낸다. 자식 객체를 감시 중이라면 <code>FAN_ATTRIB</code>, <code>FAN_DELETE_SELF</code>, <code>FAN_MOVE_SELF</code> 이벤트가 자식 객체에 대한 <code>file_handle</code> 정보를 전달해 준다.</dd>
</dl>

fanotify 파일 디스크립터에 대해 `read(2)`가 반환한 fanotify 이벤트 메타데이터들이 담긴 버퍼를 순회하기 위한 다음 매크로들이 있다.

<dl>
<dt><code>FAN_EVENT_OK(meta, len)</code></dt>
<dd>이 매크로는 버퍼의 남은 길이 <code>len</code>을 메타데이터 구조체의 길이 및 <code>meta</code>가 가리키는 메타데이터 구조체의 <code>event_len</code> 필드와 비교해 본다.</dd>

<dt><code>FAN_EVENT_NEXT(meta, len)</code></dt>
<dd>이 매크로는 <code>meta</code>가 가리키는 메타데이터 구조체의 <code>event_len</code> 필드에 있는 길이 값을 이용해 <code>meta</code> 다음에 오는 메타데이터 구조체의 주소를 계산한다. <code>len</code>은 현재 버퍼에 남아 있는 메타데이터 바이트 수이다. <code>meta</code> 다음에 있는 메타데이터 구조체에 대한 포인터를 반환하며 건너뛴 메타데이터 구조체의 바이트 수만큼 <code>len</code>을 줄인다. (즉 <code>len</code>에서 <code>meta-&gt;event_len</code>을 뺀다.)</dd>
</dl>

또한 다음 매크로가 있다.

<dl>
<dt><code>FAN_EVENT_METADATA_LEN</code></dt>
<dd>이 매크로는 <code>fanotify_event_metadata</code> 구조체의 (바이트 단위) 크기를 반환한다. 이벤트 메타데이터의 최소 크기(그리고 현재는 유일한 크기)이다.</dd>
</dl>

### fanotify 파일 디스크립터에서 이벤트 감시하기

fanotify 이벤트가 일어날 때 fanotify 파일 디스크립터를 <tt>[[epoll(7)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[select(2)]]</tt>에 주면 읽기 가능으로 표시된다.

### 권한 이벤트 처리하기

허가 이벤트의 경우 응용에서 fanotify 파일 디스크립터로 다음 형태의 구조체를 `write(2)` 해 주어야 한다.

```c
struct fanotify_response {
    __s32 fd;
    __u32 response;
};
```

이 구조체의 필드들은 다음과 같다.

<dl>
<dt><code>fd</code></dt>
<dd><code>fanotify_event_metadata</code> 구조체에서 온 파일 디스크립터이다.</dd>

<dt><code>response</code></dt>
<dd>이 필드는 동작을 인가할지 여부를 나타낸다. 파일 동작을 허용하는 <code>FAN_ALLOW</code>거나 파일 동작을 거부하는 <code>FAN_DENY</code>여야 한다.</dd>
</dl>

접근을 거부하면 요청 측 응용 호출이 `EPERM` 오류를 받게 된다.

### fanotify 파일 디스크립터 닫기

fanotify 알림 그룹을 가리키는 모든 파일 디스크립터가 닫힐 때 fanotify 그룹이 해제되고 그 자원을 커널에서 재사용할 수 있게 된다. <tt>[[close(2)]]</tt> 시에 미처리 허가 이벤트는 허용으로 처리된다.

### `/proc/[pid]/fdinfo`

`/proc/[pid]/fdinfo/[fd]` 파일이 프로세스 `pid`의 파일 디스크립터 `fd`에 대한 fanotify 표시 정보를 담고 있다. 자세한 내용은 <tt>[[proc(5)]]</tt> 참고.

## ERRORS

fanotify 파일 디스크립터에서 읽기를 할 때 일반적인 `read(2)` 오류에 더해서 다음 오류가 발생할 수 있다.

<dl>
<dt><code>EINVAL</code></dt>
<dd>버퍼가 이벤트를 담기에 너무 작다.</dd>
<dt><code>EMFILE</code></dt>
<dd>열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다. <tt>[[getrlimit(2)]]</tt>의 <code>RLIMIT_NOFILE</code> 설명 참고.</dd>
<dt><code>ENFILE</code></dt>
<dd>열린 파일 총개수에 대한 시스템 전역 제한에 도달했다. <tt>[[proc(5)]]</tt>의 <code>/proc/sys/fs/file-max</code> 참고.</dd>
<dt><code>ETXTBSY</code></dt>
<dd><tt>[[fanotify_init(2)]]</tt> 호출 시 <code>event_f_flags</code> 인자에 <code>O_RDWR</code>나 <code>O_WRONLY</code>를 지정했으며 현재 실행 중인 감시 대상 파일에 이벤트가 발생한 경우 <code>read(2)</code>가 이 오류를 반환한다.</dd>
</dl>

fanotify 파일 디스크립터에 쓰기를 할 때 일반적인 `write(2)` 오류에 더해서 다음 오류가 발생할 수 있다.

<dl>
<dt><code>EINVAL</code></dt>
<dd>커널 구성에서 fanotify 접근 허가가 켜져 있지 않거나 응답 구조체의 <code>response</code> 값이 유효하지 않다.</dd>
<dt><code>ENOENT</code></dt>
<dd>응답 구조체의 파일 디스크립터 <code>fd</code>가 유효하지 않다. 허가 이벤트에 대한 응답을 이미 써 준 경우에 이 오류가 발생할 수 있다.</dd>
</dl>

## VERSIONS

리눅스 커널 버전 2.6.36에서 fanotify API가 도입되었고 버전 2.6.37에서 활성화되었다. 버전 3.8에서 fdinfo 지원이 추가되었다.

## CONFORMING TO

fanotify API는 리눅스 전용이다.

## NOTES

fanotify API는 `CONFIG_FANOTIFY` 구성 옵션을 켜서 커널을 빌드 한 경우에만 사용 가능하다. 더불어 fanotify 권한 처리는 `CONFIG_FANOTIFY_ACCESS_PERMISSIONS` 구성 옵션이 켜진 경우에만 사용 가능하다.

### 한계과 주의점

fanotify는 사용자 공간 프로그램이 파일 시스템 API를 통해 유발한 이벤트만을 보고한다. 따라서 네트워크 파일 시스템 상에서 일어나는 원격 이벤트는 잡지 못한다.

fanotify API는 <tt>[[mmap(2)]]</tt>, <tt>[[msync(2)]]</tt>, <tt>[[munmap(2)]]</tt> 때문에 일어날 수 있는 파일 접근 및 변경에 대해 알려 주지 않는다.

디렉터리에 대한 이벤트는 디렉터리 자체를 열고 읽고 닫는 경우에만 생긴다. 표시한 디렉터리의 자식을 추가하거나 제거하거나 변경해도 감시 대상 디렉터리 자체에 대한 이벤트가 생기지 않는다.

fanotify에서의 디렉터리 감시는 재귀적이지 않다. 디렉터리 아래 서브디렉터리를 감시하려면 감시 항목을 추가로 만들어야 한다. (하지만 fanotify API에서는 표시한 디렉터리 아래에 서브디렉터리가 생긴 걸 탐지할 방법을 제공하지 않으므로 재귀적인 감시가 어렵다.) 마운트를 감시하면 디렉터리 트리 전체를 감시할 수 있다. 파일 시스템을 감시하면 파일 시스템의 모든 마운트 인스턴스에서 일어난 변경 사항을 감시할 수 있다.

이벤트 큐가 넘칠 수 있다. 이 경우 이벤트가 유실된다.

## BUGS

리눅스 3.19 전에서 <tt>[[fallocate(2)]]</tt>가 fanotify 이벤트를 생성하지 않았다. 리눅스 3.19부터 <tt>[[fallocate(2)]]</tt> 호출이 `FAN_MODIFY` 이벤트를 생성한다.

리눅스 3.17 현재 다음 버그들이 존재한다.

* 리눅스에서 어떤 파일 시스템 객체가 여러 경로로 접근 가능할 수 있다. 예를 들어 파일 시스템 일부를 <tt>[[mount(8)]]</tt>의 `--bind` 옵션으로 다시 마운트 할 수도 있다. 마운트에 표시를 한 리스너는 해당 마운트를 쓰는 파일 시스템 객체에 유발된 이벤트에 대해서만 알림을 받게 된다. 다른 이벤트는 알림이 이뤄지지 않는다.

* 이벤트가 생길 때 파일에 대한 파일 디스크립터를 전달해 주기 전에 수신 측 프로세스의 사용자 ID가 파일에 읽기 내지 쓰기 권한이 있는지 확인하지 않는다. 비특권 사용자가 실행한 프로그램에 `CAP_SYS_ADMIN` 역능이 설정돼 있는 경우 보안 위험이 된다.

* `read(2)` 호출 내에서 fanotify 큐의 여러 이벤트를 꺼내 처리한 다음 오류가 발생한 경우에 반환 값은 오류 발생 전까지 사용자 공간 버퍼로 성공적으로 복사한 이벤트들의 총 길이가 된다. 즉 반환 값이 -1이 아니고 `errno`가 설정되지 않는다. 그래서 읽는 쪽 응용에서 오류를 탐지할 방법이 없다.

## EXAMPLE

아래의 두 예시 프로그램이 fanotify API 사용 방법을 보여 준다.

### 예시 프로그램: `fanotify_example.c`

첫 번째 프로그램은 이벤트 객체 정보가 파일 디스크립터 형태로 전달되도록 해서 fanotify를 쓰는 예시이다. 프로그램에서 명령행 인자로 받은 마운트 지점에 표시를 하고서 `FAN_OPEN_PERM` 및 `FAN_CLOSE_WRITE` 이벤트를 기다린다. 권한 이벤트가 발생하면 `FAN_ALLOW` 응답을 준다.

다음 셸 세션은 프로그램 실행 예를 보여 준다. 이 세션과 더불어 파일 `/home/user/temp/notes` 편집이 이뤄졌다. 파일을 열기 전에는 `FAN_OPEN_PERM` 이벤트가 발생했다. 파일을 닫은 후에는 `FAN_CLOSE_WRITE` 이벤트가 발생했다. 사용자가 엔터 키를 누르면 프로그램 실행이 끝난다.

```
# ./fanotify_example /home
Press enter key to terminate.
Listening for events.
FAN_OPEN_PERM: File /home/user/temp/notes
FAN_CLOSE_WRITE: File /home/user/temp/notes

Listening for events stopped.
```

### 프로그램 소스: `fanotify_example.c`

```c
#define _GNU_SOURCE     /* O_LARGEFILE 정의를 얻기 위해 */
#include <errno.h>
#include <fcntl.h>
#include <limits.h>
#include <poll.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/fanotify.h>
#include <unistd.h>

/* 파일 디스크립터 'fd'에서 가용 fanotify 이벤트를 모두 읽어 들이기 */

static void
handle_events(int fd)
{
    const struct fanotify_event_metadata *metadata;
    struct fanotify_event_metadata buf[200];
    ssize_t len;
    char path[PATH_MAX];
    ssize_t path_len;
    char procfd_path[PATH_MAX];
    struct fanotify_response response;

    /* fanotify 파일 디스크립터에서 이벤트를 읽을 수 있는 동안 반복 */

    for (;;) {

        /* 이벤트 읽기 */

        len = read(fd, (void *) &buf, sizeof(buf));
        if (len == -1 && errno != EAGAIN) {
            perror("read");
            exit(EXIT_FAILURE);
        }

        /* 가용 데이터 끝에 도달했는지 확인 */

        if (len <= 0)
            break;

        /* 버퍼의 첫 번째 이벤트 가리키기 */

        metadata = buf;

        /* 버퍼 내 모든 이벤트 돌기 */

        while (FAN_EVENT_OK(metadata, len)) {

            /* 런타임 구조체와 컴파일 타임 구조체가 일치하는지 확인 */

            if (metadata->vers != FANOTIFY_METADATA_VERSION) {
                fprintf(stderr,
                        "Mismatch of fanotify metadata version.\n");
                exit(EXIT_FAILURE);
            }

            /* metadata->fd는 큐 넘침을 나타내는 FAN_NOFD거나
               파일 디스크립터(음수 아닌 정수)임. 여기서 큐 넘침은
               그냥 무시함. */

            if (metadata->fd >= 0) {

                /* 열기 권한 이벤트 처리 */

                if (metadata->mask & FAN_OPEN_PERM) {
                    printf("FAN_OPEN_PERM: ");

                    /* 파일 열기 허용 */

                    response.fd = metadata->fd;
                    response.response = FAN_ALLOW;
                    write(fd, &response,
                          sizeof(struct fanotify_response));
                }

                /* 쓰기 가능 파일 닫기 이벤트 처리 */

                if (metadata->mask & FAN_CLOSE_WRITE)
                    printf("FAN_CLOSE_WRITE: ");

                /* 접근 파일의 경로명 얻어서 찍기 */

                snprintf(procfd_path, sizeof(procfd_path),
                         "/proc/self/fd/%d", metadata->fd);
                path_len = readlink(procfd_path, path,
                                    sizeof(path) - 1);
                if (path_len == -1) {
                    perror("readlink");
                    exit(EXIT_FAILURE);
                }

                path[path_len] = '\0';
                printf("File %s\n", path);

                /* 이벤트의 파일 디스크립터 닫기 */

                close(metadata->fd);
            }

            /* 다음 이벤트로 */

            metadata = FAN_EVENT_NEXT(metadata, len);
        }
    }
}

int main(int argc, char *argv[])
{
    char buf;
    int fd, poll_num;
    nfds_t nfds;
    struct pollfd fds[2];

    /* 마운트 지점 주어졌는지 확인 */

    if (argc != 2) {
        fprintf(stderr, "Usage: %s MOUNT\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    printf("Press enter key to terminate.\n");

    /* fanotify API 접근을 위한 파일 디스크립터 만들기 */

    fd = fanotify_init(FAN_CLOEXEC | FAN_CLASS_CONTENT | FAN_NONBLOCK,
                       O_RDONLY | O_LARGEFILE);
    if (fd == -1) {
        perror("fanotify_init");
        exit(EXIT_FAILURE);
    }

    /* 마운트에 다음 표시:
       - 파일 열기 전 권한 이벤트
       - 쓰기 가능 파일 디스크립터 닫은 후
         알림 이벤트 */

    if (fanotify_mark(fd, FAN_MARK_ADD | FAN_MARK_MOUNT,
                      FAN_OPEN_PERM | FAN_CLOSE_WRITE, AT_FDCWD,
                      argv[1]) == -1) {
        perror("fanotify_mark");
        exit(EXIT_FAILURE);
    }

    /* 폴링 준비 */

    nfds = 2;

    /* 콘솔 입력 */

    fds[0].fd = STDIN_FILENO;
    fds[0].events = POLLIN;

    /* fanotify 입력 */

    fds[1].fd = fd;
    fds[1].events = POLLIN;

    /* 이벤트 입력을 기다리는 루프 */

    printf("Listening for events.\n");

    while (1) {
        poll_num = poll(fds, nfds, -1);
        if (poll_num == -1) {
            if (errno == EINTR)     /* 시그널에 의해 중단 */
                continue;           /* poll() 재시작 */

            perror("poll");         /* 예상 못한 오류 */
            exit(EXIT_FAILURE);
        }

        if (poll_num > 0) {
            if (fds[0].revents & POLLIN) {

                /* 콘솔 입력 있음. stdin 비우고 끝내기 */

                while (read(STDIN_FILENO, &buf, 1) > 0 && buf != '\0')
                    continue;
                break;
            }

            if (fds[1].revents & POLLIN) {

                /* inotify 이벤트 있음 */

                handle_events(fd);
            }
        }
    }

    printf("Listening for events stopped.\n");
    exit(EXIT_SUCCESS);
}
```

### 예시 프로그램: `fanotify_fid.c`

두 번째 프로그램은 `FAN_REPORT_FID`를 켜서 fanotify를 쓰는 예시이다. 프로그램에서 명령행 인자로 받은 파일 시스템 객체에 표시를 하고서 `FAN_CREATE` 이벤트가 발생할 때까지 기다린다. 이벤트 마스크를 보면 어떤 종류의 (파일 또는 디렉터리) 파일 시스템 객체가 생성됐는지 알 수 있다. 버퍼에서 모든 이벤트를 읽어서 차례로 처리한 다음 프로그램을 그대로 종료한다.

다음 두 번의 셸 세션은 감시 대상 객체에 다른 동작을 하면서 프로그램을 호출한 것이다.

첫 번째 세션에선 `/home/user`에 표시가 이뤄지는 걸 볼 수 있다. 그 다음에 정규 파일 `/home/user/testfile.txt` 생성이 이뤄진다. 그러면 파일의 부모인 감시 대상 디렉터리 객체에 대해 `FAN_CREATE` 이벤트가 생겨서 보고된다. 그러고서 버퍼에 잡힌 이벤트를 모두 처리하고 나면 프로그램 실행이 끝난다.

```
# ./fanotify_fid /home/user
Listening for events.
FAN_CREATE (file created): Directory /home/user has been modified.
All events processed successfully. Program exiting.

$ touch /home/user/testing              # 다른 터미널에서
```

두 번째 세션에선 `/home/user`에 표시가 이뤄지는 걸 볼 수 있다. 그 다음에 디렉터리 `/home/user/testdir` 생성이 이뤄진다. 그러면 프로그램에서 `FAN_CREATE` 및 `FAN_ONDIR` 이벤트가 만들어진다.

```
# ./fanotify_fid /home/user
Listening for events.
FAN_CREATE | FAN_ONDIR (subdirectory created):
        Directory /home/user has been modified.
All events processed successfully. Program exiting.

$ mkdir -p /home/user/testing          # 다른 터미널에서
```

### 프로그램 소스: `fanotify_fid.c`

```c
#define _GNU_SOURCE
#include <errno.h>
#include <fcntl.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/fanotify.h>
#include <unistd.h>

#define BUF_SIZE 256

int
main(int argc, char **argv)
{
    int fd, ret, event_fd;
    ssize_t len, path_len;
    char path[PATH_MAX];
    char procfd_path[PATH_MAX];
    char events_buf[BUF_SIZE];
    struct file_handle *file_handle;
    struct fanotify_event_metadata *metadata;
    struct fanotify_event_info_fid *fid;

    if (argc != 2) {
        fprintf(stderr, "Invalid number of command line arguments.\0);
        exit(EXIT_FAILURE);
    }

    /* FAN_REPORT_FID 플래그로 fanotify 파일 디스크립터를 만들어서
       프로그램이 fid 이벤트를 받게 한다. */

    fd = fanotify_init(FAN_CLASS_NOTIF | FAN_REPORT_FID, 0);
    if (fd == -1) {
        perror("fanotify_init");
        exit(EXIT_FAILURE);
    }

    /* argv[1]로 받은 파일 시스템 객체에 표시를 한다. */

    ret = fanotify_mark(fd, FAN_MARK_ADD | FAN_MARK_ONLYDIR,
                        FAN_CREATE | FAN_ONDIR,
                        AT_FDCWD, argv[1]);
    if (ret == -1) {
        perror("fanotify_mark");
        exit(EXIT_FAILURE);
    }

    printf("Listening for events.\0);

    /* 이벤트 큐에서 이벤트를 읽어서 버퍼로 */

    len = read(fd, (void *) &events_buf, sizeof(events_buf));
    if (len == -1 && errno != EAGAIN) {
        perror("read");
        exit(EXIT_FAILURE);
    }

    /* 버퍼의 모든 이벤트 처리 */

    for (metadata = (struct fanotify_event_metadata *) events_buf;
            FAN_EVENT_OK(metadata, len);
            metadata = FAN_EVENT_NEXT(metadata, len)) {
        fid = (struct fanotify_event_info_fid *) (metadata + 1);
        file_handle = (struct file_handle *) fid->handle;

        /* 이벤트 정보의 타입이 맞는지 확인 */

        if (fid->hdr.info_type != FAN_EVENT_INFO_TYPE_FID) {
            fprintf(stderr, "Received unexpected event info type.\0);
            exit(EXIT_FAILURE);
        }

        if (metadata->mask == FAN_CREATE)
            printf("FAN_CREATE (file created):");

        if (metadata->mask == FAN_CREATE | FAN_ONDIR)
            printf("FAN_CREATE | FAN_ONDIR (subdirectory created):");

        /* FAN_REPORT_FID가 켜져 있으면 metadata->fd가 FAN_NOFD로
           설정돼 있다. 이벤트 관련 파일 객체에 대한 파일 디스크립터를
           얻으려면 fanotify_event_info_fid 안에 있는 struct
           file_handle을 open_by_handle_at(2) 시스템 호출과 함께
           이용할 수 있다. 시스템 호출 전에 그 객체가 삭제되는 경우를
           대비해서 ESTALE 검사를 한다. */

        event_fd = open_by_handle_at(AT_FDCWD, file_handle, O_RDONLY);
        if (ret == -1) {
            if (errno == ESTALE) {
                printf("File handle is no longer valid. "
                        "File has been deleted\0);
                continue;
            } else {
                perror("open_by_handle_at");
                exit(EXIT_FAILURE);
         }
        }

        snprintf(procfd_path, sizeof(procfd_path), "/proc/self/fd/%d",
                event_fd);

        /* 변경된 dentry의 경로 가져와서 찍기 */

        path_len = readlink(procfd_path, path, sizeof(path) - 1);
        if (path_len == -1) {
            perror("readlink");
            exit(EXIT_FAILURE);
        }

        path[path_len] = '\ ';
        printf("\tDirectory '%s' has been modified.\0, path);

        /* 이 이벤트의 연관 파일 디스크립터 닫기 */

        close(event_fd);
    }

    printf("All events processed successfully. Program exiting.\0);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[fanotify_init(2)]]</tt>, <tt>[[fanotify_mark(2)]]</tt>, <tt>[[inotify(7)]]</tt>

----

2019-08-02
