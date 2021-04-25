## NAME

name_to_handle_at, open_by_handle_at - 경로명에 대한 핸들을 얻고 핸들을 통해 파일 열기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <sys/stat.h>
#include <fcntl.h>

int name_to_handle_at(int dirfd, const char *pathname,
                      struct file_handle *handle,
                      int *mount_id, int flags);
int open_by_handle_at(int mount_fd, struct file_handle *handle,
                      int flags);
```

## DESCRIPTION

`name_to_handle_at()` 및 `open_by_handle_at()` 시스템 호출은 <tt>[[openat(2)]]</tt>의 기능을 두 부분으로 쪼갠 것이다. `name_to_handle_at()`은 지정한 파일에 대응하는 불투명 핸들을 반환하며 `open_by_handle_at()`은 앞선 `name_to_handle_at()` 호출이 반환한 핸들에 대응하는 파일을 열어서 열린 파일 디스크립터를 반환한다.

### `name_to_handle_at()`

`name_to_handle_at()` 시스템 호출은 `dirfd` 및 `pathname` 인자로 지정한 파일에 대응하는 파일 핸들과 마운트 ID를 반환한다. 인자 `handle`을 통해 파일 핸들을 반환하는데, 다음과 같은 구조체에 대한 포인터이다.

```c
struct file_handle {
    unsigned int  handle_bytes;   /* f_handle의 크기 [in, out] */
    int           handle_type;    /* 핸들 타입 [out] */
    unsigned char f_handle[0];    /* 파일 식별자 (호출자가 크기 지정)
                                     [out] */
};
```

`f_handle`로 반환되는 핸들을 담기에 충분히 큰 구조체를 할당하는 건 호출자의 책임이다. 호출 전에 `f_handle`에 할당된 크기를 담도록 `handle_bytes` 필드를 초기화해야 한다. (`<fcntl.h>`에 정의된 상수 `MAX_HANDLE_SZ`가 파일 핸들 크기의 예상 최댓값을 나타낸다. 향후의 파일 시스템에서 더 큰 공간을 필요로 할 수 있으므로 보장된 상한은 아니다.) 성공 반환 시 실제 `f_handle`에 기록된 바이트 수를 담도록 `handle_bytes` 필드가 갱신된다.

호출자가 `handle->handle_bytes`를 0으로 호출을 해서 `file_handle` 구조체에 필요한 크기를 알아낼 수 있다. 이 경우 호출이 `EOVERFLOW` 오류로 실패하며 `handle->handle_bytes`가 필요 크기로 설정된다. 그러면 호출자가 그 정보를 이용해 올바른 크기의 구조체를 할당할 수 있다. (아래 EXAMPLES 참고.) `EOVERFLOW` 오류가 파일 핸들 검색을 정상적으로 지원하는 파일 시스템에서 이 특정 이름에 대해 사용 가능한 파일 핸들이 없다는 뜻일 수도 있기 때문에 약간의 주의가 필요하다. `handle_bytes`가 증가하지 않고 `EOVERFLOW` 오류가 반환되는 것으로 이 경우를 알아낼 수 있다.

`handle_bytes` 필드를 이용하는 것 외에는 호출자가 `file_handle` 구조체를 불투명한 데이터 타입으로 다뤄야 한다. `handle_type` 및 `f_handle` 필드가 필요한 곳은 이어지는 `open_by_handle_at()` 호출뿐이다.

`flags` 인자는 `AT_EMPTY_PATH`와 `AT_SYMLINK_FOLLOW`를 0개 이상 OR 해서 구성한 비트 마스크이다. 둘에 대해선 아래에서 설명한다.

`pathname` 및 `dirfd` 인자는 둘이 함께 핸들을 얻을 파일을 식별한다. 네 가지 경우가 있다.

* `pathname`이 비어 있지 않은 문자열이고 절대 경로명을 담고 있으면 그 경로명이 가리키는 파일에 대한 핸들을 반환한다. 이 경우 `dirfd`는 무시한다.

* `pathname`이 비어 있지 않은 문자열이고 상대 경로명을 담고 있으며 `dirfd`가 특수 값 `AT_FDCWD`이면 호출자의 현재 작업 디렉터리를 기준으로 `pathname`을 해석해서 그 경로명이 가리키는 파일에 대한 핸들을 반환한다.

* `pathname`이 비어 있지 않은 문자열이고 상대 경로명을 담고 있으며 `dirfd`가 디렉터리를 가리키는 파일 디스크립터이면 `dirfd`가 가리키는 디렉터리를 기준으로 `pathname`을 해석해서 그 경로명이 가리키는 파일에 대한 핸들을 반환한다. ("디렉터리 파일 디스크립터"가 유용한 이유에 대해선 <tt>[[openat(2)]]</tt> 참고.)

* `pathname`이 빈 문자열이고 `flags`에 `AT_EMPTY_PATH` 값이 지정돼 있으면 `dirfd`가 임의 종류의 파일을 가리키는 열린 파일 디스크립터이거나 현재 작업 디렉터리를 뜻하는 `AT_FDCWD`일 수 있으며, 그 파일 디스크립터가 가리키는 파일에 대한 핸들을 반환한다.

`mount_id` 인자는 `pathname`에 부합하는 파일 시스템 마운트에 대한 식별자를 반환한다. 이는 `/proc/self/mountinfo`에 있는 한 레코드의 첫 번째 필드에 대응한다. 그 레코드의 다섯 번째 필드에 있는 경로명을 열면 그 마운트 지점에 대한 파일 디스크립터가 나오고, 그 파일 디스크립터를 이어지는 `open_by_handle_at()` 호출에 사용할 수 있다. 성공 호출과 `EOVERFLOW` 오류가 난 호출 모두에서 `mount_id`가 반환된다.

기본적으로 `name_to_handle_at()`에서는 `pathname`이 심볼릭 링크인 경우 따라가지 않고 그 링크 자체에 대한 핸들을 반환한다. `flags`에 `AT_SYMLINK_FOLLOW`를 지정하면 `pathname`이 심볼릭 링크면 따라간다. (그래서 그 링크가 가리키는 파일에 대한 핸들을 반환한다.)

경로명의 마지막 구성 요소가 automount 지점일 때는 `name_to_handle_at()`이 마운트를 유발하지 않는다. 파일 시스템에서 파일 핸들과 automount 지점 모두를 지원할 때 automount 지점에 대한 `name_to_handle_at()` 호출은 `handle_bytes`를 올리지 않은 채 `EOVERFLOW` 오류를 반환하게 된다. 리눅스 4.13 이상에서 NFS를 쓸 때 서버 상의 별도 파일 시스템에 있는 디렉터리에 접근 시 이런 상황이 생길 수 있다. 이 경우에 경로명 끝에 "/"를 덧붙이면 automount를 일으킬 수 있다.

### `open_by_handle_at()`

`open_by_handle_at()` 시스템 호출은 앞선 `name_to_handle_at()` 호출이 반환한 파일 핸들인 `handle`이 가리키는 파일을 연다.

`mount_fd` 인자는 `handle`을 해석할 기준이 돼야 하는 마운트 된 파일 시스템 내의 임의 객체(파일, 디렉터리 등)에 대한 파일 디스크립터이다. 호출자의 현재 작업 디렉터리를 뜻하는 특수 값 `AT_FDCWD`를 지정할 수 있다.

`flags` 인자는 <tt>[[open(2)]]</tt>에서와 같다. `handle`이 심볼릭 링크를 가리키는 경우 호출자가 `O_PATH` 플래그를 지정해야 하며, 그 심볼릭 링크를 따라가지 않는다. `O_NOFOLLOW` 플래그는 지정 시 무시한다.

`open_by_handle_at()`을 쓰려면 호출자에게 `CAP_DAC_READ_SEARCH` 역능이 있어야 한다.

## RETURN VALUE

성공 시 `name_to_handle_at()`은 0을 반환하며 `open_by_handle_at()`은 파일 디스크립터(음수 아닌 정수)를 반환한다.

오류 발생 시 두 시스템 호출은 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`name_to_handle_at()`과 `open_by_handle_at()`이 <tt>[[openat(2)]]</tt>과 같은 오류로 실패할 수 있다. 더불어 아래에 적은 오류들로 실패할 수 있다.

`name_to_handle_at()`이 다음 오류로 실패할 수 있다.

`EFAULT`
:   `pathname`이나 `mount_id`, `handle`이 접근 가능한 주소 공간 밖을 가리킨다.

`EINVAL`
:   `flags`에 유효하지 않은 비트 값이 포함돼 있다.

`EINVAL`
:   `handle->handle_bytes`가 `MAX_HANDLE_SZ`보다 크다.

`ENOENT`
:   `pathname`이 빈 문자열인데 `flags`에 `AT_EMPTY_PATH`를 지정하지 않았다.

`ENOTDIR`
:   `dirfd`로 준 파일 디스크립터가 디렉터리를 가리키고 있지 않으며, `flags`에 `AT_EMPTY_PATH`가 있으면서 `pathname`이 빈 문자열인 경우가 아니다.

`EOPNOTSUPP`
:   파일 시스템에서 경로명을 파일 핸들로 디코딩 하는 걸 지원하지 않는다.

`EOVERFLOW`
:   호출로 전달한 `handle->handle_bytes` 값이 너무 작다. 이 오류 발생 시 핸들에 필요한 크기를 나타내도록 `handle->handle_bytes`가 갱신된다.

`open_by_handle_at()`이 다음 오류로 실패할 수 있다.

`EBADF`
:   `mount_fd`가 열린 파일 디스크립터가 아니다.

`EFAULT`
:   `handle`이 접근 가능한 주소 공간 밖을 가리킨다.

`EINVAL`
:   `handle->handle_bytes`가 `MAX_HANDLE_SZ`보다 크거나 0과 같다.

`ELOOP`
:   `handle`이 심볼릭 링크를 가리키는데 `flags`에 `O_PATH`가 지정돼 있지 않다.

`EPERM`
:   호출자에게 `CAP_DAC_READ_SEARCH` 역능이 없다.

`ESTALE`
:   지정한 `handle`이 유효하지 않다. 예를 들어 파일이 삭제됐을 때 이 오류가 발생하게 된다.

## VERSIONS

리눅스 2.6.39에서 이 시스템 호출들이 처음 등장했다. glibc 버전 2.14부터 라이브러리 지원을 제공한다.

## CONFORMING TO

이 시스템 호출들은 비표준 리눅스 확장이다.

FreeBSD에 대략 비슷한 형태의 시스템 호출 쌍 `getfh()`와 `openfh()`가 있다.

## NOTES

한 프로세스에서 `name_to_handle_at()`으로 파일 핸들을 만들고 나중에 다른 프로세스에서 `open_by_handle_at()` 호출에 쓸 수 있다.

일부 파일 시스템은 경로명에서 파일 핸들로의 변환을 지원하지 않는다. 예로 `/proc`, `/sys`, 여러 네트워크 파일 시스템들이 있다.

파일이 삭제되거나 기타 파일 시스템 자체적인 어떤 이유로 파일 핸들이 유효하지 않게 될 ("상할") 수 있다. 유효하지 않은 핸들은 `open_by_handle_at()`에서 `ESTALE` 오류로 알려 준다.

이 시스템 호출들은 사용자 공간 파일 서버에서 쓰도록 만들어진 것이다. 예를 들어 사용자 공간 NFS 서버에서 파일 핸들을 생성해서 NFS 클라이언트로 전달할 수 있을 것이다. 이후 클라이언트가 그 파일을 열고 싶으면 그 핸들을 서버에게 다시 전달할 수 있다. 이런 류의 기능을 이용하면 사용자 공간 파일 서버가 운용 파일들에 대해 무상태 방식으로 동작할 수 있다.

`pathname`이 심볼릭 링크를 가리키고 `flags`에 `AT_SYMLINK_FOLLOW`가 지정돼 있지 않은 경우에 `name_to_handle_at()`은 (링크가 가리키는 파일이 아니라) 링크에 대한 핸들을 반환한다. 이후 그 핸들을 받은 프로세스가 `O_PATH` 플래그를 쓴 `open_by_handle_at()`으로 핸들에서 파일 디스크립터로의 변환을 하고서 그 파일 디스크립터를 <tt>[[readlinkat(2)]]</tt>과 <tt>[[fchownat(2)]]</tt> 같은 시스템 호출에 `dirfd` 인자로 전달해서 그 심볼릭 링크에 대한 동작을 수행할 수 있다.

### 영속적인 파일 시스템 ID 얻기

파일 시스템이 마운트 되고 해제되면서 `/proc/self/mountinfo`의 마운트 ID가 재사용될 수 있다. 따라서 `name_to_handle_at()`이 (`*mount_id`에) 반환한 마운트 ID를 해당하는 마운트 된 파일 시스템에 대한 영속적 식별자로 다루지 말아야 한다. 하지만 응용에서 그 마운트 ID에 대응하는 `mountinfo` 레코드의 정보를 이용해 영속적인 식별자를 얻어낼 수 있다.

예를 들어 `mountinfo` 레코드의 다섯 번째 필드에 있는 장치 이름을 가지고 `/dev/disks/by-uuid` 내의 심볼릭 링크를 통해서 해당 장치의 UUID를 찾을 수 있다. (UUID를 얻는 더 편한 방법은 `libblkid(3)` 라이브러리를 이용하는 것이다.) 그러면 그 과정을 뒤집어서 UUID를 가지고 장치 이름을 찾아서 해당 마운트 지점을 얻고, 그래서 `open_by_handle_at()`에 쓸 `mount_fd` 인자를 만들어 낼 수 있다.

## EXAMPLES

아래 두 프로그램이 `name_to_handle_at()`과 `open_by_handle_at()` 사용 방식을 보여 준다. 첫 번째 프로그램(`t_name_to_handle_at.c`)에서는 `name_to_handle_at()`을 사용해 명령행 인자로 지정한 파일에 대한 파일 핸들과 마운트 ID를 얻는다. 그리고 그 핸들과 마운트 ID를 표준 출력으로 찍는다.

두 번째 프로그램(`t_open_by_handle_at.c`)에서는 표준 입력으로부터 마운트 ID와 파일 핸들을 읽어 들인다. 그러고 나서 `open_by_handle_at()`을 사용해 그 핸들로 파일을 연다. 선택적인 명령행 인자를 준 경우에는 지명된 디렉터리를 열어서 `open_by_handle_at()`에 쓸 `mount_fd` 인자를 얻는다. 아닌 경우에는 `/proc/self/mountinfo`를 탐색해서 표준 입력으로 읽은 마운트 ID와 일치하는 레코드를 찾은 다음 그 레코드에 나온 마운트 디렉터리를 열어서 `mount_fd`를 얻는다. (이 프로그램들에서는 마운트 ID가 영속적이지 않다는 점까지는 다루지 않는다.)

다음 셸 세션이 두 프로그램의 사용 방식을 보여 준다.

```text
$ echo 'Can you please think about it?' > cecilia.txt
$ ./t_name_to_handle_at cecilia.txt > fh
$ ./t_open_by_handle_at < fh
open_by_handle_at: Operation not permitted
$ sudo ./t_open_by_handle_at < fh      # CAP_SYS_ADMIN 필요함
Read 31 bytes
$ rm cecilia.txt
```

이제 파일을 삭제하고 (재빨리) 다시 만들어서 내용이 같고 (우연히) 아이노드도 같도록 한다. 그렇게 해도 `open_by_handle_at()`에서 파일 핸들이 가리키는 원래 파일이 더이상 존재하지 않는다는 걸 안다.

```text
$ stat --printf="%i\n" cecilia.txt     # 아이노드 번호 표시
4072121
$ rm cecilia.txt
$ echo 'Can you please think about it?' > cecilia.txt
$ stat --printf="%i\n" cecilia.txt     # 아이노드 번호 확인
4072121
$ sudo ./t_open_by_handle_at < fh
open_by_handle_at: Stale NFS file handle
```

### 프로그램 소스: `t_name_to_handle_at.c`

```c
#define _GNU_SOURCE
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

int
main(int argc, char *argv[])
{
    struct file_handle *fhp;
    int mount_id, fhsize, flags, dirfd;
    char *pathname;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s pathname\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    pathname = argv[1];

    /* file_handle 구조체 할당하기. */

    fhsize = sizeof(*fhp);
    fhp = malloc(fhsize);
    if (fhp == NULL)
        errExit("malloc");

    /* 첫 번째 name_to_handle_at() 호출로 파일 핸들에 필요한
       크기 알아내기. */

    dirfd = AT_FDCWD;           /* name_to_handle_at() 호출에 사용 */
    flags = 0;                  /* name_to_handle_at() 호출에 사용 */
    fhp->handle_bytes = 0;
    if (name_to_handle_at(dirfd, pathname, fhp,
                &mount_id, flags) != -1 || errno != EOVERFLOW) {
        fprintf(stderr, "Unexpected result from name_to_handle_at()\n");
        exit(EXIT_FAILURE);
    }

    /* 올바른 크기로 file_handle 구조체 재할당하기. */

    fhsize = sizeof(*fhp) + fhp->handle_bytes;
    fhp = realloc(fhp, fhsize);         /* fhp->handle_bytes 복사됨 */
    if (fhp == NULL)
        errExit("realloc");

    /* 명령행에서 받은 pathname을 가지고 파일 핸들 얻기. */

    if (name_to_handle_at(dirfd, pathname, fhp, &mount_id, flags) == -1)
        errExit("name_to_handle_at");

    /* 마운트 ID, 파일 핸들 크기, 파일 핸들을 stdout으로 출력.
       이후 t_open_by_handle_at.c에서 사용한다. */

    printf("%d\n", mount_id);
    printf("%u %d   ", fhp->handle_bytes, fhp->handle_type);
    for (int j = 0; j < fhp->handle_bytes; j++)
        printf(" %02x", fhp->f_handle[j]);
    printf("\n");

    exit(EXIT_SUCCESS);
}
```

### 프로그램 소스: `t_open_by_handle_at.c`

```c
#define _GNU_SOURCE
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

/* /proc/self/mountinfo를 탐색해서 마운트 ID가 'mount_id'와 일치하는
   행을 찾아낸다. (더 쉬운 방법은 'util-linux' 프로젝트에서 제공하는
   'libmount'를 설치해서 쓰는 것이다.)
   해당 마운트 경로를 열어서 나오는 파일 디스크립터를 반환한다. */

static int
open_mount_path_by_id(int mount_id)
{
    char *linep;
    size_t lsize;
    char mount_path[PATH_MAX];
    int mi_mount_id, found;
    ssize_t nread;
    FILE *fp;

    fp = fopen("/proc/self/mountinfo", "r");
    if (fp == NULL)
        errExit("fopen");

    found = 0;
    linep = NULL;
    while (!found) {
        nread = getline(&linep, &lsize, fp);
        if (nread == -1)
            break;

        nread = sscanf(linep, "%d %*d %*s %*s %s",
                       &mi_mount_id, mount_path);
        if (nread != 2) {
            fprintf(stderr, "Bad sscanf()\n");
            exit(EXIT_FAILURE);
        }

        if (mi_mount_id == mount_id)
            found = 1;
    }
    free(linep);

    fclose(fp);

    if (!found) {
        fprintf(stderr, "Could not find mount point\n");
        exit(EXIT_FAILURE);
    }

    return open(mount_path, O_RDONLY);
}

int
main(int argc, char *argv[])
{
    struct file_handle *fhp;
    int mount_id, fd, mount_fd, handle_bytes;
    ssize_t nread;
    char buf[1000];
#define LINE_SIZE 100
    char line1[LINE_SIZE], line2[LINE_SIZE];
    char *nextp;

    if ((argc > 1 && strcmp(argv[1], "--help") == 0) || argc > 2) {
        fprintf(stderr, "Usage: %s [mount-path]\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    /* 표준 입력에는 마운트 ID와 파일 핸들 정보가 있다:

         1행: <mount_id>
         2행: <handle_bytes> <handle_type>   <16진수로 나타낸 핸들>
    */

    if ((fgets(line1, sizeof(line1), stdin) == NULL) ||
           (fgets(line2, sizeof(line2), stdin) == NULL)) {
        fprintf(stderr, "Missing mount_id / file handle\n");
        exit(EXIT_FAILURE);
    }

    mount_id = atoi(line1);

    handle_bytes = strtoul(line2, &nextp, 0);

    /* handle_bytes를 아니까 이제 file_handle 구조체를 할당할 수 있다. */

    fhp = malloc(sizeof(*fhp) + handle_bytes);
    if (fhp == NULL)
        errExit("malloc");

    fhp->handle_bytes = handle_bytes;

    fhp->handle_type = strtoul(nextp, &nextp, 0);

    for (int j = 0; j < fhp->handle_bytes; j++)
        fhp->f_handle[j] = strtoul(nextp, &nextp, 16);

    /* 마운트 지점에 대한 파일 디스크립터 얻기.
       명령행에 지정된 경로명을 열거나, /proc/self/mounts를
       탐색해서 stdin으로 받은 'mount_id'에 일치하는 마운트 항목
       찾아내기. */

    if (argc > 1)
        mount_fd = open(argv[1], O_RDONLY);
    else
        mount_fd = open_mount_path_by_id(mount_id);

    if (mount_fd == -1)
        errExit("opening mount fd");

    /* 핸들과 마운트 지점으로 파일 열기. */

    fd = open_by_handle_at(mount_fd, fhp, O_RDONLY);
    if (fd == -1)
        errExit("open_by_handle_at");

    /* 파일에서 몇 바이트 읽어 보기. */

    nread = read(fd, buf, sizeof(buf));
    if (nread == -1)
        errExit("read");

    printf("Read %zd bytes\n", nread);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[open(2)]]</tt>, `libblkid(3)`, `blkid(8)`, `findfs(8)`, <tt>[[mount(8)]]</tt>

`util-linux` 최신 릴리스의 `libblkid` 및 `libmount` 문서 (<https://www.kernel.org/pub/linux/utils/util-linux/>)

----

2021-03-22
