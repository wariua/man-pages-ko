## NAME

fopencookie - 맞춤형 스트림 열기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <stdio.h>

FILE *fopencookie(void *cookie, const char *mode,
                  cookie_io_functions_t io_funcs);
```

## DESCRIPTION

`fopencookie()` 함수를 통해 프로그래머가 표준 I/O 스트림의 맞춤형 구현을 만들 수 있다. 그 구현에서는 원하는 위치에 스트림의 데이터를 저장할 수 있다. 예를 들어 메모리 내 버퍼에 저장된 데이터에 대해 스트림 인터페이스를 제공하는 <tt>[[fmemopen(3)]]</tt>을 구현하는 데 `fopencookie()`가 쓰인다.

맞춤형 스트림을 만들려면 프로그래머가 다음을 해 주어야 한다.

* 스트림에 I/O를 수행할 때 표준 I/O 라이브러리에서 내부적으로 쓰는 네 가지 "훅" 함수를 구현해야 한다.

* "쿠키" 자료형을 정의한다. 앞서 언급한 훅 함수들에서 사용하는 정보(가령 데이터를 어디에 저장하는지)를 담는 자료 구조이다. 표준 I/O 패키지는 이 쿠키의 내용에 대해 아무것도 알지 못하며 (그래서 `fopencookie()`로 전달할 때 `void *` 타입) 훅 함수 호출 시 기계적으로 쿠키를 첫 번째 인자로 제공하기만 한다.

* `fopencookie()`를 호출해 새 스트림을 열고 스트림에 쿠키와 훅 함수를 연계한다.

`fopencookie()` 함수는 <tt>[[fopen(3)]]</tt>과 비슷한 역할을 한다. 즉 새 스트림을 열고 그 스트림 조작 시 사용할 `FILE` 객체에 대한 포인터를 반환한다.

`cookie` 인자는 호출자의 쿠키 자료 구조에 대한 포인터이며 새 스트림에 연계된다. 표준 I/O 라이브러리에서 아래 기술하는 훅 함수들을 호출할 때 이 포인터를 첫 번째 인자로 제공한다.

`mode` 인자는 <tt>[[fopen(3)]]</tt>에서와 같은 역할을 한다. 지원하는 모드는 `r`, `w`, `a`, `r+`, `w+`, `a+`이다. 자세한 내용은 <tt>[[fopen(3)]]</tt>을 보라.

`io_funcs` 인자는 스트림 구현을 위해 프로그래머가 정의한 훅 함수들을 가리키는 필드 네 개가 담긴 구조체이다. 이 구조체는 다음과 같이 정의돼 있다.

```c
typedef struct {
    cookie_read_function_t  *read;
    cookie_write_function_t *write;
    cookie_seek_function_t  *seek;
    cookie_close_function_t *close;
} cookie_io_functions_t;
```

네 필드는 다음과 같다.

<dl>
<dt><code>cookie_read_function_t *read</code></dt>
<dd>

이 함수는 스트림의 읽기 동작을 구현한다. 호출 시 인자 세 개를 받는다.

```c
ssize_t read(void *cookie, char *buf, size_t size);
```

<code>buf</code> 인자와 <code>size</code> 인자는 각각 입력 데이터를 넣을 수 있는 버퍼와 그 버퍼의 크기이다. 함수 결과로 <code>read</code> 함수는 <code>buf</code>로 복사한 바이트 수를, 또는 파일 끝이면 0을, 또는 오류 시 -1을 반환해야 한다. <code>read</code> 함수는 스트림 오프셋을 적절히 갱신해야 한다.

<code>*read</code>가 널 포인터이면 맞춤형 스트림 읽기가 항상 파일 끝을 반환한다.
</dd>

<dt><code>cookie_write_function_t *write</code></dt>
<dd>

이 함수는 스트림의 쓰기 동작을 구현한다. 호출 시 인자 세 개를 받는다.

```c
ssize_t write(void *cookie, const char *buf, size_t size);
```

<code>buf</code> 인자와 <code>size</code> 인자는 각각 스트림으로 출력할 데이터가 있는 버퍼와 그 버퍼의 크기이다. 함수 결과로 <code>write</code> 함수는 <code>buf</code>로부터 복사한 바이트 수를, 또는 오류 시 0을 반환해야 한다. (함수가 음수 값을 반환해선 안 된다.) <code>write</code> 함수는 스트림 오프셋을 적절히 갱신해야 한다.

<code>*write</code>가 널 포인터이면 스트림에 대한 출력이 버려진다.
</dd>

<dt><code>cookie_seek_function_t *seek</code></dt>
<dd>

이 함수는 스트림의 탐색 동작을 구현한다. 호출 시 인자 세 개를 받는다.

```c
int seek(void *cookie, off64_t *offset, int whence);
```

<code>*offset</code> 인자는 새 파일 오프셋을 나타내는데, <code>whence</code>에 다음 중 어느 값이 있느냐에 따라 달라진다.

 <dl>
 <dt><code>SEEK_SET</code></dt>
 <dd>스트림 오프셋을 스트림 시작점 기준 <code>*offset</code> 바이트로 설정해야 한다.</dd>

 <dt><code>SEEK_CUR</code></dt>
 <dd>현재 스트림 오프셋에 <code>*offset</code>을 더해야 한다.</dd>

 <dt><code>SEEK_END</code></dt>
 <dd>스트림 오프셋을 스트림 크기 더하기 <code>*offset</code>으로 설정해야 한다.</dd>
 </dl>

반환 전에 <code>seek</code> 함수는 새 스트림 오프셋을 나타내도록 <code>*offset</code>을 갱신해야 한다.

함수 결과로 <code>seek</code> 함수는 성공 시 0을 반환하고 오류 시 -1을 반환해야 한다.

<code>*seek</code>이 널 포인터이면 스트림에서 탐색 동작을 수행하는 게 불가능하다.
</dd>

<dt><code>cookie_close_function_t *close</code></dt>
<dd>

이 함수는 스트림을 닫는다. 스트림을 위해 할당했던 버퍼를 해제하는 것 같은 일을 훅 함수에서 할 수 있다. 호출 시 인자 한 개를 받는다.

```c
int close(void *cookie);
```

<code>cookie</code> 인자는 프로그래머가 <code>fopencookie()</code> 호출 시 제공했던 쿠키이다.

함수 결과로 <code>close</code> 함수는 성공 시 0을 반환하고 오류 시 <code>EOF</code>를 반환해야 한다.

<code>*close</code>가 NULL이면 스트림을 닫을 때 별다른 동작을 수행하지 않는다.
</dd>

## RETURN VALUE

성공 시 `fopencookie()`는 새 스트림에 대한 포인터를 반환한다. 오류 시 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `fopencookie()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 비표준 GNU 확장이다.

## EXAMPLE

아래 프로그램에서는 <tt>[[fmemopen(3)]]</tt>을 통해 사용 가능한 것과 기능이 비슷한 (하지만 동일하지는 않은) 맞춤형 스트림을 구현한다. 즉 메모리 버퍼에 데이터를 저장하는 스트림을 구현한다. 프로그램에서 명령행 인자들을 스트림으로 써넣은 다음 스트림을 탐색하며 다섯 글자마다 두 글자씩을 읽어서 표준 출력에 쓴다. 다음 셸 세션이 프로그램 사용 방식을 보여 준다.

```
$ ./a.out 'hello world'
/he/
/ w/
/d/
Reached end of file
```

참고로 아래 프로그램을 더 범용적으로 만들려면 다양한 오류 상황들(가령 스트림 여는 데 사용한 쿠키로 다른 스트림 열기, 이미 닫힌 스트림 닫기)을 견고하게 다루도록 개선할 수 있을 것이다.

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

#define INIT_BUF_SIZE 4

struct memfile_cookie {
    char   *buf;         /* 동적 크기의 데이터 버퍼 */
    size_t  allocated;   /* buf의 크기 */
    size_t  endpos;      /* buf 내의 문자 개수 */
    off_t   offset;      /* buf 내의 현재 파일 오프셋 */
};

ssize_t
memfile_write(void *c, const char *buf, size_t size)
{
    char *new_buff;
    struct memfile_cookie *cookie = c;

    /* 버퍼 부족? 충분히 커질 때까지 두 배씩 키우기 */

    while (size + cookie->offset > cookie->allocated) {
        new_buff = realloc(cookie->buf, cookie->allocated * 2);
        if (new_buff == NULL) {
            return -1;
        } else {
            cookie->allocated *= 2;
            cookie->buf = new_buff;
        }
    }

    memcpy(cookie->buf + cookie->offset, buf, size);

    cookie->offset += size;
    if (cookie->offset > cookie->endpos)
        cookie->endpos = cookie->offset;

    return size;
}

ssize_t
memfile_read(void *c, char *buf, size_t size)
{
    ssize_t xbytes;
    struct memfile_cookie *cookie = c;

    /* 요청 바이트와 가용 바이트 중 작은 쪽 가져오기 */

    xbytes = size;
    if (cookie->offset + size > cookie->endpos)
        xbytes = cookie->endpos - cookie->offset;
    if (xbytes < 0)     /* offset이 endpos를 지났을 수 있음 */
        xbytes = 0;

    memcpy(buf, cookie->buf + cookie->offset, xbytes);

    cookie->offset += xbytes;
    return xbytes;
}

int
memfile_seek(void *c, off64_t *offset, int whence)
{
    off64_t new_offset;
    struct memfile_cookie *cookie = c;

    if (whence == SEEK_SET)
        new_offset = *offset;
    else if (whence == SEEK_END)
        new_offset = cookie->endpos + *offset;
    else if (whence == SEEK_CUR)
        new_offset = cookie->offset + *offset;
    else
        return -1;

    if (new_offset < 0)
        return -1;

    cookie->offset = new_offset;
    *offset = new_offset;
    return 0;
}

int
memfile_close(void *c)
{
    struct memfile_cookie *cookie = c;

    free(cookie->buf);
    cookie->allocated = 0;
    cookie->buf = NULL;

    return 0;
}

int
main(int argc, char *argv[])
{
    cookie_io_functions_t  memfile_func = {
        .read  = memfile_read,
        .write = memfile_write,
        .seek  = memfile_seek,
        .close = memfile_close
    };
    FILE *stream;
    struct memfile_cookie mycookie;
    ssize_t nread;
    long p;
    int j;
    char buf[1000];

    /* 쿠키 준비하고 fopencookie() 호출 */

    mycookie.buf = malloc(INIT_BUF_SIZE);
    if (mycookie.buf == NULL) {
        perror("malloc");
        exit(EXIT_FAILURE);
    }

    mycookie.allocated = INIT_BUF_SIZE;
    mycookie.offset = 0;
    mycookie.endpos = 0;

    stream = fopencookie(&mycookie,"w+", memfile_func);
    if (stream == NULL) {
        perror("fopencookie");
        exit(EXIT_FAILURE);
    }

    /* 명령행 인자를 우리 파일로 기록 */

    for (j = 1; j < argc; j++)
        if (fputs(argv[j], stream) == EOF) {
            perror("fputs");
            exit(EXIT_FAILURE);
        }

    /* EOF 전까지 다섯 바이트마다 두 바이트씩 읽기 */

    for (p = 0; ; p += 5) {
        if (fseek(stream, p, SEEK_SET) == -1) {
            perror("fseek");
            exit(EXIT_FAILURE);
        }
        nread = fread(buf, 1, 2, stream);
        if (nread == -1) {
            perror("fread");
            exit(EXIT_FAILURE);
        }
        if (nread == 0) {
            printf("Reached end of file\n");
            break;
        }

        printf("/%.*s/\n", nread, buf);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[fclose(3)]]</tt>, <tt>[[fmemopen(3)]]</tt>, <tt>[[fopen(3)]]</tt>, `fseek(3)`

----

2019-03-06
