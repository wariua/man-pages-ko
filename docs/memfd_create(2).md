## NAME

memfd_create - 익명 파일 만들기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <sys/mman.h>

int memfd_create(const char *name, unsigned int flags);
```

## DESCRIPTION

`memfd_create()`는 익명 파일을 생성해서 그 파일을 가리키는 파일 디스크립터를 반환한다. 그 파일은 일반 파일처럼 동작하므로 변경하거나 잘라내거나 메모리에 맵 하거나 할 수 있다. 하지만 일반 파일과 달리 램 안에만 있으므로 기반 저장소가 휘발성이다. 파일에 대한 참조가 모두 없어지면 자동으로 해제된다. 그 파일의 기반 페이지 모두에 익명 메모리가 쓰인다. 따라서 `memfd_create()`로 만든 파일은 `MAP_ANONYMOUS` 플래그로 <tt>[[mmap(2)]]</tt>을 사용해 할당한 것 같은 다른 익명 메모리 할당과 동작 방식이 같다.

처음에 파일의 크기는 0으로 설정돼 있다. 호출 후에 <tt>[[ftruncate(2)]]</tt>를 사용해 파일 크기를 설정해 주어야 할 것이다. (또는 `write(2)` 등을 호출해서 파일을 채울 수도 있다.)

`name`에 준 이름은 파일명으로 쓰며 `/proc/self/fd/` 디렉터리의 대응하는 심볼릭 링크의 대상으로 표시된다. 표시되는 이름 앞에 항상 `memfd:`가 붙으며 디버깅 용도로 쓰일 뿐이다. 파일 디스크립터의 동작에 이름이 영향을 주지 않으며 그래서 아무 부작용 없이 여러 파일이 같은 이름을 가질 수 있다.

`flags`에 다음 값들을 비트 OR 해서 `memfd_create()`의 동작 방식을 바꿀 수 있다.

<dl>
<dt><code>MFD_CLOEXEC</code></dt>
<dd>새 파일 디스크립터에 'exec에서 닫기'(<code>FD_CLOEXEC</code>) 플래그를 설정한다. 이게 유용할 수 있는 이유에 대해선 <tt>[[open(2)]]</tt>의 <code>O_CLOEXEC</code> 플래그 설명을 보라.</dd>

<dt><code>MFD_ALLOW_SEALING</code></dt>
<dd>파일에 대한 봉인 동작을 허용한다. <tt>[[fcntl(2)]]</tt>의 <code>F_ADD_SEALS</code> 및 <code>F_GET_SEALS</code> 동작 논의와 아래 NOTES를 참고하라. 최초 봉인 집합은 비어 있다. 이 플래그가 설정돼 있지 않으면 최초 봉인 집합이 <code>F_SEAL_SEAL</code>이 된다. 즉 파일에 다른 봉인을 설정할 수 없다.</dd>

<dt><code>MFD_HUGETLB</code> (리눅스 4.14부터)</dt>
<dd>익명 파일이 거대 페이지를 이용해 hugetlbfs 파일 시스템 안에 만들어지게 된다. hugetlbfs에 대한 자세한 내용은 리눅스 커널 소스 파일 <code>Documentation/admin-guide/mm/hugetlbpage.rst</code>를 보라. <code>flags</code>에 <code>MFD_HUGETLB</code>와 <code>MFD_ALLOW_SEALING</code>을 함께 지정하는 것은 리눅스 4.16부터 지원한다.</dd>

<dt><code>MFD_HUGE_2MB</code>, <code>MFD_HUGE_1GB</code>, ...</dt>
<dd>

hugetlb 페이지 크기를 여러 가지 지원하는 시스템에서 <code>MFD_HUGETLB</code>와 조합해 사용해서 다른 hugetlb 페이지 크기(각각 2MB, 1GB, ...)를 선택한다. 알려진 거대 페이지 크기들의 정의가 헤더 파일 <code>&lt;linux/memfd.h&gt;</code>에 포함돼 있다.

헤더 파일에 포함돼 있지 않은 거대 페이지 크기를 인코딩 하는 자세한 방법에 대해선 <tt>[[mmap(2)]]</tt>의 비슷한 이름의 상수에 대한 설명을 보라.
</dd>
</dl>

`flags`의 안 쓰는 비트들은 0이어야 한다.

반환 값으로 `memfd_create()`가 반환하는 새 파일 디스크립터를 파일을 가리킬 데 쓸 수 있다. 그 파일 디스크립터는 읽기 및 쓰기(`O_RDWR`)로 열려 있으며 파일 디스크립터에 `O_LARGEFILE`이 설정돼 있다.

<tt>[[fork(2)]]</tt> 및 <tt>[[execve(2)]]</tt>와 관련해선 일반적인 의미론이 `memfd_create()`로 만든 파일 디스크립터에 적용된다. <tt>[[fork(2)]]</tt>로 생긴 자식이 파일 디스크립터 사본을 물려받으며 그 사본은 파일을 가리킨다. 'exec에서 닫기' 플래그를 설정하지 않았으면 <tt>[[execve(2)]]</tt>를 거치면서 파일 디스크립터가 유지된다.

## RETURN VALUE

성공 시 `memfd_create()`는 새 파일 디스크립터를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EFAULT</code></dt>
<dd><code>name</code>의 주소가 유효하지 않은 메모리를 가리키고 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>flags</code>에 알 수 없는 비트가 포함돼 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>name</code>이 너무 길다. (종료용 널 바이트를 제외하고 249바이트가 제한 길이이다.)</dd>
<dt><code>EINVAL</code></dt>
<dd><code>flags</code>에 <code>MFD_HUGETLB</code>와 <code>MFD_ALLOW_SEALING</code>을 함께 지정했다.</dd>
<dt><code>EMFILE</code></dt>
<dd>열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다.</dd>
<dt><code>ENFILE</code></dt>
<dd>열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>새 익명 파일을 생성하기에 메모리가 충분하지 않았다.</dd>
</dl>

## VERSIONS

리눅스 3.17에서 `memfd_create()` 시스템 호출이 처음 등장했다. glibc 버전 2.27에서 지원이 추가되었다.

## CONFORMING TO

`memfd_create()` 시스템 호출은 리눅스 전용이다.

## NOTES

`memfd_create()` 시스템 호출은 수동으로 <tt>[[tmpfs(5)]]</tt> 파일 시스템을 마운트 해서 거기서 파일을 여는 방식의 간단한 대안이 돼 준다. `memfd_create()`의 주된 목적은 <tt>[[fcntl(2)]]</tt>에서 제공하는 파일 봉인 API와 함께 사용할 파일 및 연계 파일 디스크립터를 만드는 것이다.

파일 봉인 없이도 `memfd_create()` 시스템 호출에 쓰임새가 있다. (그래서 `MFD_ALLOW_SEALING` 플래그로 명시적으로 요청하지 않으면 파일 봉인이 꺼져 있다.) 특히 결과 파일을 파일 시스템에 실제로 연결시킬 의도가 없는 경우에 `tmp`에 파일을 생성하는 것이나 <tt>[[open(2)]]</tt> `O_TMPFILE`를 사용하는 것의 대안으로 쓸 수 있다.

### 파일 봉인

파일 봉인이 없는 경우에는 공유 메모리를 통해 통신하는 프로세스들이 서로를 신뢰하든지, 아니면 신뢰할 수 없는 상대가 문제 있는 방식으로 공유 메모리 영역을 조작할 가능성에 대처해야 한다. 예를 들어 신뢰할 수 없는 상대가 아무 때나 공유 메모리의 내용물을 변경할 수도 있을 것이고, 공유 메모리 영역을 줄여 버릴 수도 있을 것이다. 앞쪽 가능성은 프로세스가 검사 시점 사용 시점(TOCTTOU) 경쟁 조건에 노출되게 놔둔다. (보통 공유 메모리 영역의 데이터를 복사한 다음에 검사 및 사용하는 것으로 대처한다.) 뒤쪽 가능성은 프로세스가 공유 메모리 영역의 더는 존재하지 않는 위치에 접근하려 할 때 `SIGBUS`를 받게 한다. (이 경우에 대처하려면 `SIGBUS` 시그널에 대한 핸들러 사용이 필요해진다.)

신뢰할 수 없는 상대를 다루려면 공유 메모리 사용 코드에 복잡도가 더해져야 한다. 메모리 봉인을 쓰면 상대가 바람직하지 않은 방식으로 공유 메모리를 조작할 수 없다고 안심하고 동작할 수 있으므로 그 추가 복잡도를 없앨 수 있다.

다음은 봉인 메커니즘 사용 예시이다.

1. 첫 번째 프로세스가 `memfd_create()`를 써서 <tt>[[tmpfs(5)]]</tt> 파일을 생성한다. 호출이 내놓는 파일 디스크립터를 이후 단계들에서 사용한다.

2. 첫 번째 프로세스가 앞서 생성한 파일 크기를 <tt>[[ftruncate(2)]]</tt>로 조정하고 <tt>[[mmap(2)]]</tt>으로 맵 한 다음 그 공유 메모리를 원하는 데이터로 채운다.

3. 첫 번째 프로세스가 파일에 대한 추가 변경을 제약하기 위해 <tt>[[fcntl(2)]]</tt> `F_ADD_SEALS` 동작을 이용해 파일에 한 개 이상의 봉인을 한다. (`F_SEAL_WRITE` 봉인을 하는 경우에는 앞 단계에서 만든 쓰기 가능한 공유 매핑을 먼저 해제해야 할 것이다.)

4. 두 번째 프로세스가 그 <tt>[[tmpfs(5)]]</tt> 파일에 대한 파일 디스크립터를 얻어서 맵 한다. 다음을 포함한 여러 방식으로 그렇게 할 수 있다.

   * `memfd_create()`를 호출한 프로세스가 결과로 나온 파일 디스크립터를 유닉스 도메인 소켓을 통해 두 번째 프로세스에게 전송할 수 있다. (<tt>[[unix(7)]]</tt> 및 <tt>[[cmsg(3)]]</tt> 참고.) 그러면 두 번째 프로세스가 <tt>[[mmap(2)]]</tt>으로 그 파일을 맵 한다.

   * 두 번째 프로세스가 <tt>[[fork(2)]]</tt>를 통해 생성되어 자동으로 파일 디스크립터와 매핑을 물려받는다. (참고로 이 경우와 다음 경우에서는 두 프로세스가 같은 사용자 ID 하에서 돌기 때문에 당연한 신뢰 관계가 있다. 따라서 보통은 파일 봉인이 필요하지 않다.)

   * 두 번째 프로세스가 `/proc/<pid>/fd/<fd>` 파일을 연다. 여기서 `<pid>`는 (`memfd_create()`을 호출한) 첫 번째 프로세스의 PID이고 `<fd>`는 그 프로세스의 `memfd_create()` 호출이 반환한 파일 디스크립터 번호이다. 그러고서 두 번째 프로세스가 <tt>[[mmap(2)]]</tt>으로 그 파일을 맵 한다.

5. 두 번째 프로세스가 <tt>[[fcntl(2)]]</tt> `F_GET_SEALS` 동작을 써서 그 파일에 적용된 봉인들의 비트 마스크를 가져온다. 이 비트 마스크를 살펴보면 파일 변경 방식에 어떤 제약이 가해졌는지 알아낼 수 있다. 원한다면 두 번째 프로세스에서 봉인을 더 적용해서 제약을 추가로 가할 수 있다. (단 `F_SEAL_SEAL` 봉인이 적용되지 않았어야 한다.)

## EXAMPLE

`memfd_create()`와 파일 봉인 API 사용 방식을 보여 주는 두 가지 예시 프로그램이 아래에 있다.

첫 번째 프로그램인 `t_memfd_create.c`에서는 `memfd_create()`로 <tt>[[tmpfs(5)]]</tt> 파일을 만들고, 파일 크기를 설정하고, 메모리로 맵 하고, 선택적으로 파일에 몇 가지 봉인을 둔다. 프로그램이 명령행 인자를 세 개까지 받는데 처음 두 개는 필수이다. 첫 번째 인자는 파일에 연계할 이름이고 두 번째 인자는 파일에 설정할 크기이며 선택적인 세 번째 인자는 파일에 설정할 봉인을 나타내는 문자열이다.

두 번째 프로그램인 `t_get_seals.c`를 이용해 `memfd_create()`로 생성된 기존 파일을 열어서 그 파일에 적용된 봉인 집합을 살펴볼 수 있다.

다음 셸 세션은 이 프로그램들의 사용 방식을 보여 준다. 먼저 <tt>[[tmpfs(5)]]</tt> 파일을 만들고 몇 가지 봉인을 설정한다.

```
$ ./t_memfd_create my_memfd_file 4096 sw &
[1] 11775
PID: 11775; fd: 3; /proc/11775/fd/3
```

이 시점에서 `t_memfd_create` 프로그램은 배경에서 실행을 계속한다. `memfd_create()`로 열었던 파일 디스크립터에 대응하는 `/proc/[pid]/fd` 파일을 또 다른 프로그램에서 열어서 `memfd_create()` 생성 파일에 대한 파일 디스크립터를 얻을 수 있다. 그 경로명을 이용해 심볼릭 링크 `/proc/[pid]/fd`의 내용을 확인하고 `t_get_seals` 프로그램을 이용해 그 파일에 적용된 봉인들을 본다.

```
$ readlink /proc/11775/fd/3
/memfd:my_memfd_file (deleted)
$ ./t_get_seals /proc/11775/fd/3
Existing seals: WRITE SHRINK
```

### 프로그램 소스: `t_memfd_create.c`

```c
#define _GNU_SOURCE
#include <sys/mman.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <stdio.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

int
main(int argc, char *argv[])
{
    int fd;
    unsigned int seals;
    char *addr;
    char *name, *seals_arg;
    ssize_t len;

    if (argc < 3) {
        fprintf(stderr, "%s name size [seals]\n", argv[0]);
        fprintf(stderr, "\t'seals' can contain any of the "
                "following characters:\n");
        fprintf(stderr, "\t\tg - F_SEAL_GROW\n");
        fprintf(stderr, "\t\ts - F_SEAL_SHRINK\n");
        fprintf(stderr, "\t\tw - F_SEAL_WRITE\n");
        fprintf(stderr, "\t\tS - F_SEAL_SEAL\n");
        exit(EXIT_FAILURE);
    }

    name = argv[1];
    len = atoi(argv[2]);
    seals_arg = argv[3];

    /* tmpfs에 익명 파일 생성. 파일에 봉인 할 수 있게 하기 */

    fd = memfd_create(name, MFD_ALLOW_SEALING);
    if (fd == -1)
        errExit("memfd_create");

    /* 명령행에서 지정한 대로 파일 크기 조정 */

    if (ftruncate(fd, len) == -1)
        errExit("truncate");

    printf("PID: %ld; fd: %d; /proc/%ld/fd/%d\n",
            (long) getpid(), fd, (long) getpid(), fd);

    /* 파일을 맵 하고 그 매핑에 데이터를 채우는 코드는 생략 */

    /* 명령행 인자 'seals'를 받았으면 파일에 봉인 설정 */

    if (seals_arg != NULL) {
        seals = 0;

        if (strchr(seals_arg, 'g') != NULL)
            seals |= F_SEAL_GROW;
        if (strchr(seals_arg, 's') != NULL)
            seals |= F_SEAL_SHRINK;
        if (strchr(seals_arg, 'w') != NULL)
            seals |= F_SEAL_WRITE;
        if (strchr(seals_arg, 'S') != NULL)
            seals |= F_SEAL_SEAL;

        if (fcntl(fd, F_ADD_SEALS, seals) == -1)
            errExit("fcntl");
    }

    /* memfd_create()로 생성한 파일이 계속 존재하도록
       실행을 계속 */

    pause();

    exit(EXIT_SUCCESS);
}
```

### 프로그램 소스: `t_get_seals.c`

```c
#define _GNU_SOURCE
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

int
main(int argc, char *argv[])
{
    int fd;
    unsigned int seals;

    if (argc != 2) {
        fprintf(stderr, "%s /proc/PID/fd/FD\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    fd = open(argv[1], O_RDWR);
    if (fd == -1)
        errExit("open");

    seals = fcntl(fd, F_GET_SEALS);
    if (seals == -1)
        errExit("fcntl");

    printf("Existing seals:");
    if (seals & F_SEAL_SEAL)
        printf(" SEAL");
    if (seals & F_SEAL_GROW)
        printf(" GROW");
    if (seals & F_SEAL_WRITE)
        printf(" WRITE");
    if (seals & F_SEAL_SHRINK)
        printf(" SHRINK");
    printf("\n");

    /* 파일을 맵 하고 결과 매핑 내용에 접근하는 코드는 생략 */

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[fcntl(2)]]</tt>, <tt>[[ftruncate(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[shmget(2)]]</tt>, <tt>[[shm_open(3)]]</tt>

----

2019-03-06
