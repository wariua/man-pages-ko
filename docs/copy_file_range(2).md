## NAME

copy_file_range - 한 파일에서 다른 파일로 데이터 구간 복사하기

## SYNOPSIS

```c
#include _GNU_SOURCE
#include <unistd.h>

ssize_t copy_file_range(int fd_in, off64_t *off_in,
                        int fd_out, off64_t *off_out,
                        size_t len, unsigned int flags);
```

## DESCRIPTION

`copy_file_range()` 시스템 호출은 커널에서 사용자 공간으로 데이터를 보내고 받는 추가 비용 없이 커널 내에서 두 파일 디스크립터 사이에 복사를 수행한다. 출발 파일 디스크립터 `fd_in`에서 대상 파일 디스크립터 `fd_out`으로 최대 `len` 바이트까지 데이터를 복사하며, 대상 파일의 요청 범위 안에 데이터가 존재하면 덮어 쓴다.

`off_in`을 다음처럼 처리하며, 마찬가지 내용이 `off_out`에도 적용된다.

* `off_in`이 NULL이면 파일 오프셋부터 시작해서 `fd_in`에서 바이트들을 읽어 들이며, 복사한 바이트 수만큼 파일 오프셋을 조정한다.

* `off_in`이 NULL이 아니면 `off_in`은 `fd_in`에서 바이트들을 읽어 들이기 시작할 오프셋을 나타내는 블록을 가리켜야 한다. `fd_in`의 파일 오프셋이 바뀌지 않으며, 대신 `off_in`을 적절히 조정한다.

`fd_in`과 `fd_out`이 같은 파일을 가리킬 수 있다. 같은 파일을 가리키는 경우에는 출발 범위와 대상 범위가 겹칠 수 없다.

`flags` 인자는 향후 확장을 위한 것이며 현재는 0으로 설정해야 한다.

## RETURN VALUE

성공 완료 시 `copy_file_range()`는 파일들 사이에서 복사한 바이트 수를 반환한다. 원래 요청 길이보다 작을 수도 있다. `fd_in`의 파일 오프셋이 파일 끝이나 그 너머에 있는 경우에는 아무것도 복사하지 않으며 `copy_file_range()`가 0을 반환한다.

오류 시 `copy_file_range()`는 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   한 개 이상의 파일 디스크립터가 유효하지 않다.

`EBADF`
:   `fd_in`이 읽기 가능하게 열려 있지 않다. 또는 `fd_out`이 쓰기 가능하게 열려 있지 않다.

`EBADF`
:   파일 디스크립터 `fd_out`이 가리키는 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)에 `O_APPEND` 플래그가 설정돼 있다.

`EFBIG`
:   커널에서 지원하는 최대 파일 오프셋을 넘는 위치에서 쓰기 시도가 이뤄졌다.

`EFBIG`
:   허용된 최대 파일 크기를 초과하는 범위로 쓰기 시도가 이뤄졌다. 최대 파일 크기는 파일 시스템 구현에 따라 다르며 허용되는 최대 파일 오프셋과 다를 수 있다.

`EFBIG`
:   프로세스의 파일 크기 자원 제한을 초과하도록 쓰기 시도가 이뤄졌다. 이로 인해 프로세스가 `SIGXFSZ` 시그널도 받게 될 수 있다.

`EINVAL`
:   `flags` 인자가 0이 아니다.

`EINVAL`
:   `fd_in`과 `fd_out`이 같은 파일을 가리키며 출발 범위와 대상 범위가 겹친다.

`EINVAL`
:   `fd_in`이나 `fd_out`이 정규 파일이 아니다.

`EIO`
:   복사 중에 저수준 I/O 오류가 발생했다.

`EISDIR`
:   `fd_in`이나 `fd_out`이 디렉터리를 가리키고 있다.

`ENOMEM`
:   메모리 부족.

`ENOSPC`
:   대상 파일 시스템에 복사를 완료하기 위한 충분한 공간이 없다.

`EOVERFLOW`
:   요청한 출발 범위나 대상 범위가 너무 커서 명세된 데이터 타입으로 표현할 수 없다.

`EPERM`
:   `fd_out`이 불변 파일을 가리키고 있다.

`ETXTBSY`
:   `fd_in`이나 `fd_out`이 활성 상태의 스왑 파일을 가리키고 있다.

`EXDEV`
:   `fd_in`과 `fd_out`이 가리키는 파일이 같은 파일 시스템 상에 있지 않다. (리눅스 5.3 전)

## VERSIONS

리눅스 4.5에서 `copy_file_range()` 시스템 호출이 처음 등장했다. 이용 가능하지 않은 경우 glibc 2.27에서 사용자 공간 에뮬레이션을 제공한다.

5.3에서 커널 구현을 크게 고치는 작업이 이뤄졌다. API에서 명확하게 규정돼 있지 않던 부분을 명확히 하고 앞선 커널들보다 API 제한치를 훨씬 엄격하게 검사한다. 응용들에선 5.3 커널의 동작 방식과 요구 사항을 목표로 하는 게 좋다.

리눅스 5.3에서 파일 시스템 간 복사를 지원하기 시작했다. 그 전 커널에선 파일 시스템 간 복사 시도 시 -EXDEV를 반환하게 된다.

## CONFORMING TO

`copy_file_range()` 시스템 호출은 비표준 리눅스 및 GNU 확장이다.

## NOTES

`fd_in`이 희소(sparse) 파일이면 요청 범위 내에 존재하는 구멍을 `copy_file_range()`가 채워 버릴 수 있다. 루프에서 `copy_file_range()`를 호출하면서 <tt>[[lseek(2)]]</tt> `SEEK_DATA` 및 `SEEK_HOLE` 동작으로 데이터 세그먼트 위치를 알아내는 방식이 좋을 수 있다.

`copy_file_range()`를 쓰면 파일 시스템에서 reflink(즉 여러 아이노드가 동일 copy-on-write 디스크 블록에 대한 포인터 공유) 사용이나 서버 측 복사(NFS인 경우) 같은 "복사 가속" 기법을 적용할 기회가 생긴다.

## EXAMPLES

```c
#define _GNU_SOURCE
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <unistd.h>

int
main(int argc, char **argv)
{
    int fd_in, fd_out;
    struct stat stat;
    off64_t len, ret;

    if (argc != 3) {
        fprintf(stderr, "Usage: %s <source> <destination>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    fd_in = open(argv[1], O_RDONLY);
    if (fd_in == -1) {
        perror("open (argv[1])");
        exit(EXIT_FAILURE);
    }

    if (fstat(fd_in, &stat) == -1) {
        perror("fstat");
        exit(EXIT_FAILURE);
    }

    len = stat.st_size;

    fd_out = open(argv[2], O_CREAT | O_WRONLY | O_TRUNC, 0644);
    if (fd_out == -1) {
        perror("open (argv[2])");
        exit(EXIT_FAILURE);
    }

    do {
        ret = copy_file_range(fd_in, NULL, fd_out, NULL, len, 0);
        if (ret == -1) {
            perror("copy_file_range");
            exit(EXIT_FAILURE);
        }

        len -= ret;
    } while (len > 0 && ret > 0);

    close(fd_in);
    close(fd_out);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[lseek(2)]]</tt>, <tt>[[sendfile(2)]]</tt>, <tt>[[splice(2)]]</tt>

----

2021-03-22
