## NAME

vmsplice - 사용자 페이지를 파이프와 연결하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <fcntl.h>
#include <sys/uio.h>

ssize_t vmsplice(int fd, const struct iovec *iov,
                 unsigned long nr_segs, unsigned int flags);
```

## DESCRIPTION

`fd`가 쓰기용으로 열려 있으면 `vmsplice()` 시스템 호출은 `iov`가 기술하는 `nr_segs` 개 사용자 메모리 범위를 파이프로 매핑 한다. `fd`가 읽기용으로 열려 있으면 `vmsplice()` 시스템 호출은 `iov`가 기술하는 `nr_segs` 개 사용자 메모리 범위를 파이프에서 읽은 내용으로 채운다. 파일 디스크립터 `fd`는 파이프를 가리켜야 한다.

포인터 `iov`는 `<sys/uio.h>`에 정의된 `iovec` 구조체의 배열을 가리킨다.

```c
struct iovec {
    void  *iov_base;        /* 시작 주소 */
    size_t iov_len;         /* 바이트 수 */
};
```

`flags` 인자는 다음 값을 0개 이상 OR 해서 구성한 비트 마스크이다.

<dl>
<dt><code>SPLICE_F_MOVE</code></dt>
<dd><code>vmsplice()</code>에서는 쓰지 않는다. <tt>[[splice(2)]]</tt> 참고.</dd>
<dt><code>SPLICE_F_NONBLOCK</code></dt>
<dd>I/O에서 블록 하지 않는다. 자세한 내용은 <tt>[[splice(2)]]</tt> 참고.</dd>
<dt><code>SPLICE_F_MORE</code></dt>
<dd>현재 <code>vmsplice()</code>에 아무 효과가 없지만 향후에 구현될 수도 있다. <tt>[[splice(2)]]</tt> 참고.</dd>
<dt><code>SPLICE_F_GIFT</code></dt>
<dd>사용자 페이지가 커널에 주는 선물이다. 응용에서 그 메모리를 절대 변경해서는 안 되며, 만약 그러면 페이지 캐시와 디스크 상의 데이터가 달라질 수 있다. 커널에 페이지를 선물한다는 것은 이어지는 <tt>[[splice(2)]]</tt> <code>SPLICE_F_MOVE</code>에서 그 페이지를 성공적으로 옮길 수 있다는 뜻이다. 이 플래그를 지정하지 않으면 이어지는 <tt>[[splice(2)]]</tt> <code>SPLICE_F_MOVE</code>에서 페이지를 복사해야 한다. 데이터의 위치와 길이 모두 페이지에 맞게 정렬되어 있어야 한다.</dd>
</dl>

## RETURN VALUE

성공 완료 시 `vmsplice()`는 파이프로 전달한 바이트 수를 반환한다. 오류 시 `vmsplice()`는 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EAGAIN</code></dt>
<dd><code>flags</code>에 <code>SPLICE_F_NONBLOCK</code>이 지정되었으며 동작이 블록 되려 한다.</dd>
<dt><code>EBADF</code></dt>
<dd><code>fd</code>가 유효하지 않거나 파이프를 가리키고 있지 않다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>nr_segs</code>가 <code>IOV_MAX</code>보다 크다. 또는 <code>SPLICE_F_GIFT</code>가 설정된 경우에서 메모리가 정렬되어 있지 않다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>메모리 부족.</dd>
</dl>

## VERSIONS

리눅스 2.6.17에서 `vmsplice()` 시스템 호출이 처음 등장했다. glibc 버전 2.5에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

전달하는 조각 개수 제한에 있어서 `vmsplice()`는 다른 벡터 방식 읽기/쓰기 함수들을 따른다. 그 제한은 `<limits.h>`에 정의된 `IOV_MAX`이다. 현재 그 제한값은 1024이다.

`vmsplice()`는 사용자 메모리에서 파이프 방향으로만 진짜 연결을 지원한다. 반대 방향으로는 사실 사용자 공간으로 데이터를 복사할 뿐이다. 하지만 양방향을 지원해서 인터페이스가 깔끔하게 대칭이 되며 사람들이 `vmsplice()`를 이용하도록 하고 향후 성능 개선 여지를 남길 수 있다.

## SEE ALSO

<tt>[[splice(2)]]</tt>, <tt>[[tee(2)]]</tt>, <tt>[[pipe(7)]]</tt>

----

2019-03-06
