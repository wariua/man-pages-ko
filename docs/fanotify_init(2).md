## NAME

fanotify_init - fanotify 그룹 만들어서 초기화 하기

## SYNOPSIS

```c
#include <fcntl.h>
#include <sys/fanotify.h>

int fanotify_init(unsigned int flags, unsigned int event_f_flags);
```

## DESCRIPTION

fanotify API에 대한 소개는 <tt>[[fanotify(7)]]</tt>를 보라.

`fanotify_init()`은 새 fanotify 그룹을 초기화 하고 그룹에 연계된 이벤트 큐에 대한 파일 디스크립터를 반환한다.

그 파일 디스크립터를 <tt>[[fanotify_mark(2)]]</tt> 호출에 사용해서 fanotify 이벤트가 발생할 파일, 디렉터리, 마운트, 파일 시스템을 지정한다. 그리고 파일 디스크립터에 읽기를 해서 그 이벤트를 읽는다. 어떤 이벤트는 정보를 주기만 하며 파일에 접근이 이뤄졌음을 나타낸다. 다른 이벤트는 다른 응용이 파일이나 디렉터리에 접근할 수 있는지 결정하는 데 쓸 수 있다. 파일 디스크립터에 쓰기를 해서 그 파일 시스템 객체에 대한 접근을 인가한다.

여러 프로그램이 fanotify 인터페이스로 같은 파일을 동시에 감시할 수도 있다.

현재 구현에서 사용자별 fanotify 그룹 수는 128개로 제한돼 있다. 이 제한을 무시할 수 없다.

`fanotify_init()` 호출을 위해선 `CAP_SYS_ADMIN` 역능이 필요하다. API 향후 버전에서는 이 제약이 완화될 수도 있다. 그래서 아래에서 보이는 것처럼 어떤 역능 검사들이 추가로 구현돼 있다.

`flags` 인자는 청취 응용의 알림 등급을 지정하는 비트 필드와 파일 디스크립터의 동작 방식을 지정하는 비트 필드들을 담고 있다.

허가 이벤트를 듣는 청취자가 여럿 존재하는 경우 알림 등급을 이용해 이벤트를 수신하는 순서를 정한다.

`flags`에 다음 알림 등급들 중 하나를 지정할 수 있다.

<dl>
<dt><code>FAN_CLASS_PRE_CONTENT</code></dt>
<dd>이 값은 파일에 접근이 이뤄졌음을 알리는 이벤트와 파일에 접근할 수 있냐는 허가 결정 이벤트를 수신하도록 한다. 파일에 최종 내용물이 담기기 전에 접근해야 하는 청취자를 위한 것이다. 예를 들면 계층 저장소 관리자에서 이 알림 등급을 사용할 수 있을 것이다.</dd>

<dt><code>FAN_CLASS_CONTENT</code></dt>
<dd>이 값은 파일에 접근이 이뤄졌음을 알리는 이벤트와 파일에 접근할 수 있냐는 허가 결정 이벤트를 수신하도록 한다. 파일에 최종 내용물이 이미 담겨 있을 때 접근해야 하는 청취자를 위한 것이다. 예를 들어 멀웨어 탐지 프로그램에서 이 알림 등급을 사용할 수 있을 것이다.</dd>

<dt><code>FAN_REPORT_FID</code> (리눅스 5.1부터)</dt>
<dd>이 값은 이벤트와 연관된 하위 파일 시스템 객체에 대한 추가 정보를 담은 이벤트를 수신하도록 한다. 또 다른 구조체가 객체에 대한 정보를 담고서 범용 이벤트 메타데이터 구조체와 함께 포함돼 있다. 그리고 이벤트와 연관된 객체를 나타내는 데 쓰는 파일 디스크립터는 파일 핸들로 교체된다. 이는 파일 디스크립터 대신 파일 핸들을 쓰면 객체를 더 쉽게 식별할 수 있는 응용들을 위한 것이다. 또한 <code>FAN_CREATE</code>, <code>FAN_ATTRIB</code>, <code>FAN_MOVE</code>, <code>FAN_DELETE</code> 등과 같은 디렉터리 항목 이벤트에 관심 있는 응용들에서도 이용할 수 있다. 참고로 마운트 지점을 감시할 때 디렉터리 변경 이벤트 사용을 지원하지 않는다. 또 <code>FAN_CLASS_CONTENT</code>나 <code>FAN_CLASS_PRE_CONTENT</code>를 이 플래그와 함께 쓰지 못하며 그렇게 하면 <code>EINVAL</code> 오류가 발생한다. 자세한 내용은 <tt>[[fanotify(7)]]</tt> 참고.</dd>

<dt><code>FAN_CLASS_NOTIF</code></dt>
<dd>기본값이다. 따로 지정하지 않아도 된다. 이 값은 파일에 접근이 이뤄졌음을 알리는 이벤트만 수신하도록 한다. 파일 접근 전의 허가 결정은 불가능하다.</dd>
</dl>

알림 등급이 다른 청취자들은 `FAN_CLASS_PRE_CONTENT`, `FAN_CLASS_CONTENT`, `FAN_CLASS_NOTIF` 순서로 이벤트를 받게 된다. 알림 등급이 같은 청취자들에 대한 알림 순서는 규정돼 있지 않다.

`flags`에 추가로 다음 비트들을 설정할 수 있다.

<dl>
<dt><code>FAN_CLOEXEC</code></dt>
<dd>새 파일 디스크립터에 'exec에서 닫기' 플래그(<code>FD_CLOEXEC</code>)를 설정한다. <tt>[[open(2)]]</tt>의 <code>O_CLOEXEC</code> 플래그 설명을 보라.</dd>

<dt><code>FAN_NONBLOCK</code></dt>
<dd>파일 디스크립터에 논블로킹 플래그(<code>O_NONBLOCK</code>)를 켠다. 파일 디스크립터에서 읽기가 블록 하지 않게 된다. 읽을 수 있는 데이터가 없으면 대신 <code>read(2)</code>가 <code>EAGAIN</code> 오류로 실패한다.</dd>

<dt><code>FAN_UNLIMITED_QUEUE</code></dt>
<dd>이벤트 큐에서 16384개 이벤트 제한을 없앤다. 이 플래그를 사용하려면 <code>CAP_SYS_ADMIN</code> 역능이 필요하다.</dd>

<dt><code>FAN_UNLIMITED_MARKS</code></dt>
<dd>8192개 표시 제한을 없앤다. 이 플래그를 사용하려면 <code>CAP_SYS_ADMIN</code> 역능이 필요하다.</dd>

<dt><code>FAN_REPORT_TID</code> (리눅스 4.20부터)</dt>
<dd><code>read(2)</code>가 내놓는 <code>struct fanotify_event_metadata</code>(<tt>[[fanotify(7)]]</tt> 참고)의 <code>pid</code> 필드로 프로세스 ID(PID) 대신 스레드 ID(TID)를 알려 준다.</dd>
</dl>

`event_f_flags` 인자는 fanotify 이벤트에 대해 생성되는 열린 파일 기술 항목에 설정할 파일 상태 플래그를 지정한다. 이 플래그들에 대한 자세한 내용은 <tt>[[open(2)]]</tt>의 `flags` 값 설명을 보라. `event_f_flags`에는 접근 모드를 위한 비트 필드가 포함된다. 그 필드에 다음 값이 들어갈 수 있다.

<dl>
<dt><code>O_RDONLY</code></dt>
<dd>읽기 접근만 허용한다.</dd>

<dt><code>O_WRONLY</code></dt>
<dd>쓰기 접근만 허용한다.</dd>

<dt><code>O_RDWR</code></dt>
<dd>읽기 및 쓰기 접근을 허용한다.</dd>
</dl>

`event_f_flags`에 다른 비트들을 설정할 수 있다. 특히 유용한 값들은 다음과 같다.

<dl>
<dt><code>O_LARGEFILE</code></dt>
<dd>2GB 초과 파일 지원을 켠다. 이 플래그를 설정하지 않으면 32비트 시스템에서 fanotify 그룹으로 감시하는 큰 파일을 열려고 할 때 <code>EOVERFLOW</code> 오류가 발생하게 된다.</dd>

<dt><code>O_CLOEXEC</code> (리눅스 3.18부터)</dt>
<dd>파일 디스크립터에 'exec에서 닫기' 플래그를 켠다. 이게 유용할 수 있는 이유에 대해선 <tt>[[open(2)]]</tt>의 <code>O_CLOEXEC</code> 플래그 설명을 보라.</dd>
</dl>

`O_APPEND`, `O_DSYNC`, `O_NOATIME`, `O_NONBLOCK`, `O_SYNC`도 사용 가능하다. `event_f_flags`에 그 외 다른 플래그를 지정하면 `EINVAL` 오류가 생긴다. (하지만 BUGS 참고.)

## RETURN VALUE

성공 시 `fanotify_init()`은 새 파일 디스크립터를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>flags</code>나 <code>event_f_flags</code>에 유효하지 않은 값을 줬다. <code>FAN_ALL_INIT_FLAGS</code>(리눅스 커널 버전 4.20부터 폐기 예정 상태)가 <code>flags</code>에 가능한 모든 비트들이다.</dd>
<dt><code>EMFILE</code></dt>
<dd>이 사용자의 알림 그룹 수가 128개를 초과한다.</dd>
<dt><code>EMFILE</code></dt>
<dd>열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>알림 그룹을 위한 메모리 할당에 실패했다.</dd>
<dt><code>ENOSYS</code></dt>
<dd>커널에서 <code>fanotify_init()</code>을 구현하고 있지 않다. 커널을 <code>CONFIG_FANOTIFY</code>로 구성한 경우에만 fanotify API를 쓸 수 있다.</dd>
<dt><code>EPERM</code></dt>
<dd>호출자에게 <code>CAP_SYS_ADMIN</code> 역능이 없어서 동작이 허가되지 않는다.</dd>
</dl>

## VERSIONS

리눅스 커널 버전 2.6.36에서 `fanotify_init()`이 도입되었고 버전 2.6.37에서 활성화되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## BUGS

리눅스 커널 버전 3.18 전에 다음 버그가 있었다.

* `event_f_flags`에 `O_CLOEXEC`를 주면 무시한다.

리눅스 커널 버전 3.14 전에 다음 버그가 있었다.

* `event_f_flags`에 유효하지 않은 플래그가 있는지 검사하지 않는다. `FMODE_EXEC`처럼 내부 용도로만 쓰는 플래그를 설정할 수 있으며, 그러면 fanotify 파일 디스크립터 읽기가 반환하는 파일 디스크립터에 설정이 된다.

## SEE ALSO

<tt>[[fanotify_mark(2)]]</tt>, <tt>[[fanotify(7)]]</tt>

----

2019-08-02
