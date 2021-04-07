## NAME

fallocate - 파일 공간 조작하기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <fcntl.h>

int fallocate(int fd, int mode, off_t offset, off_t len);
```

## DESCRIPTION

이식성 없는 리눅스 전용 시스템 호출이다. 파일 공간 할당을 보장하기 위한 이식성 있고 POSIX.1에 명세된 방법에 대해선 <tt>[[posix_fallocate(3)]]</tt>를 보라.

`fallocate()`를 통해 호출자는 `fd`가 가리키는 파일의 `offset`부터 `len` 바이트만큼 이어지는 바이트 범위에 대해 할당 디스크 공간을 직접 조작할 수 있다.

`mode` 인자는 해당 범위에 수행할 동작을 결정한다. 지원하는 동작들을 아래에서 자세히 설명한다.

### 디스크 공간 할당하기

`fallocate()`의 기본 동작(즉 `mode` 0)은 `offset`과 `len`으로 지정한 범위의 디스크 공간을 할당한다. `offset`+`len`이 (<tt>[[stat(2)]]</tt>이 알려 주는) 파일 크기보다 크면 파일 크기가 바뀌게 된다. `offset`과 `len`으로 지정한 범위 내에서 호출 전에 데이터를 담고 있지 않았던 영역은 0으로 초기화 된다. 이 기본 동작 방식은 라이브러리 함수 <tt>[[posix_fallocate(3)]]</tt>의 동작과 매우 비슷한데, 그 함수를 최적으로 구현할 수 있도록 하기 위한 것이다.

성공 호출 후에는 `offset`과 `len`으로 지정한 범위 내에서의 쓰기 동작이 디스크 공간 부족 때문에 실패하지 않는 게 보장된다.

`mode`에 `FALLOC_FL_KEEP_SIZE` 플래그를 지정하면 호출이 비슷하게 동작하되 `offset`+`len`이 파일 크기보다 큰 경우에도 파일 크기가 바뀌지 않게 된다. 이런 식으로 파일 끝 너머에 0 채운 블록들을 미리 할당해 두는 것이 덧붙이기 작업 최적화에 쓸모가 있다.

`mode`에 `FALLOC_FL_UNSHARE` 플래그를 지정하면 공유 파일 데이터 익스텐트(extent)를 그 파일 전용으로 만들어서 이후의 쓰기 동작이 공간 부족으로 실패하지 않게 보장한다. 보통 이를 위해 파일 내 모든 공유 데이터에 copy-on-write 동작을 수행한다. 모든 파일 시스템에서 이 플래그를 지원하지는 않을 수도 있다.

할당이 블록 크기 단위로 이뤄지기 때문에 지정한 것보다 큰 디스크 공간 범위를 `fallocate()`에서 할당할 수도 있다.

### 파일 공간 해제하기

`mode`에 (리눅스 2.6.38부터 사용 가능한) `FALLOC_FL_PUNCH_HOLE` 플래그를 지정하면 `offset`부터 `len` 바이트만큼 이어지는 바이트 범위의 공간을 해제한다. (즉 구멍을 만든다.) 지정 범위 내에 일부만 속하는 파일 시스템 블록은 0으로 채워지고 전체가 속하는 파일 시스템 블록은 파일에서 제거된다. 성공 호출 후에는 그 범위에서의 읽기 동작이 0 값들을 반환하게 된다.

`mode`에서 `FALLOC_FL_PUNCH_HOLE` 플래그에 반드시 `FALLOC_FL_KEEP_SIZE`를 OR 해서 써야 한다. 달리 말해 파일 끝에 구멍을 내는 경우에도 (<tt>[[stat(2)]]</tt>이 알려 주는) 파일 크기는 바뀌지 않는다.

모든 파일 시스템에서 `FALLOC_FL_PUNCH_HOLE`을 지원하지는 않는다. 파일 시스템에서 이 동작을 지원하지 않으면 오류를 반환한다. 적어도 다음 파일 시스템에서 이 동작을 지원한다.

* XFS (리눅스 2.6.38부터)

* ext4 (리눅스 3.0부터)

* Btrfs (리눅스 3.7부터)

* <tt>[[tmpfs(5)]]</tt> (리눅스 3.5부터)

### 파일 공간 압착하기

`mode`에 (리눅스 3.15부터 사용 가능한) `FALLOC_FL_COLLAPSE_RANGE` 플래그를 지정하면 파일에서 어떤 바이트 범위를 구멍을 남기지 않고 제거한다. 압착할 바이트 범위가 `offset`부터 `len` 바이트만큼 이어진다. 동작이 완료되면 `offset+len` 위치부터의 파일 내용물이 `offset` 위치에 붙으며 파일이 `len` 바이트만큼 작아진다.

파일 시스템에서 효율적 구현이 가능케 하기 위해 동작 단위에 제한을 둘 수 있다. 보통 `offset`과 `len`이 파일 시스템의 논리적 블록 크기의 배수여야 하는데, 그 크기는 파일 시스템 종류와 설정에 따라 다르다. 파일 시스템에 그런 조건이 있는 경우 그 요건이 충족되지 않으면 `fallocate()`가 `EINVAL` 오류로 실패한다.

`offset` 더하기 `len`이 나타내는 범위가 파일 끝에 닿거나 끝을 넘으면 오류를 반환한다. 파일을 절단하려면 <tt>[[ftruncate(2)]]</tt>를 쓰면 된다.

`mode`에 `FALLOC_FL_COLLAPSE_RANGE`와 다른 플래그를 함께 지정할 수 없다.

리눅스 3.15 기준으로 ext4(익스텐트 기반 파일 한정)와 XFS에서 `FALLOC_FL_COLLAPSE_RANGE`를 지원한다.

### 파일 공간 0으로 채우기

`mode`에 (리눅스 3.15부터 사용 가능한) `FALLOC_FL_ZERO_RANGE` 플래그를 지정하면 `offset`부터 `len` 바이트만큼 이어지는 바이트 범위의 공간을 0으로 채운다. 지정 범위 내에서 파일 내 구멍에 걸쳐 있는 영역들에 블록을 미리 할당한다. 성공 호출 후에는 그 범위에서의 읽기 동작이 0을 반환하게 된다.

파일 시스템 내에서 0으로 채우는 동작은 가급적이면 그 범위를 기록 안 된 익스텐트로 변환하는 방식으로 이뤄진다. 이 방식에서는 장치 상에서 지정 범위가 (범위 양끝의 일부만 속한 블록을 제외하고) 물리적으로 0으로 채워지는 게 아니며, (일부만 속한 블록이 없다면) 메타데이터 갱신에만 I/O가 필요하다.

`mode`에 `FALLOC_FL_KEEP_SIZE` 플래그를 추가로 지정하면 호출이 비슷하게 동작하되 `offset`+`len`이 파일 크기보다 큰 경우에도 파일 크기가 바뀌지 않게 된다. 이는 공간을 미리 할당하면서 `FALLOC_FL_KEEP_SIZE`를 지정할 때와 같은 동작 방식이다.

모든 파일 시스템에서 `FALLOC_FL_ZERO_RANGE`를 지원하지는 않는다. 파일 시스템에서 이 동작을 지원하지 않으면 오류를 반환한다. 적어도 다음 파일 시스템에서 이 동작을 지원한다.

* XFS (리눅스 3.15부터)

* ext4, 익스텐트 기반 파일 (리눅스 3.15부터)

* SMB3 (리눅스 3.17부터)

* Btrfs (리눅스 4.16부터)

### 파일 공간 늘이기

`mode`에 (리눅스 4.1부터 사용 가능한) `FALLOC_FL_INSERT_RANGE` 플래그를 지정하면 기존 데이터를 덮어 쓰지 않으면서 파일 크기 내에 구멍을 집어넣어서 파일 공간을 늘인다. 그 구멍은 `offset`부터 `len` 바이트만큼 이어지게 된다. 파일 내부에 구멍을 집어넣을 때 `offset`부터 있는 파일 내용물이 `len` 바이트만큼 위쪽으로 (즉 파일 오프셋이 높아지는 쪽으로) 밀리게 된다. 파일 내부에 구멍을 집어넣으면 파일 크기가 `len` 바이트만큼 커진다.

이 모드는 동작 단위와 관련해 `FALLOC_FL_COLLAPSE_RANGE`와 같은 제약이 있다. 크기 단위 요건이 충족되지 않으면 `fallocate()`가 `EINVAL` 오류로 실패한다. `offset`이 파일 끝과 같거나 그보다 크면 오류를 반환한다. 그런 동작에는 (즉 파일 끝에 구멍을 집어넣는 데는) <tt>[[ftruncate(2)]]</tt>를 쓰면 된다.

`mode`에 `FALLOC_FL_INSERT_RANGE`와 다른 플래그를 함께 지정할 수 없다.

`FALLOC_FL_INSERT_RANGE`에는 파일 시스템 지원이 필요하다. 이 동작을 지원하는 파일 시스템으로 XFS(리눅스 4.1부터)와 ext4(리눅스 4.2부터) 등이 있다.

## RETURN VALUE

성공 시 `fallocate()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EBADF</code></dt>
<dd><code>fd</code>가 유효한 파일 디스크립터가 아니거나 쓰기 가능하게 열려 있지 않다.</dd>
<dt><code>EFBIG</code></dt>
<dd><code>offset</code>+<code>len</code>이 최대 파일 크기를 초과한다.</dd>
<dt><code>EFBIG</code></dt>
<dd><code>mode</code>가 <code>FALLOC_FL_INSERT_RANGE</code>이며 현재 파일 크기+<code>len</code>이 최대 파일 크기를 초과한다.</dd>
<dt><code>EINTR</code></dt>
<dd>실행 중 시그널을 잡았다. <tt>[[signal(7)]]</tt> 참고.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>offset</code>이 0보다 작거나 <code>len</code>이 0 이하이다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>mode</code>가 <code>FALLOC_FL_COLLAPSE_RANGE</code>인데 <code>offset</code> 더하기 <code>len</code>이 나타내는 범위가 파일 끝에 닿거나 끝을 넘는다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>mode</code>가 <code>FALLOC_FL_INSERT_RANGE</code>인데 <code>offset</code>이 나타내는 범위가 파일 끝에 닿거나 끝을 넘는다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>mode</code>가 <code>FALLOC_FL_COLLAPSE_RANGE</code>나 <code>FALLOC_FL_INSERT_RANGE</code>인데 <code>offset</code>이나 <code>len</code>이 파일 시스템 블록 크기의 배수가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>mode</code>에 <code>FALLOC_FL_COLLAPSE_RANGE</code>나 <code>FALLOC_FL_INSERT_RANGE</code> 중 하나와 더불어 다른 플래그가 있다. <code>FALLOC_FL_COLLAPSE_RANGE</code>와 <code>FALLOC_FL_INSERT_RANGE</code>에 다른 플래그를 허용하지 않는다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>mode</code>가 <code>FALLOC_FL_COLLAPSE_RANGE</code>나 <code>FALLOC_FL_ZERO_RANGE</code>, <code>FALLOC_FL_INSERT_RANGE</code>인데 <code>fd</code>가 가리키는 파일이 정규 파일이 아니다.</dd>
<dt><code>EIO</code></dt>
<dd>파일 시스템에서 읽기나 쓰기를 하는 중에 I/O 오류가 발생했다.</dd>
<dt><code>ENODEV</code></dt>
<dd><code>fd</code>가 정규 파일이나 디렉터리를 가리키고 있지 않다. (<code>fd</code>가 파이프라 FIFO이면 다른 오류가 나온다.)</dd>
<dt><code>ENOSPC</code></dt>
<dd><code>fd</code>가 가리키는 파일을 담은 장치에 충분한 공간이 남아 있지 않다.</dd>
<dt><code>ENOSYS</code></dt>
<dd>이 커널에서 <code>fallocate()</code>를 구현하고 있지 않다.</dd>
<dt><code>EOPNOTSUPP</code></dt>
<dd><code>fd</code>가 가리키는 파일을 포함하는 파일 시스템에서 이 동작을 지원하지 않는다. 또는 <code>fd</code>가 가리키는 파일을 포함하는 파일 시스템에서 <code>mode</code>를 지원하지 않는다.</dd>
<dt><code>EPERM</code></dt>
<dd><code>fd</code>가 가리키는 파일이 불변으로 표시돼 있다. (<tt>[[chattr(1)]]</tt> 참고.)</dd>
<dt><code>EPERM</code></dt>
<dd><code>mode</code>가 <code>FALLOC_FL_PUNCH_HOLE</code>이나 <code>FALLOC_FL_COLLAPSE_RANGE</code>, <code>FALLOC_FL_INSERT_RANGE</code>를 나타내는데 <code>fd</code>가 가리키는 파일이 덧붙이기 전용으로 표시돼 있다. (<tt>[[chattr(1)]]</tt> 참고.)</dd>
<dt><code>EPERM</code></dt>
<dd>파일 봉인 때문에 동작이 막혔다. <tt>[[fcntl(2)]]</tt> 참고.</dd>
<dt><code>ESPIPE</code></dt>
<dd><code>fd</code>가 파이프나 FIFO를 가리키고 있다.</dd>
<dt><code>ETXTBSY</code></dt>
<dd><code>mode</code>에 <code>FALLOC_FL_COLLAPSE_RANGE</code>나 <code>FALLOC_FL_INSERT_RANGE</code>를 지정했는데 <code>fd</code>가 가리키는 파일이 현재 실행 중이다.</dd>
</dl>

## VERSIONS

리눅스 커널 2.6.23부터 `fallocate()`가 사용 가능하다. glibc 버전 2.10부터 지원을 제공한다. `FALLOC_FL_*` 플래그들은 glibc 버전 2.18부터 헤더에 정의돼 있다.

## CONFORMING TO

`fallocate()`는 리눅스 전용이다.

## SEE ALSO

`fallocate(1)`, <tt>[[ftruncate(2)]]</tt>, <tt>[[posix_fadvise(3)]]</tt>, <tt>[[posix_fallocate(3)]]</tt>

----

2018-04-30
