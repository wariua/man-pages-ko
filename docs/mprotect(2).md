## NAME

mprotect, pkey_mprotect - 메모리 영역에 보호 설정하기

## SYNOPSIS

```c
#include <sys/mman.h>

int mprotect(void *addr, size_t len, int prot);

#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <sys/mman.h>

int pkey_mprotect(void *addr, size_t len, int prot, int pkey);
```

## DESCRIPTION

`mprotect()`는 [`addr`, `addr+len-1`] 구간 주소 범위를 일부라도 담고 있는 호출 프로세스의 메모리 페이지들에 대한 접근 보호를 변경한다. `addr`이 페이지 경계에 맞게 정렬되어 있어야 한다.

호출 프로세스에서 그 보호를 위반하는 방식으로 메모리에 접근하려 시도하면 커널이 그 프로세스에 `SIGSEGV` 시그널을 생성한다.

`prot`는 `PROT_NONE`이거나 다음 목록의 다른 값들을 비트 OR 한 것이다.

`PROT_NONE`
:   메모리에 전혀 접근할 수 없다.

`PROT_READ`
:   메모리를 읽을 수 있다.

`PROT_WRITE`
:   메모리를 변경할 수 있다.

`PROT_EXEC`
:   메모리를 실행할 수 있다.

`PROT_SEM` (리눅스 2.5.7부터)
:   메모리를 원자 연산에 쓸 수 있다. 이 플래그는 <tt>[[futex(2)]]</tt> 구현의 일부로서 (`FUTEX_WAIT` 같은 명령에 필요한 원자 연산 수행이 가능함을 보장하기 위해) 도입되었는데 현재는 어느 아키텍처에서도 쓰지 않는다.

`PROT_SAO` (리눅스 2.6.26부터)
:   메모리에 강한 접근 순서(strong access ordering)가 있어야 한다. 이 기능은 PowerPC 아키텍처에 한정된 것이다. (아키텍처 명세 2.06 버전에서 SAO CPU 기능을 추가하며 POWER 7이나 PowerPC A2 등에서 사용 가능하다.)

추가로 (리눅스 2.6.0부터) `prot`에 다음 플래그들 중 하나를 설정할 수 있다.

`PROT_GROWSUP`
:   위로 자라는 매핑의 끝점까지 보호 모드를 적용한다. (스택이 위로 자라는, 가령 HP-PARISC 같은 아키텍처에서 스택 영역에 그런 매핑을 만든다.)

`PROT_GROWSDOWN`
:   아래로 자라는 매핑의 시작점까지 보호 모드를 적용한다. (스택 세그먼트이거나 `MMAP_GROWSDOWN` 플래그를 설정해 맵 한 세그먼트일 것이다.)

`mprotect()`처럼 `pkey_mprotect()`는 `addr`과 `len`으로 지정한 페이지들의 보호를 변경한다. `pkey` 인자는 그 메모리에 할당한 보호 키(<tt>[[pkeys(7)]]</tt> 참고)를 나타낸다. 보호 키는 <tt>[[pkey_alloc(2)]]</tt>으로 할당해서 `pkey_mprotect()`에게 전달해야 한다. 이 시스템 호출의 사용례는 <tt>[[pkeys(7)]]</tt>를 보라.

## RETURN VALUE

성공 시 `mprotect()`와 `pkey_mprotect()`는 0을 반환한다. 오류 시 이 시스템 호출들은 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   지정한 접근권을 메모리에 줄 수 없다. 예를 들어 읽기 전용으로만 접근할 수 있는 파일을 <tt>[[mmap(2)]]</tt> 하고서 `mprotect()`에게 `PROT_WRITE` 표시를 하라고 하는 경우에 발생할 수 있다.

`EINVAL`
:   `addr`이 유효한 포인터가 아니거나 시스템 페이지 크기의 배수가 아니다.

`EINVAL`
:   (`pkey_mprotect()`) `pkey`가 <tt>[[pkey_alloc(2)]]</tt>으로 할당된 것이 아니다.

`EINVAL`
:   `prot`에 `PROT_GROWSUP`과 `PROT_GROWSDOWN`을 모두 지정했다.

`EINVAL`
:   `prot`에 유효하지 않은 플래그를 지정했다.

`EINVAL`
:   (PowerPC 아키텍처) `prot`에 `PROT_SAO`를 지정했지만 SAO 하드웨어 기능이 사용 가능하지 않다.

`ENOMEM`
:   내부 커널 구조체를 할당할 수 없다.

`ENOMEM`
:   [`addr`, `addr+len-1`] 구간 내의 주소가 프로세스의 주소 공간에서 유효하지 않거나, 한 개 이상의 맵 되지 않은 페이지를 나타낸다. (커널 2.4.19 전에서는 이 경우에 잘못해서 `EFAULT` 오류를 생성했다.)

`ENOMEM`
:   메모리 영역의 보호를 변경하면 상이한 속성(가령 읽기 보호와 읽기/쓰기 보호)의 매핑 총개수가 허용 최대치를 초과하게 된다. (예를 들어 현재 `PROT_READ|PROT_WRITE`로 보호하는 영역 중간의 어느 범위를 `PROT_READ` 보호로 만들면 양쪽의 읽기/쓰기 매핑과 가운데의 읽기 전용 매핑을 합쳐서 세 개 매핑이 생긴다.)

## VERSIONS

리눅스 4.9에서 `pkey_mprotect()`가 처음 등장했다. glibc 2.27에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`mprotect()`: POSIX.1-2001, POSIX.1-2008, SVr4. POSIX에서는 <tt>[[mmap(2)]]</tt>을 통해 얻은 것이 아닌 메모리 영역에 적용 시 `mprotect()`의 동작 방식이 명세되어 있지 않다고 한다.

`pkey_mprotect()`는 이식성 없는 리눅스 확장이다.

## NOTES

리눅스에서는 프로세스 주소 공간 내의 (커널 vsyscall 영역을 제외한) 어느 주소에도 `mprotect()` 호출을 항상 허용한다. 특히 이를 이용해 기존의 코드 매핑을 쓰기 가능으로 바꿀 수 있다.

`PROT_EXEC`에 `PROT_READ`와는 다른 어떤 효력이 있는지 여부는 프로세서 아키텍처, 커널 버전, 프로세스 상태에 따라 정해진다. 프로세스 인격 플래그(<tt>[[personality(2)]]</tt> 참고)에 `READ_IMPLIES_EXEC`가 설정돼 있으면 `PROT_READ`를 지정할 때 `PROT_EXEC`가 암묵적으로 추가된다.

일부 하드웨어 아키텍처(가령 i386)에서는 `PROT_WRITE`가 `PROT_READ`를 함의한다.

POSIX.1에서는 구현에서 `prot`에 지정된 것 이외의 접근을 허용할 수도 있되, 최소한으로는 `PROT_WRITE`가 설정된 경우에만 쓰기 접근을 허용할 수 있고 `PROT_NONE`이 설정된 경우에는 어떤 접근도 허용해선 안 된다고 한다.

응용에서 `mprotect()`와 `pkey_mprotect()`를 섞어서 사용할 때는 주의해야 한다. x86에서 `prot`를 `PROT_EXEC`로 설정해서 `mprotect()`를 쓰면 커널에서 암묵적으로 pkey를 할당해서 메모리에 설정할 수도 있다. 단 pkey가 앞서 0이었을 때만 그렇게 한다.

하드웨어에서 보호 키를 지원하지 않는 시스템에서도 `pkey_mprotect()`를 쓸 수는 있지만 `pkey`를 -1로 설정해야 한다. 그렇게 호출할 때 `pkey_mprotect()`의 동작은 `mprotect()`와 동등하다.

## EXAMPLES

아래 프로그램은 `mprotect()` 사용 방식을 보여 준다. 프로그램에서 메모리 페이지 4개를 할당해서 그 중 세 번째를 읽기 전용으로 만든 다음 할당 영역을 위로 훑으면서 바이트를 변경하는 루프를 실행한다.

다음은 프로그램 실행 시 볼 수 있을 출력 예이다.

```text
$ ./a.out
Start of region:        0x804c000
Got SIGSEGV at address: 0x804e000
```

### 프로그램 소스

```c
#include <unistd.h>
#include <signal.h>
#include <stdio.h>
#include <malloc.h>
#include <stdlib.h>
#include <errno.h>
#include <sys/mman.h>

#define handle_error(msg) \
    do { perror(msg); exit(EXIT_FAILURE); } while (0)

static char *buffer;

static void
handler(int sig, siginfo_t *si, void *unused)
{
    /* 주의: 시그널 핸들러에서 printf()를 호출하는 것은 안전하지
       않다. (따라서 실제 사용하는 프로그램에서는 하지 말아야
       한다.) printf()가 비동기 시그널 안전이 아니기 때문이다.
       signal-safety(7) 참고. 그럼에도 불구하고 핸들러가 호출된
       것을 간단히 보여 주기 위해 여기에 printf()를 사용한다. */

    printf("Got SIGSEGV at address: %p\n", si->si_addr);
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int pagesize;
    struct sigaction sa;

    sa.sa_flags = SA_SIGINFO;
    sigemptyset(&sa.sa_mask);
    sa.sa_sigaction = handler;
    if (sigaction(SIGSEGV, &sa, NULL) == -1)
        handle_error("sigaction");

    pagesize = sysconf(_SC_PAGE_SIZE);
    if (pagesize == -1)
        handle_error("sysconf");

    /* 페이지 경계에 맞게 정렬된 버퍼 할당.
       초기 보호 방식은 PROT_READ | PROT_WRITE. */

    buffer = memalign(pagesize, 4 * pagesize);
    if (buffer == NULL)
        handle_error("memalign");

    printf("Start of region:        %p\n", buffer);

    if (mprotect(buffer + pagesize * 2, pagesize,
                PROT_READ) == -1)
        handle_error("mprotect");

    for (char *p = buffer ; ; )
        *(p++) = 'a';

    printf("Loop completed\n");     /* 절대 여기까지 오지 않음 */
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[mmap(2)]]</tt>, <tt>[[sysconf(3)]]</tt>, <tt>[[pkeys(7)]]</tt>

----

2021-03-22
