## NAME

ioctl_ns - 리눅스 네임스페이스를 위한 ioctl() 동작들

## DESCRIPTION

### 네임스페이스 관계 알아내기

네임스페이스 관계(<tt>[[user_namespaces(7)]]</tt> 및 <tt>[[pid_namespaces(7)]]</tt> 참고)를 알아낼 수 있도록 다음 `ioctl(2)` 동작을 제공한다. 호출 형태는 이렇다.

```c
new_fd = ioctl(fd, request);
```

각 경우에서 `fd`는 `/proc/[pid]/ns/*` 파일을 가리킨다. 성공 시 두 동작 모두 새 파일 디스크립터를 반환한다.

`NS_GET_USERNS` (리눅스 4.9부터)
:   `fd`가 가리키는 네임스페이스를 소유한 사용자 네임스페이스를 가리키는 파일 디스크립터를 반환한다.

`NS_GET_PARENT` (리눅스 4.9부터)
:   `fd`가 가리키는 네임스페이스의 부모 네임스페이스를 가리키는 파일 디스크립터를 반환한다. 계층적 네임스페이스(즉 PID 네임스페이스와 사용자 네임스페이스)에만 이 동작이 유효하다. 사용자 네임스페이스에서 `NS_GET_PARENT`와 `NS_GET_USERNS`는 같은 의미이다.

이 동작들이 반환하는 새 파일 디스크립터는 `O_RDONLY` 및 `O_CLOEXEC`(exec에서 닫기, <tt>[[fcntl(2)]]</tt> 참고) 플래그로 열려 있다.

반환된 파일 디스크립터에 <tt>[[fstat(2)]]</tt> 하면 얻는 `stat` 구조체의 `st_dev`(거주 장치) 및 `st_ino`(아이노드 번호) 필드로 소유/부모 네임스페이스를 식별한다. 그 아이노드 번호를 다른 `/proc/[pid]/ns/{pid,user}` 파일의 아이노드 번호와 맞춰 봐서 소유/부모 네임스페이스인지를 알아낼 수 있다.

어느 쪽 `ioctl(2)` 동작이든 다음 오류로 실패할 수 있다.

`EPERM`
:   요청한 네임스페이스가 호출자의 네임스페이스 범위 밖에 있다. 예를 들어 소유 사용자 네임스페이스가 호출자의 현재 사용자 네임스페이스의 조상이면 이 오류가 발생할 수 있다. 최초 사용자 내지 PID 네임스페이스의 부모를 얻으려고 할 때에도 발생할 수 있다.

`ENOTTY`
:   현 커널 버전에서 동작을 지원하지 않는다.

추가로 `NS_GET_PARENT` 동작이 다음 오류로 실패할 수 있다.

`EINVAL`
:   `fd`가 비계층적 네임스페이스를 가리키고 있다.

이 동작들의 사용 예시는 EXAMPLE 절 참고.

### 네임스페이스 종류 알아내기

`NS_GET_NSTYPE` 동작(리눅스 4.11부터 사용 가능)을 이용해 파일 디스크립터 `fd`가 가리키는 네임스페이스의 유형을 알아낼 수 있다.

```c
nstype = ioctl(fd, NS_GET_NSTYPE);
```

`fd`는 `/proc/[pid]/ns/*` 파일을 가리킨다.

반환 값은 네임스페이스를 만들기 위해 <tt>[[clone(2)]]</tt>이나 <tt>[[unshare(2)]]</tt>에 지정할 수 있는 `CLONE_NEW*` 값들 중 하나이다.

### 사용자 네임스페이스 소유자 알아내기

`NS_GET_OWNER_UID` 동작(리눅스 4.11부터 사용 가능)을 이용해 사용자 네임스페이스를 소유한 사용자 ID를 (즉 사용자 네임스페이스를 생성한 프로세스의 실효 사용자 ID를) 알아낼 수 있다. 다음처럼 호출한다.

```c
uid_t uid;
ioctl(fd, NS_GET_OWNER_UID, &uid);
```

`fd`는 `/proc/[pid]/ns/user` 파일을 가리킨다.

세 번째 인자가 가리키는 `uid_t`로 소유 사용자 ID가 반환된다.

이 동작이 다음 오류로 실패할 수 있다.

`EINVAL`
:   `fd`가 사용자 네임스페이스를 가리키고 있지 않다.

## ERRORS

위의 `ioctl()` 동작들 어느 것이든 다음 오류를 반환할 수 있다.

`ENOTTY`
:   `fd`가 `/proc/[pid]/ns/*` 파일을 가리키고 있지 않다.

## CONFORMING TO

이 페이지에서 설명하는 네임스페이스 및 동작들은 리눅스 전용이다.

## EXAMPLE

아래 있는 예시 프로그램에서는 위에 설명한 `ioctl(2)` 동작들을 사용해 간단한 네임스페이스 관계 탐색을 수행한다. 다음 셸 세션들은 이 프로그램의 다양한 사용 방식을 보여 준다.

최초 사용자 네임스페이스의 부모를 얻으려고 시도하면 부모가 없으므로 실패한다.

```
$ ./ns_show /proc/self/ns/user p
The parent namespace is outside your namespace scope
```

새로운 사용자 및 UTS 네임스페이스 내에서 `sleep(1)`을 실행하는 프로세스를 만든다. 그리고 새 UTS 네임스페이스가 새 사용자 네임스페이스와 연계돼 있음을 보인다.

```
$ unshare -Uu sleep 1000 &
[1] 23235
$ ./ns_show /proc/23235/ns/uts u
Device/Inode of owning user namespace is: [0,3] / 4026532448
$ readlink /proc/23235/ns/user
user:[4026532448]
```

그리고 위 예의 새 사용자 네임스페이스의 부모가 최초 사용자 네임스페이스임을 보인다.

```
$ readlink /proc/self/ns/user
user:[4026531837]
$ ./ns_show /proc/23235/ns/user p
Device/Inode of parent namespace is: [0,3] / 4026531837
```

새 사용자 네임스페이스에서 셸을 시작한다. 그리고 그 셸 내에서 부모 사용자 네임스페이스를 알아낼 수 없음을 보인다. 마찬가지로 (최초 사용자 네임스페이스에 연계돼 있는) UTS 네임스페이스도 알아낼 수 없다.

```
$ PS1="sh2$ " unshare -U bash
sh2$ ./ns_show /proc/self/ns/user p
The parent namespace is outside your namespace scope
sh2$ ./ns_show /proc/self/ns/uts u
The owning user namespace is outside your namespace scope
```

### 프로그램 소스

```c
/* ns_show.c

   Licensed under the GNU General Public License v2 or later.
*/
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/ioctl.h>
#include <errno.h>
#include <sys/sysmacros.h>

#ifndef NS_GET_USERNS
#define NSIO    0xb7
#define NS_GET_USERNS   _IO(NSIO, 0x1)
#define NS_GET_PARENT   _IO(NSIO, 0x2)
#endif

int
main(int argc, char *argv[])
{
    int fd, userns_fd, parent_fd;
    struct stat sb;

    if (argc < 2) {
        fprintf(stderr, "Usage: %s /proc/[pid]/ns/[file] [p|u]\n",
                argv[0]);
        fprintf(stderr, "\nDisplay the result of one or both "
                "of NS_GET_USERNS (u) or NS_GET_PARENT (p)\n"
                "for the specified /proc/[pid]/ns/[file]. If neither "
                "'p' nor 'u' is specified,\n"
                "NS_GET_USERNS is the default.\n");
        exit(EXIT_FAILURE);
    }

    /* argv[1]에 지정된 'ns' 파일에 대한 파일 디스크립터 얻기 */

    fd = open(argv[1], O_RDONLY);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }

    /* 소유 사용자 네임스페이스에 대한 파일 디스크립터를 얻고
       그 네임스페이스의 아이노드 번호를 얻어서 표시 */

    if (argc < 3 || strchr(argv[2], 'u')) {
        userns_fd = ioctl(fd, NS_GET_USERNS);

        if (userns_fd == -1) {
            if (errno == EPERM)
                printf("The owning user namespace is outside "
                        "your namespace scope\n");
            else
                perror("ioctl-NS_GET_USERNS");
            exit(EXIT_FAILURE);
        }

        if (fstat(userns_fd, &sb) == -1) {
            perror("fstat-userns");
            exit_EXIT_FAILURE);
        }
        printf("Device/Inode of owning user namespace is: "
                "[%lx,%lx] / %ld\n",
                (long) major(sb.st_dev), (long) minor(sb.st_dev),
                (long) sb.st_ino);

        close(userns_fd);
    }

    /* 부모 네임스페이스에 대한 파일 디스크립터를 얻고
       그 네임스페이스의 아이노드 번호를 얻어서 표시 */

    if (argc > 2 && strchr(argv[2], 'p')) {
        parent_fd = ioctl(fd, NS_GET_PARENT);

        if (parent_fd == -1) {
            if (errno == EINVAL)
                printf("Can't get parent namespace of a "
                        "nonhierarchical namespace\n");
            else if (errno == EPERM)
                printf("The parent namespace is outside "
                        "your namespace scope\n");
            else
                perror("ioctl-NS_GET_PARENT");
            exit(EXIT_FAILURE);
        }

        if (fstat(parent_fd, &sb) == -1) {
            perror("fstat-parentns");
            exit_EXIT_FAILURE);
        }
        printf("Device/Inode of parent namespace is: [%lx,%lx] / %ld\n",
                (long) major(sb.st_dev), (long) minor(sb.st_dev),
                (long) sb.st_ino);

        close(parent_fd);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[fstat(2)]]</tt>, `ioctl(2)`, <tt>[[proc(5)]]</tt>, <tt>[[namespaces(7)]]</tt>

----

2019-03-06
