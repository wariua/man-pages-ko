## NAME

mmap2 - 파일이나 장치를 메모리로 맵 하기

## SYNOPSIS

```c
#include <sys/mman.h>

void *mmap2(void *addr, size_t length, int prot,
            int flags, int fd, off_t pgoffset);
```

## DESCRIPTION

아마 찾으려던 시스템 호출이 아닐 것이다. 이 시스템 호출을 부르는 glibc 래퍼 함수를 기술하는 <tt>[[mmap(2)]]</tt>을 보라.

`mmap2()` 시스템 호출은 <tt>[[mmap(2)]]</tt>과 같은 인터페이스를 제공하되 마지막 인자가 (<tt>[[mmap(2)]]</tt>에서처럼 바이트 단위가 아니라) 4096바이트 단위로 파일 내 오프셋을 지정한다. 그래서 응용에서 32비트 `off_t`를 사용해 큰 (최대 2^44바이트) 파일을 맵 할 수 있게 된다.

## RETURN VALUE

성공 시 `mmap2()`는 맵 한 영역에 대한 포인터를 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EFAULT</code></dt>
<dd>사용자 공간으로부터 데이터를 가져오는 데 문제 발생.</dd>
<dt><code>EINVAL</code></dt>
<dd>(페이지 크기가 4096바이트가 아닌 다양한 플랫폼에서) <code>offset * 4096</code>이 시스템 페이지 크기의 배수가 아니다.</dd>
</dl>

`mmap2()`가 <tt>[[mmap(2)]]</tt>에서 기술하는 오류들 중 하나를 반환할 수도 있다.

## VERSIONS

리눅스 2.3.31부터 `mmap2()`가 사용 가능하다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

이 시스템 호출이 존재하는 플랫폼에서는 glibc의 `mmap()` 래퍼 함수가 <tt>[[mmap(2)]]</tt> 시스템 호출 대신 이 시스템 호출을 실행한다.

x86-64에는 이 시스템 호출이 존재하지 않는다.

ia64에서는 `offset`의 단위가 실제로는 4096바이트가 아니라 시스템 페이지 크기이다.

## SEE ALSO

<tt>[[getpagesize(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[mremap(2)]]</tt>, <tt>[[msync(2)]]</tt>, <tt>[[shm_open(3)]]</tt>

----

2017-09-15
