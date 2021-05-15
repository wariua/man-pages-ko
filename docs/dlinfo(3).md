## NAME

dlinfo - 동적 적재 오브젝트에 대한 정보 얻기

## SYNOPSIS

```c
#define _GNU_SOURCE
#include <link.h>
#include <dlfcn.h>

int dlinfo(void *restrict handle, int request, void *restrict info);
```

`-ldl`로 링크.

## DESCRIPTION

`dlinfo()` 함수는 (보통 앞선 <tt>[[dlopen(3)]]</tt> 내지 <tt>[[dlmopen(3)]]</tt> 호출로 얻은) `handle`이 가리키는 동적 적재 오브젝트에 대한 정보를 얻어 온다. `request` 인자는 어떤 정보를 반환해야 하는지 지정한다. `info` 인자는 호출에서 반환하는 정보를 저장하는 데 쓰는 버퍼의 포인터다. `request`에 따라 이 인자의 타입이 달라진다.

`request`에 다음 값들을 지원한다. (괄호 안은 `info`의 타입이다.)

`RTLD_DI_LMID` (`Lmid_t *`)
:   `handle`이 적재된 링크 맵 목록(네임스페이스)의 ID를 얻는다.

`RTLD_DI_LINKMAP` (`struct link_map **`)
:   `handle`에 대응하는 `link_map` 구조체 포인터를 얻는다. `info` 인자가 `<link.h>`에 다음처럼 정의된 `link_map` 구조체 포인터를 가리킨다.

        struct link_map {
            ElfW(Addr) l_addr;  /* ELF 파일 내 주소와
                                   메모리 내 주소의 차이 */
            char      *l_name;  /* 오브젝트가 있던
                                   절대 경로명 */
            ElfW(Dyn) *l_ld;    /* 공유 오브젝트의
                                   dynamic 섹션 */
            struct link_map *l_next, *l_prev;
                                /* 적재된 오브젝트들을 연결 */

            /* 추가로 구현 내부용 필드들이 있음 */
        };

`RTLD_DI_ORIGIN` (`char *`)
:   `handle`에 대응하는 공유 오브젝트의 출처(origin) 경로명을 `info`가 가리키는 위치로 복사한다.

`RTLD_DI_SERINFO` (`Dl_serinfo *`)
:   `handle`이 가리키는 공유 오브젝트에 대한 라이브러리 탐색 경로 목록을 얻는다. `info` 인자가 탐색 경로들을 담는 `Dl_serinfo`의 포인터다. 탐색 경로 수가 다를 수 있기 때문에 `info`가 가리키는 구조체의 크기가 달라질 수 있다. 아래에서 기술하는 `RTLD_DI_SERINFOSIZE` 요청을 이용하면 응용에서 적절한 크기로 버퍼를 만들 수 있다. 호출자가 다음 단계들을 수행해야 한다.

    1. `RTLD_DI_SERINFOSIZE` 요청을 사용해 이어질 `RTLD_DI_SERINFO` 요청에 필요한 크기(`dls_size`)를 `DL_serinfo` 구조체에 채운다.

    2. 올바른 크기(`dls_size`)로 `Dl_serinfo` 버퍼를 할당한다.

    3. 다시 `RTLD_DI_SERINFOSIZE` 요청을 사용해 앞 단계에서 할당한 버퍼의 `dls_size` 및 `dls_cnt` 필드를 채운다.

    4. `RTLD_DI_SERINFO`를 사용해 라이브러리 탐색 경로들을 얻는다.

    `Dl_serinfo` 구조체는 다음과 같이 정의돼 있다.

        typedef struct {
            size_t dls_size;           /* 전체 버퍼의
                                          바이트 단위 크기 */
            unsigned int dls_cnt;      /* 'dls_serpath'의
                                          항목 수 */
            Dl_serpath dls_serpath[1]; /* 실제로는 항목
                                          'dls_cnt' 개 */
        } Dl_serinfo;

    위 구조체의 `dls_serpath` 항목 각각은 다음 형태의 구조체다.

        typedef struct {
            char *dls_name;            /* 라이브러리 탐색 경로
                                          디렉터리의 이름 */
            unsigned int dls_flags;    /* 이 디렉터리가 어디서
                                          왔는지 나타냄 */
        } Dl_serpath;

    `dls_flags` 필드는 현재 쓰지 않으며 항상 0을 담는다.

`RTLD_DI_SERINFOSIZE` (`Dl_serinfo *`)
:   `info`가 가리키는 `Dl_serinfo` 구조체의 `dls_size` 및 `dls_cnt` 필드를 이어질 `RTLD_DI_SERINFO` 요청에 쓸 버퍼 할당에 적합한 값들로 채운다.

`RTLD_DI_TLS_MODID` (`size_t *`, glibc 2.4부터)
:   이 공유 오브젝트의 TLS(스레드 로컬 저장 공간) 세그먼트의 TLS 재배치에 사용된 모듈 ID를 얻는다. 오브젝트에 TLS 세그먼트가 정의돼 있지 않으면 `*info`에 0이 들어간다.

`RTLD_DI_TLS_DATA` (`void **`, glibc 2.4부터)
:   이 공유 오브젝트의 TLS 세그먼트에 대응하는 호출 스레드의 TLS 블록에 대한 포인터를 얻는다. 오브젝트에 PT_TLS 세그먼트가 정의돼 있지 않거나 호출 스레드에서 블록을 할당하지 않았으면 `*info`에 NULL이 들어간다.

## RETURN VALUE

성공 시 `dlinfo()`는 0을 반환한다. 실패 시 -1을 반환한다. <tt>[[dlerror(3)]]</tt>로 오류 원인을 진단할 수 있다.

## VERSIONS

glibc 2.3.3에서 `dlinfo()`가 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `dlinfo()` | 스레드 안전성 | MT-Safe |

## NOTES

이 함수는 같은 이름의 솔라리스 함수에서 유래한 것이며 몇몇 다른 시스템에도 있다. 다양한 구현들에서 지원하는 요청 세트들은 일부만 서로 겹친다.

## EXAMPLES

아래 프로그램에서는 <tt>[[dlopen(3)]]</tt>을 써서 공유 오브젝트를 연 다음 `RTLD_DI_SERINFOSIZE` 및 `RTLD_DI_SERINFO` 요청을 이용해 그 라이브러리에 대한 라이브러리 탐색 경로 목록을 얻는다. 다음은 프로그램 실행 시 볼 수 있는 출력의 예다.

```text
$ ./a.out /lib64/libm.so.6
dls_serpath[0].dls_name = /lib64
dls_serpath[1].dls_name = /usr/lib64
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <dlfcn.h>
#include <link.h>
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
    void *handle;
    Dl_serinfo serinfo;
    Dl_serinfo *sip;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s <libpath>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    /* 명령행에서 지정한 공유 오브젝트에 대한 핸들 얻기. */

    handle = dlopen(argv[1], RTLD_NOW);
    if (handle == NULL) {
        fprintf(stderr, "dlopen() failed: %s\n", dlerror());
        exit(EXIT_FAILURE);
    }

    /* RTLD_DI_SERINFO에 줘야 하는 버퍼 크기 알아내기. */

    if (dlinfo(handle, RTLD_DI_SERINFOSIZE, &serinfo) == -1) {
        fprintf(stderr, "RTLD_DI_SERINFOSIZE failed: %s\n", dlerror());
        exit(EXIT_FAILURE);
    }

    /* RTLD_DI_SERINFO에 쓸 버퍼 할당하기. */

    sip = malloc(serinfo.dls_size);
    if (sip == NULL) {
        perror("malloc");
        exit(EXIT_FAILURE);
    }

    /* 새로 할당된 버퍼의 'dls_size' 및 'dls_cnt' 필드 설정하기. */

    if (dlinfo(handle, RTLD_DI_SERINFOSIZE, sip) == -1) {
        fprintf(stderr, "RTLD_DI_SERINFOSIZE failed: %s\n", dlerror());
        exit(EXIT_FAILURE);
    }

    /* 라이브러리 탐색 경로 목록 얻어 와서 찍기. */

    if (dlinfo(handle, RTLD_DI_SERINFO, sip) == -1) {
        fprintf(stderr, "RTLD_DI_SERINFO failed: %s\n", dlerror());
        exit(EXIT_FAILURE);
    }

    for (int j = 0; j < serinfo.dls_cnt; j++)
        printf("dls_serpath[%d].dls_name = %s\n",
                j, sip->dls_serpath[j].dls_name);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[dl_iterate_phdr(3)]]</tt>, <tt>[[dladdr(3)]]</tt>, <tt>[[dlerror(3)]]</tt>, <tt>[[dlopen(3)]]</tt>, <tt>[[dlsym(3)]]</tt>, <tt>[[ld.so(8)]]</tt>

----

2021-03-22
