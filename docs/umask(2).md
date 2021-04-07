## NAME

umask - 파일 모드 생성 마스크 설정하기

## SYNOPSIS

```c
#include <sys/types.h>
#include <sys/stat.h>

mode_t umask(mode_t mask);
```

## DESCRIPTION

`umask()`는 호출 프로세스의 파일 모드 생성 마스크(umask)를 `mask` & 0777로 설정하고 (즉 `mask`의 파일 권한 비트들만 사용) 이전 마스크 값을 반환한다.

파일을 생성하는 <tt>[[open(2)]]</tt>, <tt>[[mkdir(2)]]</tt> 등의 시스템 호출에서 umask를 사용해서 새로 생성하는 파일 내지 디렉터리에 줄 권한을 변경한다. 구체적으로는 <tt>[[open(2)]]</tt> 및 <tt>[[mkdir(2)]]</tt>의 `mode` 인자에서 umask에 있는 권한들을 끈다.

반면 부모 디렉터리에 기본 ACL(`acl(5)` 참고)이 있으면 umask를 무시하고, 그 기본 ACL을 물려받고, 물려받은 ACL에 따라 권한 비트들을 설정하고, `mode` 인자에 빠져 있는 권한 비트들을 끈다. 예를 들어 다음 기본 ACL이 umask 022와 동등하다.

```
u::rwx,g::r-x,o::r-x
```

이 기본 ACL의 효과에 `mode` 인자 0666(rw-rw-rw-)을 합쳐서 나오는 파일 권한은 0644(rw-r--r--)가 될 것이다.

`mask` 지정에 사용해야 할 상수들을 <tt>[[inode(7)]]</tt>에서 기술한다.

프로세스 umask의 일반적인 기본값은 `S_IWGRP | S_IWOTH`(8진수로 022)이다. <tt>[[open(2)]]</tt>의 `mode` 인자를 다음(8진수로 0666)으로 지정하는 일반적인 경우에

```
S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH
```

새 파일 생성 시 결과 파일의 권한은 다음이 된다.

```
S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH
```

(0666 & ~022 = 0644, 즉 rw-r--r--.)

## RETURN VALUE

이 시스템 호출은 항상 성공하며 이전 마스크 값을 반환한다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

## NOTES

<tt>[[fork(2)]]</tt>를 통해 생성된 자식 프로세스는 부모의 umask를 물려받는다. <tt>[[execve(2)]]</tt>에서 umask를 바꾸지 않고 놔둔다.

`umask()`를 사용해 프로세스의 umask를 변경 없이 가져오는 건 불가능하다. 그래서 umask를 복원하기 위해 다시 `umask()` 호출이 필요해진다. 이 두 단계의 비원자성이 다중 스레드 프로그램에서 경쟁 가능성을 만든다.

리눅스 4.7부터 `/proc/[pid]/status`의 `Umask` 필드를 통해 모든 프로세스의 umask를 볼 수 있다. `/proc/self/status`의 이 필드를 확인하면 프로세스에서 자기 umask를 변경 없이 얻어 올 수 있다.

umask 설정은 프로세스에서 만드는 POSIX IPC 객체(<tt>[[mq_open(3)]]</tt>, <tt>[[sem_open(3)]]</tt>, <tt>[[shm_open(3)]]</tt>), FIFO(<tt>[[mkfifo(3)]]</tt>), 유닉스 도메인 소켓(<tt>[[unix(7)]]</tt>)에 할당되는 권한에도 영향을 준다. umask가 프로세스에서 (<tt>[[msgget(2)]]</tt>, <tt>[[semget(2)]]</tt>, <tt>[[shmget(2)]]</tt>으로) 만드는 시스템 V IPC 객체에 할당되는 권한에는 영향을 주지 않는다.

## SEE ALSO

<tt>[[chmod(2)]]</tt>, <tt>[[mkdir(2)]]</tt>, <tt>[[open(2)]]</tt>, <tt>[[stat(2)]]</tt>, `acl(5)`

----

2017-09-15
