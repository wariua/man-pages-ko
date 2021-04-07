## NAME

chroot - 루트 디렉터리 바꾸기

## SYNOPSIS

```c
#include <unistd.h>

int chroot(const char *path);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>chroot()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.2.2부터:</dt>
 <dd>
<code>_XOPEN_SOURCE && ! (_POSIX_C_SOURCE >= 200112L)</code><br>
<code>    || /* glibc 2.20부터: */ _DEFAULT_SOURCE</code><br>
<code>    || /* glibc 버전 <= 2.19: */ _BSD_SOURCE</code>
 </dd>
 <dt>glibc 2.2.2 전:</dt>
 <dd>없음</dd>
 </dl>
</dd>

## DESCRIPTION

`chroot()`는 호출 프로세스의 루트 디렉터리를 `path`에 지정한 디렉터리로 바꾼다. `/`으로 시작하는 경로명에 그 디렉터리를 쓰게 된다. 호출 프로세스의 모든 자식들이 그 루트 디렉터리를 물려받는다.

특권을 가진 (리눅스: 자기 사용자 네임스페이스에서 `CAP_SYS_CHROOT` 역능을 가진) 프로세스만 `chroot()`를 호출할 수 있다.

이 호출은 경로명 해석 과정의 재료 하나를 바꿀 뿐 다른 아무것도 하지 않는다. 특히 완전한 프로세스 샌드박스나 파일 시스템 호출 제약처럼 어떤 종류든 보안 목적으로 쓰기 위한 것이 아니다. 과거에는 데몬들에서 `chroot()`를 사용해 스스로를 제약한 후에 신뢰할 수 없는 사용자가 제공한 경로를 <tt>[[open(2)]]</tt> 같은 시스템 호출에 전달했다. 하지만 폴더가 chroot 디렉터리 밖으로 옮겨진다면 공격자가 그걸 악용해 chroot 디렉터리를 벗어날 수 있다. 가장 간단한 방법은 옮겨질 디렉터리로 <tt>[[chdir(2)]]</tt> 하고서 이동되길 기다린 다음 `../../../etc/passwd` 같은 경로를 여는 것이다.

<tt>[[chdir(2)]]</tt>이 허용되지 않는다면 살짝 더 복잡한 변형 기법이 일부 경우에 통한다. 어떤 데몬의 "chroot 디렉터리"를 지정할 수 있다면 그건 일반적으로 원격 사용자가 chroot 디렉터리 밖의 파일에 접근하지 못하게 하려면 절대로 폴더들이 밖으로 옮겨지지 않게 해야 한다는 뜻이다.

이 호출은 현재 작업 디렉터리를 바꾸지 않으며, 따라서 호출 후에 '`.`'가 '`/`'에 위치한 트리 밖에 있을 수 있다. 특히 수퍼유저가 다음과 같이 해서 "chroot 감옥"을 탈출할 수 있다.

```
mkdir foo; chroot foo; cd ..
```

이 호출은 열린 파일 디스크립터를 닫지 않으며, 그래서 그런 파일 디스크립터를 통해 chroot 트리 밖의 파일에 접근이 가능할 수도 있다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

파일 시스템에 따라 다른 오류들을 반환할 수 있다. 아래에 나열한 것은 일반적인 오류들이다.

<dl>
<dt><code>EACCES</code></dt>
<dd>경로 선두부의 어느 요소에 대해 탐색 권한이 거부되었다. (<tt>[[path_resolution(7)]]</tt> 참고.)</dd>
<dt><code>EFAULT</code></dt>
<dd><code>path</code>가 접근 가능한 주소 공간 밖을 가리키고 있다.</dd>
<dt><code>EIO</code></dt>
<dd>I/O 오류가 발생했다.</dd>
<dt><code>ELOOP</code></dt>
<dd><code>path</code>를 해석하는 동안 너무 많은 심볼릭 링크를 만났다.</dd>
<dt><code>ENAMETOOLONG</code></dt>
<dd><code>path</code>가 너무 길다.</dd>
<dt><code>ENOENT</code></dt>
<dd>파일이 존재하지 않는다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>사용 가능한 커널 메모리가 충분하지 않다.</dd>
<dt><code>ENOTDIR</code></dt>
<dd><code>path</code>의 한 요소가 디렉터리가 아니다.</dd>
<dt><code>EPERM</code></dt>
<dd>호출자에게 충분한 특권이 없다.</dd>
</dl>

## CONFORMING TO

SVr4, 4.4BSD, SUSv2 (LEGACY로 표시). 이 함수는 POSIX.1-2001에 포함되어 있지 않다.

## NOTES

<tt>[[fork(2)]]</tt>를 통해 생긴 자식 프로세스는 부모의 루트 디렉터리를 물려받는다. <tt>[[execve(2)]]</tt>에서 루트 디렉터리를 바꾸지 않는다.

특수 심볼릭 링크 `/proc/[pid]/root`를 사용해 프로세스의 루트 디렉터리를 알아낼 수 있다. 자세한 내용은 <tt>[[proc(5)]]</tt> 참고.

FreeBSD에는 더 강력한 `jail()` 시스템 호출이 있다.

## SEE ALSO

`chroot(1)`, <tt>[[chdir(2)]]</tt>, <tt>[[pivot_root(2)]]</tt>, <tt>[[path_resolution(7)]]</tt>, `switch_root(8)`

----

2019-03-06
