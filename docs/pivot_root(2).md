## NAME

pivot_root - 루트 파일 시스템 바꾸기

## SYNOPSIS

```c
int pivot_root(const char *new_root, const char *put_old);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`pivot_root()`는 호출 프로세스의 루트 파일 시스템을 디렉터리 `put_old`로 옮기고 `new_root`를 호출 프로세스의 새 루트 파일 시스템으로 만든다.

대표적으로 `pivot_root()`를 쓰는 곳이 시스템 시동 때인데, 시스템에서 임시 루트 파일 시스템(가령 `initrd`)을 마운트 하고서 실제 루트 파일 시스템을 마운트 한 다음 최종적으로 후자를 모든 관련 프로세스 내지 스레드의 현재 루트로 바꾼다.

이전 루트 디렉터리를 쓰는 프로세스 내지 스레드의 현재 루트 및 현재 작업 디렉터리를 `pivot_root()`에서 바꿀 수도 있고 바꾸지 않을 수도 있다. `pivot_root()`를 호출하는 쪽에선 이전 루트를 루트나 현재 작업 디렉터리로 하는 프로세스가 어느 경우에도 올바르게 동작하도록 해야 한다. 그걸 보장하는 손쉬운 방법 하나는 `pivot_root()` 호출 전에 그 프로세스들의 루트 및 현재 작업 디렉터리를 `new_root`로 바꾸는 것이다.

위 문단은 의도적으로 모호하게 작성됐는데, `pivot_root()` 구현이 향후에 바뀔 수도 있기 때문이다. 작성 시점 현재는 각 프로세스 내지 스레드의 루트 및 현재 작업 디렉터리가 이전 루트 디렉터리를 가리키고 있으면 그걸 `pivot_root()`에서 `new_root`로 바꾼다. 이렇게 해 주어야 전혀 파일 시스템에 접근도 하지 않는 커널 스레드가 이전 루트 디렉터리를 자기 루트 및 현재 작업 디렉터리로 잡아서 계속 사용 중이도록 하는 걸 막을 수 있다. 향후에는 커널 스레드에서 명시적으로 파일 시스템 접근을 포기하는 메커니즘이 있어서 꽤 침습적인 이 메커니즘을 `pivot_root()`에서 없앨 수 있게 될 수도 있다.

참고로 호출 프로세스에도 위 사항이 적용된다. 즉 `pivot_root()`가 호출 프로세스의 현재 작업 디렉터리에 영향을 줄 수도 있고 주지 않을 수도 있다. 따라서 `pivot_root()` 직후에 `chdir("/")`을 호출하기를 권장한다.

`new_root`와 `put_old`에 다음 제약들이 적용된다.

* 디렉터리여야 한다.

* `new_root`와 `put_old`가 현재 루트와 같은 파일 시스템 상에 있어선 안 된다.

* `put_old`가 `new_root` 아래에 있어야 한다. 즉 `put_old`가 가리키는 문자열에 `/..`를 1개 이상 덧붙여서 `new_root`와 동일한 디렉터리가 나와야 한다.

* `put_old`에 다른 파일 시스템이 마운트 돼 있어선 안 된다.

더 많은 사용례는 `pivot_root(8)`를 보라.

현재 루트가 (가령 <tt>[[chroot(2)]]</tt>나 `pivot_root()`를 호출한 후에, 아래 참고) 마운트 지점이 아니면 이전 루트 디렉터리가 아니라 그 파일 시스템의 마운트 지점이 `put_old`에 마운트 된다.

`new_root`가 마운트 지점이어야 한다. (아니라면 `new_root`에 스스로를 바인드 바운트 해 주면 된다.)

`new_root`와 그 부모 마운트의 전파 유형이 `MS_SHARED`여선 안 된다. 마찬가지로 `put_old`가 기존 마운트 지점인 경우 그 전파 유형이 `MS_SHARED`여선 안 된다.

`new_root`가 꼭 마운트 지점일 필요는 없다. 그 경우에는 `/proc/mounts`에서 `new_root`를 담은 파일 시스템의 마운트 지점을 루트(`/`)로 보여 주게 된다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<tt>[[stat(2)]]</tt>이 반환하는 오류들을 `pivot_root()`가 (`errno`로) 반환할 수 있다. 더불어 다음을 반환할 수 있다.

<dl>
<dt><code>EBUSY</code></dt>
<dd>현재 루트 파일 시스템 상에 <code>new_root</code>나 <code>put_old</code>가 있거나 <code>put_old</code>에 이미 파일 시스템이 마운트 돼 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>new_root</code>가 마운트 지점이 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>put_old</code>가 <code>new_root</code> 아래에 있지 않다.</dd>
<dt><code>EINVAL</code></dt>
<dd>현재 루트가 rootfs(초기 ramfs) 파일 시스템 파일 시스템 상에 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>new_root</code>의 마운트 지점이나 그 마운트 지점의 부모 마운트가 전파 유형이 <code>MS_SHARED</code>다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>put_old</code>가 아문트 지점이고 전파 유형이 <code>MS_SHARED</code>다.</dd>
<dt><code>ENOTDIR</code></dt>
<dd><code>new_root</code>나 <code>put_old</code>가 디렉터리가 아니다.</dd>
<dt><code>EPERM</code></dt>
<dd>호출 프로세스가 <code>CAP_SYS_ADMIN</code> 역능을 가지고 있지 않다.</dd>
</dl>

## VERSIONS

리눅스 2.3.41에서 `pivot_root()`가 도입됐다.

## CONFORMING TO

`pivot_root()`는 리눅스 전용이므로 이식성이 없다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

rootfs(초기 ramfs)를 `pivot_root()` 할 수 없다. 이 경우 루트 파일 시스템을 바꾸는 권장하는 방법은 rootfs에서 모든 걸 지우고, 새 root로 rootfs를 덮어서 마운트 하고, 새 `/dev/console`에 `stdin`/`stdout`/`stderr`를 붙이고, 새 `init(1)`을 exec 하는 것이다. 이 과정을 위한 헬퍼 프로그램들이 존재한다. `switch_root(8)` 참고.

## BUGS

`pivot_root()`가 시스템 내 다른 모든 프로세스들의 루트 및 현재 작업 디렉터리를 바꿔야 해서는 안 된다.

덜 명확한 `pivot_root()` 사용 방식 몇 가지는 빠르게 사람을 돌게 만들 수 있다.

## SEE ALSO

<tt>[[chdir(2)]]</tt>, <tt>[[chroot(2)]]</tt>, <tt>[[mount(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[initrd(4)]]</tt>, `pivot_root(8)`, `switch_root(8)`

----

2019-08-02
