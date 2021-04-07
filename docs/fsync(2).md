## NAME

fsync, fdatasync - 파일의 코어 내 상태를 저장 장치와 동기화하기

## SYNOPSIS

```c
#include <unistd.h>

int fsync(int fd);

int fdatasync(int fd);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>fsync()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.16 및 이후:</dt>
 <dd>어떤 기능 확인 매크로도 정의돼 있을 필요 없음</dd>
 <dt>glibc 2.15까지:</dt>
 <dd>
 </dd>
<code>_BSD_SOURCE || _XOPEN_SOURCE</code><br>
<code>    || /* glibc 2.8부터: */ _POSIX_C_SOURCE >= 200112L</code>
 </dl>
</dd>
<dt><code>fdatasync()</code>:</dt>
<dd><code>_POSIX_C_SOURCE >= 199309L || _XOPEN_SOURCE >= 500</code></dd>
</dl>

## DESCRIPTION

`fsync()`는 파일 디스크립터 `fd`가 가리키는 파일의 변경된 코어 내 데이터 (즉 변경된 버퍼 캐시 페이지) 모두를 디스크 장치로 (또는 다른 영속 저장 장치로) 이동("플러시")한다. 그래서 시스템이 죽거나 재부팅 되는 경우에도 바뀐 정보를 모두 얻을 수 있도록 한다. 디스크 캐시가 존재하는 경우 그 캐시를 통과해 기록하거나 플러시 하는 것까지 포함한다. 이동이 완료됐다고 장치가 알릴 때까지 호출이 블록 한다.

`fsync()`는 파일 데이터를 플러시 할 뿐 아니라 파일에 연계된 메타데이터 정보(<tt>[[inode(7)]]</tt> 참고)도 플러시 한다.

`fsync()`를 호출해도 그 파일을 담은 디렉터리 내 항목까지 디스크에 도달했다고 보장되지는 않는다. 디렉터리에 대한 파일 디스크립터에도 따로 `fsync()`가 필요하다.

`fdatasync()`는 `fsync()`와 비슷하되 후속 데이터 조회를 올바로 처리하는 데 필요한 게 아니라면 변경된 메타데이터를 플러시 하지 않는다. 예를 들어 `st_atime`이나 `st_mtime`(각각 최근 접근 시간과 최근 수정 시간. <tt>[[inode(7)]]</tt> 참고) 변경은 후속 데이터 읽기를 올바로 처리하는 데 필요하지 않으므로 플러시 할 필요가 없다. 반면 (가령 <tt>[[ftruncate(2)]]</tt>를 통한) 파일 크기(`st_size`) 변경은 메타데이터 플러시가 필요할 것이다.

`fdatasync()`의 목적은 모든 메타데이터가 디스크에 동기화돼야 하는 건 아닌 응용에서 디스크 활동을 줄이는 것이다.

## RETURN VALUE

성공 시 이 시스템 호출들은 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EBADF</code></dt>
<dd><code>fd</code>가 유효한 열린 파일 디스크립터가 아니다.</dd>
<dt><code>EIO</code></dt>
<dd>동기화 중에 오류가 발생했다. 이 오류는 같은 파일에 대한 어떤 다른 파일 디스크립터에 데이터를 기록한 것과 관련돼 있을 수 있다. 리눅스 4.13부터는 write-back에서의 오류를 그 오류를 촉발한 데이터를 기록했을 수도 있는 모든 파일 디스크립터들로 보고한다. 어떤 파일 시스템들(가령 NFS)은 어느 데이터가 어느 파일 디스크립터를 통해 왔는지 밀접하게 추적하므로 더 정확하게 보고한다. 하지만 다른 파일 시스템들(가령 대부분의 로컬 파일 시스템)은 오류가 기록됐을 때 파일에 대해 열려 있던 모든 파일 디스크립터로 오류를 보고한다.</dd>
<dt><code>ENOSPC</code></dt>
<dd>동기화 중에 디스크 공간이 고갈되었다.</dd>
<dt><code>EROFS</code>, <code>EINVAL</code></dt>
<dd><code>fd</code>가 동기화를 지원하지 않는 특수 파일(가령 파이프, FIFO, 소켓)에 결속돼 있다.</dd>
<dt><code>ENOSPC</code>, <code>EDQUOT</code></dt>
<dd><code>write(2)</code> 시스템 호출 시점에 공간을 할당하지 않는 NFS나 다른 파일 시스템 상의 파일에 <code>fd</code>가 결속돼 있으며 이전의 어떤 쓰기 동작이 저장 공간 불충분 때문에 실패했다.</dd>
</dl>

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, 4.3BSD.

## AVAILABILITY

`fdatasync()`가 사용 가능한 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_SYNCHRONIZED_IO`가 0보다 큰 값으로 정의되어 있다. (<tt>[[sysconf(3)]]</tt>도 참고.)

## NOTES

어떤 유닉스 시스템에서는 (리눅스는 아님) `fd`가 <em>쓰기 가능한</em> 파일 디스크립터여야 한다.

리눅스 2.2 및 이전에서는 `fdatasync()`가 `fsync()`와 동등하며, 그래서 성능에서 유리한 점이 없다.

오래된 커널과 잘 안 쓰는 파일 시스템의 `fsync()` 구현에서는 디스크 캐시를 플러시 할 줄 모른다. 이 경우 안전한 동작을 보장하려면 `hdparm(8)`이나 `sdparm(8)`을 이용해 디스크 캐시를 비활성화해야 한다.

## SEE ALSO

`sync(1)`, <tt>[[bdflush(2)]]</tt>, <tt>[[open(2)]]</tt>, <tt>[[posix_fadvise(2)]]</tt>, <tt>[[pwritev(2)]]</tt>, <tt>[[sync(2)]]</tt>, <tt>[[sync_file_range(2)]]</tt>, <tt>[[fflush(3)]]</tt>, <tt>[[fileno(3)]]</tt>, `hdparm(8)`, <tt>[[mount(8)]]</tt>

----

2019-03-06
