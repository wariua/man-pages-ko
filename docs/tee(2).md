## NAME

tee - 파이프 내용물 복제하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <fcntl.h>

ssize_t tee(int fd_in, int fd_out, size_t len, unsigned int flags);
```

## DESCRIPTION

`tee()`는 파일 디스크립터 `fd_in`이 가리키는 파이프에서 온 데이터를 파일 디스크립터 `fd_out`이 가리키는 파이프로 최대 `len` 바이트까지 복제한다. `fd_in`에서 복제한 데이터를 소모하지 않는다. 따라서 이어지는 <tt>[[splice(2)]]</tt>로 그 데이터를 복사할 수 있다.

`flags`는 다음 값을 0개 이상 OR 해서 구성한 비트 마스크이다.

<dl>
<dt><code>SPLICE_F_MOVE</code></dt>
<dd>현재 <code>tee()</code>에 아무 효과가 없다. <tt>[[splice(2)]]</tt> 참고.</dd>
<dt><code>SPLICE_F_NONBLOCK</code></dt>
<dd>I/O에서 블록 하지 않는다. 자세한 내용은 <tt>[[splice(2)]]</tt> 참고.</dd>
<dt><code>SPLICE_F_MORE</code></dt>
<dd>현재 <code>tee()</code>에 아무 효과가 없지만 향후에 구현될 수도 있다. <tt>[[splice(2)]]</tt> 참고.</dd>
<dt><code>SPLICE_F_GIFT</code></dt>
<code>tee()</code>에서는 쓰지 않는다. <tt>[[vmsplice(2)]]</tt> 참고.
</dl>

## RETURN VALUE

성공 완료 시 `tee()`는 입력과 출력 간에 복제한 바이트 수를 반환한다. 반환 값 0은 이동시킬 데이터가 없었으며 `fd_in`이 가리키는 파이프의 쓰기 쪽에 연결된 쓰기 수행자가 없어서 블록 하는 것이 의미가 없었음을 뜻한다.

오류 시 `tee()`는 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EAGAIN</code></dt>
<dd><code>flags</code>에 <code>SPLICE_F_NONBLOCK</code>이 지정되었거나 파일 디스크립터들 중 하나에 논블로킹 표시(<code>O_NONBLOCK</code>)가 돼 있는데 동작이 블록 되려 한다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>fd_in</code>이나 <code>fd_out</code>이 파이프를 가리키고 있지 않다. 또는 <code>fd_in</code>과 <code>fd_out</code>이 같은 파이프를 가리키고 있다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>메모리 부족.</dd>
</dl>

## VERSIONS

리눅스 2.6.17에서 `tee()` 시스템 호출이 처음 등장했다. glibc 버전 2.5에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

개념적으로 `tee()`는 두 파이프 간에 데이터를 복사한다. 하지만 실제로는 어떤 데이터 복사도 이뤄지지 않는다. 내부적으로 `tee()`는 입력에 대한 참조만 잡고서 데이터를 출력 쪽으로 할당한다.

## EXAMPLE

아래 예에서는 `tee()` 시스템 호출을 이용해 기본적인 `tee(1)` 프로그램을 구현한다. 다음이 사용 예시이다.

```
$ date |./a.out out.log | cat
Tue Oct 28 10:06:00 CET 2014
$ cat out.log
Tue Oct 28 10:06:00 CET 2014
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <limits.h>

int
main(int argc, char *argv[])
{
    int fd;
    int len slen;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s <file>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    fd = open(argv[1], O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }

    do {
        /*
         * stdin을 stdout으로 분기 연결.
         */
        len = tee(STDIN_FILENO, STDOUT_FILENO,
                  INT_MAX, SPLICE_F_NONBLOCK);

        if (len < 0) {
            if (errno == EAGAIN)
                continue;
            perror("tee");
            exit(EXIT_FAILURE);
        } else
            if (len == 0)
                break;

        /*
         * stdin을 파일로 연결해서 데이터 먹기.
         */
        while (len > 0) {
            slen = splice(STDIN_FILENO, NULL, fd, NULL,
                          len, SPLICE_F_MOVE);
            if (slen < 0) {
                perror("splice");
                break;
            }
            len -= slen;
        }
    } while (1);

    close(fd);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[splice(2)]]</tt>, <tt>[[vmsplice(2)]]</tt>, <tt>[[pipe(7)]]</tt>

----

2019-03-06
