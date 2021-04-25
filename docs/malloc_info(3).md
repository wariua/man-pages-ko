## NAME

malloc_info - malloc 상태를 스트림으로 내보내기

## SYNOPSIS

```c
#include <malloc.h>

int malloc_info(int options, FILE *stream);
```

## DESCRIPTION

`malloc_info()` 함수는 호출자 내의 메모리 할당 구현의 현재 상태를 기술하는 XML 문자열을 내보낸다. 파일 스트림 `stream`으로 그 문자열을 찍는다. 내보내는 문자열에는 모든 아레나에 대한 정보가 포함되어 있다. (<tt>[[malloc(3)]]</tt> 참고.)

현재 구현 기준으로 `options`는 0이어야 한다.

## RETURN VALUE

성공 시 `malloc_info()`는 0을 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `options`가 0이 아니다.

## VERSIONS

glibc 버전 2.10에서 `malloc_info()`가 추가되었다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `malloc_info()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 GNU 확장이다.

## NOTES

메모리 할당 정보를 (C 구조체가 아니라) XML 문자열로 제공하는 이유는 시간이 흐르면서 (기반 구현의 변화에 따라) 정보가 바뀔 수 있기 때문이다. 출력 XML 문자열에 버전 필드가 들어 있다.

<tt>[[open_memstream(3)]]</tt> 함수를 쓰면 `malloc_info()` 출력을 파일이 아니라 메모리 내 버퍼로 바로 보낼 수 있다.

`malloc_info()` 함수는 <tt>[[malloc_stats(3)]]</tt>와 <tt>[[mallinfo(3)]]</tt>의 결점들을 해결하도록 설계되었다.

## EXAMPLES

아래 프로그램은 4개까지의 명령행 인자를 받으며, 그 중 처음 3개는 필수이다. 첫 번째 인자는 프로그램이 생성해야 하는 스레드 수를 지정한다. 메인 스레드를 포함한 모든 스레드에서 두 번째 인자로 지정한 개수의 메모리 블록을 할당한다. 세 번째 인자는 할당할 블록의 크기를 제어한다. 메인 스레드는 이 크기의 블록을 만들고, 두 번째 스레드는 두 배 크기 블록을 할당하고, 세 번째 스레드는 세 배 크기 블록을 할당하고 하는 식이다.

프로그램에서 `malloc_info()`를 두 번 호출해서 메모리 할당 상태를 표시한다. 첫 번째 호출은 스레드 생성이나 메모리 할당을 하기 전에 한다. 두 번째 호출은 모든 스레드가 메모리를 할당한 후에 수행한다.

다음 예의 명령행 인자는 추가 스레드를 1개 만들고 메인 스레드와 그 추가 스레드 모두 메모리 블록 10000개를 할당하라는 뜻이다. 그 메모리 블록들을 할당한 후에 `malloc_info()`로 두 할당 아레나의 상태를 보인다.

```text
$ getconf GNU_LIBC_VERSION
glibc 2.13
$ ./a.out 1 10000 100
============ Before allocating blocks ============
<malloc version="1">
<heap nr="0">
<sizes>
</sizes>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="135168"/>
<system type="max" size="135168"/>
<aspace type="total" size="135168"/>
<aspace type="mprotect" size="135168"/>
</heap>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="135168"/>
<system type="max" size="135168"/>
<aspace type="total" size="135168"/>
<aspace type="mprotect" size="135168"/>
</malloc>

============ After allocating blocks ============
<malloc version="1">
<heap nr="0">
<sizes>
</sizes>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="1081344"/>
<system type="max" size="1081344"/>
<aspace type="total" size="1081344"/>
<aspace type="mprotect" size="1081344"/>
</heap>
<heap nr="1">
<sizes>
</sizes>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="1032192"/>
<system type="max" size="1032192"/>
<aspace type="total" size="1032192"/>
<aspace type="mprotect" size="1032192"/>
</heap>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="2113536"/>
<system type="max" size="2113536"/>
<aspace type="total" size="2113536"/>
<aspace type="mprotect" size="2113536"/>
</malloc>
```

### 프로그램 소스

```c
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>
#include <malloc.h>
#include <errno.h>

static size_t blockSize;
static int numThreads, numBlocks;

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

static void *
thread_func(void *arg)
{
    int tn = (int) arg;

    /* '(2 + tn)'을 곱해 주면 각 스레드(메인 스레드 포함)가
       각기 다른 양의 메모리를 할당하게 된다. */

    for (int j = 0; j < numBlocks; j++)
        if (malloc(blockSize * (2 + tn)) == NULL)
            errExit("malloc-thread");

    sleep(100);         /* 메인 스레드가 종료할 때까지 잠들기. */
    return NULL;
}

int
main(int argc, char *argv[])
{
    int sleepTime;

    if (argc < 4) {
        fprintf(stderr,
                "%s num-threads num-blocks block-size [sleep-time]\n",
                argv[0]);
        exit(EXIT_FAILURE);
    }

    numThreads = atoi(argv[1]);
    numBlocks = atoi(argv[2]);
    blockSize = atoi(argv[3]);
    sleepTime = (argc > 4) ? atoi(argv[4]) : 0;

    pthread_t *thr = calloc(numThreads, sizeof(*thr));
    if (thr == NULL)
        errExit("calloc");

    printf("============ Before allocating blocks ============\n");
    malloc_info(0, stdout);

    /* 각기 다른 양의 메모리를 할당하는 스레드 생성하기. */

    for (int tn = 0; tn < numThreads; tn++) {
        errno = pthread_create(&thr[tn], NULL, thread_func,
                               (void *) tn);
        if (errno != 0)
            errExit("pthread_create");

        /* 스레드 시작 후마다 잠자는 간격을 추가하면 스레드들이
	   malloc 뮤텍스를 두고 경쟁하지 않게 되고, 그래서 아레나를
	   추가로 할당하지 않게 됨. (malloc(3) 참고.) */

        if (sleepTime > 0)
            sleep(sleepTime);
    }

    /* 메인 스레드에서도 메모리를 좀 할당한다. */

    for (int j = 0; j < numBlocks; j++)
        if (malloc(blockSize) == NULL)
            errExit("malloc");

    sleep(2);           /* 모든 스레드가 할당을 마칠 시간을 준다. */

    printf("\n============ After allocating blocks ============\n");
    malloc_info(0, stdout);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[mallinfo(3)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[malloc_stats(3)]]</tt>, <tt>[[mallopt(3)]]</tt>, <tt>[[open_memstream(3)]]</tt>

----

2021-03-22
