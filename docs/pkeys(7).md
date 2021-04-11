## NAME

pkeys - 메모리 보호 키 개요

## DESCRIPTION

메모리 보호 키(Memory Protection Key, pkey)는 기존의 페이지 기반 메모리 접근 허가에 대한 확장이다. 페이지 테이블을 이용한 일반적인 페이지 접근 허가에서는 허가 변경 시 값비싼 시스템 호출과 TLB 무효화가 필요하다. 메모리 보호 키는 허가 변경 때마다 페이지 테이블을 변경할 필요 없이 보호 변경이 가능한 메커니즘을 제공한다.

pkey를 사용하려면 먼저 소프트웨어에서 페이지 테이블 내의 페이지에 pkey "태그"를 붙여야 한다. 그 후에 응용에서 태그 붙은 페이지에 대한 쓰기 접근권이나 전체 접근권을 제거하려면 레지스터의 내용만 바꾸면 된다.

보호 키는 <tt>[[mprotect(2)]]</tt>와 <tt>[[mmap(2)]]</tt> 같은 시스템 호출에 전달한 기존의 `PROT_READ`/`PROT_WRITE`/`PROT_EXEC` 허가와 결합돼서 동작한다. 언제나 그 전통적 허가 메커니즘을 더 제약하는 방식으로 동작한다.

어느 프로세스에서 키 제약을 위반하는 접근을 수행하면 `SIGSEGV` 시그널을 받는다. 그 시그널에서 얻을 수 있는 정보에 대한 자세한 내용은 <tt>[[sigaction(2)]]</tt>에 있다.

pkey 기능을 쓰려면 프로세서에서 이를 지원해야 하고 해당 프로세서에서의 기능 지원이 커널에 포함되어 있어야 한다. 2016년 초 현재 향후의 인텔 x86 프로세스들만 지원하며, 그 하드웨어에서는 프로세스별로 16개 보호 키를 지원한다. 하지만 pkey 0은 기본 키로 쓰이기 때문에 실제 응용 용도로는 최대 15개가 사용 가능하다. <tt>[[pkey_mprotect(2)]]</tt>를 통해 명시적으로 pkey를 부여하지 않은 모든 메모리 영역에 기본 키가 부여된다.

보호 키에는 응용에 보안 및 신뢰성 계층을 더해 줄 잠재력이 있다. 하지만 기본적으로는 보안 기능으로 설계된 것이 아니다. 예를 들어 WRPKRU는 완전한 비특권 인스트럭션이고, 따라서 공격자가 PKRU 레지스터를 통제하고 있거나 임의 인스트럭션을 실행할 수 있는 경우에는 pkey가 소용없다.

응용에서는 보호 키를 "누출"하지 않도록 아주 조심해야 한다. 예를 들어 응용에서 <tt>[[pkey_free(2)]]</tt> 호출 전에는 어떤 메모리에도 그 pkey가 할당되어 있지 않도록 해야 한다. 응용에서 해제된 그 pkey를 할당된 채로 두면 향후 그 pkey의 사용자가 의도치 않게 상관없는 자료 구조의 접근 허가를 바꾸게 될 수도 있을 테고, 그게 보안성이나 안정성에 영향을 줄 수도 있다. 커널에서는 현재 사용 중인 pkey에 <tt>[[pkey_free(2)]]</tt> 호출을 하는 것을 허용하고 있는데, 그걸 막기 위해 추가 검사를 수행하는 것이 프로세스 내지 메모리 성능에 영향을 줄 것이기 때문이다. 필요한 검사를 구현하는 것은 응용의 몫이다. `/proc/[pid]/smaps` 파일에서 pkey가 할당된 메모리 영역을 탐색하는 방식으로 응용에서 그런 검사를 구현할 수 있다.

보호 키를 사용하고자 하는 응용은 보호 키 없이도 동작할 수 있어야 한다. 응용이 동작하는 하드웨어에서 지원하지 않거나, 커널 코드에 지원이 포함되어 있지 않거나, 커널 지원이 꺼져 있거나, 키가 모두 (경우에 따라선 응용에서 사용하는 라이브러리에 의해) 할당되어서 사용이 불가능할 수도 있을 것이다. 보호 키를 사용하고자 하는 응용에서 기능 지원을 탐지하기 위해 다른 방식을 시도하는 대신 그냥 <tt>[[pkey_alloc(2)]]</tt>을 호출해 보기를 권장한다.

불필요한 일이긴 하지만 하드웨어의 보호 키 지원을 `cpuid` 인스트럭션으로 확인해 볼 수도 있다. 자세한 방법은 Intel Software Developers Manual에서 찾을 수 있다. 커널에서 그 확인을 해서 `/proc/cpuinfo` 내의 "flags" 필드로 정보를 드러낸다. 이 필드에 문자열 "pku"가 있으면 보호 키를 하드웨어에서 지원한다는 표시이고 문자열 "ospke"가 있으면 커널에 보호 키 지원이 포함되어 있고 켜져 있다는 표시이다.

스레드와 보호 키를 사용하는 응용에서는 특히 조심해야 할 것이다. 스레드는 <tt>[[clone(2)]]</tt> 시스템 호출 시점에 부모로부터 보호 키 권한을 물려받는다. 응용에서는 <tt>[[clone(2)]]</tt> 호출 시점에 자기 권한이 자식 스레드에게 적절하도록 하거나, 아니면 각 자식 스레드에서 자체적으로 보호 키 권한 초기화를 수행할 수 있도록 해야 할 것이다.

### 시그널 핸들러 동작 방식

시그널 핸들러가 불릴 때마다 (중첩 시그널 포함) 중단된 문맥의 권한을 덮는 새로운 기본 보호 키 권한 세트가 스레드에게 임시로 주어진다. 즉 응용에서 원하는 권한이 기본 세트와 다르다면 시그널 핸들러 진입 직후에 원하는 보호 키 권한을 재설정해야 한다. 시그널 핸들러가 반환할 때 중단됐던 문맥의 권한이 복원된다.

이런 특이한 시그널 동작 방식은 (보호 키 접근권을 저장하는) x86 PKRU 레지스터를 부동소수점 레지스터와 같은 하드웨어 메커니즘(XSAVE)으로 관리하기 때문이다. 이 시그널 동작 방식은 부동소수점 레지스터와 동일하다.

### 보호 키 시스템 호출

리눅스 커널에서 구현하고 있는 pkey 관련 시스템 호출은 <tt>[[pkey_mprotect(2)]]</tt>, <tt>[[pkey_alloc(2)]]</tt>, <tt>[[pkey_free(2)]]</tt>이다.

리눅스의 pkey 시스템 호출은 `CONFIG_X86_INTEL_MEMORY_PROTECTION_KEYS` 옵션으로 커널을 구성하여 빌드 한 경우에만 사용 가능하다.

## EXAMPLE

아래 프로그램에서는 메모리 한 페이지를 읽기 및 쓰기 허가로 할당한다. 그러고서 메모리에 어떤 데이터를 써넣고 성공적으로 읽어들인다. 그 후에 보호 키를 할당하여 WRPKRU 인스트럭션으로 그 페이지에 대한 접근을 불허해 본다. 그러고서 페이지에 접근을 시도하는데, 이번에는 응용에 치명적 시그널을 일으키게 된다.

```text
$ ./a.out
buffer contains: 73
about to read buffer again...
Segmentation fault (core dumped)
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <unistd.h>
#include <sys/syscall.h>
#include <stdio.h>
#include <sys/mman.h>

static inline void
wrpkru(unsigned int pkru)
{
    unsigned int eax = pkru;
    unsigned int ecx = 0;
    unsigned int edx = 0;

    asm volatile(".byte 0x0f,0x01,0xef\n\t"
                 : : "a" (eax), "c" (ecx), "d" (edx));
}

int
pkey_set(int pkey, unsigned long rights, unsigned long flags)
{
    unsigned int pkru = (rights << (2 * pkey));
    return wrpkru(pkru);
}

int
pkey_mprotect(void *ptr, size_t size, unsigned long orig_prot,
              unsigned long pkey)
{
    return syscall(SYS_pkey_mprotect, ptr, size, orig_prot, pkey);
}

int
pkey_alloc(void)
{
    return syscall(SYS_pkey_alloc, 0, 0);
}

int
pkey_free(unsigned long pkey)
{
    return syscall(SYS_pkey_free, pkey);
}

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                           } while (0)

int
main(void)
{
    int status;
    int pkey;
    int *buffer;

    /*
     * 메모리 페이지 한 개 할당
     */
    buffer = mmap(NULL, getpagesize(), PROT_READ | PROT_WRITE,
                  MAP_ANONYMOUS | MAP_PRIVATE, -1, 0);
    if (buffer == MAP_FAILED)
        errExit("mmap");

    /*
     * 페이지에 적당한 임의 데이터 넣기 (아직 건드려도 괜찮음)
     */
    *buffer = __LINE__;
    printf("buffer contains: %d\n", *buffer);

    /*
     * 보호 키 할당
     */
    pkey = pkey_alloc();
    if (pkey == -1)
        errExit("pkey_alloc");

    /*
     * "pkey"가 설정된 메모리(현재는 없음)에 대한 접근 끄기
     */
    status = pkey_set(pkey, PKEY_DISABLE_ACCESS, 0);
    if (status)
        errExit("pkey_set");

    /*
     * "buffer"에 보호 키 설정.
     * mprotect()와 관련해선 여전히 읽기/쓰기이며
     * 앞의 pkey_set()이 이를 오버라이드 한다.
     */
    status = pkey_mprotect(buffer, getpagesize(),
                           PROT_READ | PROT_WRITE, pkey);
    if (status == -1)
        errExit("pkey_mprotect");

    printf("about to read buffer again...\n");

    /*
     * 접근을 불허했으므로 죽게 됨
     */
    printf("buffer contains: %d\n", *buffer);

    status = pkey_free(pkey);
    if (status == -1)
        errExit("pkey_free");

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[pkey_alloc(2)]]</tt>, <tt>[[pkey_free(2)]]</tt>, <tt>[[pkey_mprotect(2)]]</tt>, <tt>[[sigaction(2)]]</tt>

----

2019-03-06
