## NAME

signal-safety - 비동기 시그널 안전 함수들

## DESCRIPTION

*비동기 시그널 안전(async-signal-safe)* 함수는 시그널 핸들러 안에서 안전하게 호출할 수 있는 함수이다. 많은 함수들은 비동기 시그널 안전 함수가 *아니다*. 특히 재진입 가능하지 않은 함수들은 일반적으로 시그널 핸들러 안에서 호출하는 게 안전하지 않다.

모든 함수들이 비동기 시그널 안전이 아닌 `stdio` 라이브러리 구현을 생각해 보면 함수를 안전하지 않게 만드는 종류의 문제들을 금방 이해할 수 있다.

파일에 대해 버퍼링 I/O를 수행할 때 `stdio` 함수들에서는 정적 할당 데이터 버퍼와 함께 버퍼 내 데이터 양과 현재 위치를 기록하는 연계 카운터 및 인덱스를 (또는 포인터들을) 유지해야 한다. 주 프로그램이 <tt>[[printf(3)]]</tt> 같은 `stdio` 함수 호출 중에 있고 그 안에서 버퍼와 연관 변수들을 부분적으로 갱신했다고 하자. 만약 그 순간에 시그널 핸들러가 프로그램을 중단시키고 핸들러에서 다시 <tt>[[printf(3)]]</tt>를 호출하면 그 두 번째 <tt>[[printf(3)]]</tt> 호출은 일관적이지 않은 데이터로 동작하게 되며 결과가 예측 불가능하다.

안전하지 않은 함수의 문제를 피하려면 두 가지 선택지가 있다.

1. (a) 시그널 핸들러에서 비동기 시그널 안전 함수만 호출하고 (b) 시그널 핸들러 자체가 주 프로그램의 전역 변수들에 대해 재진입 가능이게 한다.

2. 주 프로그램에서 안전하지 않은 함수를 호출하거나 시그널 핸들러에서도 접근하는 전역 데이터를 다룰 때 시그널 전달을 막는다.

일반적으로 두 번째 선택지는 여하한 복잡도의 프로그램에서도 어렵기 때문에 첫 번째 선택지를 취한다.

POSIX.1에서는 구현에서 비동기 시그널 안전으로 만들어야 하는 함수들을 명세한다. (어떤 구현에서는 더 많은 함수들에 안전한 구현을 제공할 수도 있다. 하지만 표준에서 요구하는 것이 아니므로 다른 구현에서는 같은 보장을 하지 않을 수도 있다.)

일반적으로 함수가 비동기 시그널 안전 건 재진입 가능하거나 시그널에 대해 원자적(즉 시그널 핸들러로 실행을 중단시킬 수 없음)이어서이다.

POSIX.1에서 비동기 시그널 안전이기를 요구하는 함수들이 아래 표에 있다. 별다른 언급이 없는 함수는 POSIX.1-2001에서 비동기 시그널 안전을 요구한 것이며, 후속 표준에서 바뀐 내용이 적혀 있다.

| 함수                             | 참고 |
| -------------------------------- | --- |
| <tt>[[abort(3)]]</tt>            | POSIX.1-2001 TC1에서 추가 |
| `accept(2)`                      | |
| <tt>[[access(2)]]</tt>           | |
| <tt>[[aio_error(3)]]</tt>        | |
| <tt>[[aio_return(3)]]</tt>       | |
| <tt>[[aio_suspend(3)]]</tt>      | 아래 참고 |
| <tt>[[alarm(2)]]</tt>            | |
| `bind(2)`                        | |
| <tt>[[cfgetispeed(3)]]</tt>      | |
| <tt>[[cfgetospeed(3)]]</tt>      | |
| <tt>[[cfsetispeed(3)]]</tt>      | |
| <tt>[[cfsetospeed(3)]]</tt>      | |
| <tt>[[chdir(2)]]</tt>            | |
| <tt>[[chmod(2)]]</tt>            | |
| <tt>[[chown(2)]]</tt>            | |
| <tt>[[clock_gettime(2)]]</tt>    | |
| <tt>[[close(2)]]</tt>            | |
| `connect(2)`                     | |
| <tt>[[creat(2)]]</tt>            | |
| <tt>[[dup(2)]]</tt>              | |
| <tt>[[dup2(2)]]</tt>             | |
| <tt>[[execl(3)]]</tt>            | POSIX.1-2008에서 추가, 아래 참고 |
| <tt>[[execle(3)]]</tt>           | 아래 참고 |
| <tt>[[execv(3)]]</tt>            | POSIX.1-2008에서 추가 |
| <tt>[[execve(2)]]</tt>           | |
| <tt>[[_exit(2)]]</tt>            | |
| <tt>[[_Exit(2)]]</tt>            | |
| <tt>[[faccessat(2)]]</tt>        | POSIX.1-2008에서 추가 |
| <tt>[[fchdir(2)]]</tt>           | POSIX.1-2008 TC1에서 추가 |
| <tt>[[fchmod(2)]]</tt>           | |
| <tt>[[fchmodat(2)]]</tt>         | POSIX.1-2008에서 추가 |
| <tt>[[fchown(2)]]</tt>           | |
| <tt>[[fchownat(2)]]</tt>         | POSIX.1-2008에서 추가 |
| <tt>[[fcntl(2)]]</tt>            | |
| <tt>[[fdatasync(2)]]</tt>        | |
| <tt>[[fexecve(3)]]</tt>          | POSIX.1-2008에서 추가 |
| <tt>[[ffs(3)]]</tt>              | POSIX.1-2008 TC2에서 추가 |
| <tt>[[fork(2)]]</tt>             | 아래 참고 |
| <tt>[[fstat(2)]]</tt>            | |
| <tt>[[fstatat(2)]]</tt>          | POSIX.1-2008에서 추가 |
| <tt>[[fsync(2)]]</tt>            | |
| <tt>[[ftruncate(2)]]</tt>        | |
| <tt>[[futimens(3)]]</tt>         | POSIX.1-2008에서 추가 |
| <tt>[[getegid(2)]]</tt>          | |
| <tt>[[geteuid(2)]]</tt>          | |
| <tt>[[getgid(2)]]</tt>           | |
| <tt>[[getgroups(2)]]</tt>        | |
| <tt>[[getpeername(2)]]</tt>      | |
| <tt>[[getpgrp(2)]]</tt>          | |
| <tt>[[getpid(2)]]</tt>           | |
| <tt>[[getppid(2)]]</tt>          | |
| <tt>[[getsockname(2)]]</tt>      | |
| `getsockopt(2)`                  | |
| <tt>[[getuid(2)]]</tt>           | |
| <tt>[[htonl(3)]]</tt>            | POSIX.1-2008 TC2에서 추가 |
| <tt>[[htons(3)]]</tt>            | POSIX.1-2008 TC2에서 추가 |
| <tt>[[kill(2)]]</tt>             | |
| <tt>[[link(2)]]</tt>             | |
| <tt>[[linkat(2)]]</tt>           | POSIX.1-2008에서 추가 |
| `listen(2)`                      | |
| <tt>[[longjmp(3)]]</tt>          | POSIX.1-2008 TC2에서 추가, 아래 참고 |
| <tt>[[lseek(2)]]</tt>            | |
| <tt>[[lstat(2)]]</tt>            | |
| <tt>[[memccpy(3)]]</tt>          | POSIX.1-2008 TC2에서 추가 |
| <tt>[[memchr(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| <tt>[[memcmp(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| <tt>[[memcpy(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| <tt>[[memmove(3)]]</tt>          | POSIX.1-2008 TC2에서 추가 |
| <tt>[[memset(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| <tt>[[mkdir(2)]]</tt>            | |
| <tt>[[mkdirat(2)]]</tt>          | POSIX.1-2008에서 추가 |
| <tt>[[mkfifo(3)]]</tt>           | |
| <tt>[[mkfifoat(3)]]</tt>         | POSIX.1-2008에서 추가 |
| <tt>[[mknod(2)]]</tt>            | POSIX.1-2008에서 추가 |
| <tt>[[mknodat(2)]]</tt>          | POSIX.1-2008에서 추가 |
| <tt>[[ntohl(3)]]</tt>            | POSIX.1-2008 TC2에서 추가 |
| <tt>[[ntohs(3)]]</tt>            | POSIX.1-2008 TC2에서 추가 |
| <tt>[[open(2)]]</tt>             | |
| <tt>[[openat(2)]]</tt>           | POSIX.1-2008에서 추가 |
| <tt>[[pause(2)]]</tt>            | |
| <tt>[[pipe(2)]]</tt>             | |
| <tt>[[poll(2)]]</tt>             | |
| `posix_trace_event(3)`           | |
| <tt>[[pselect(2)]]</tt>          | |
| <tt>[[pthread_kill(3)]]</tt>     | POSIX.1-2008 TC1에서 추가 |
| <tt>[[pthread_self(3)]]</tt>     | POSIX.1-2008 TC1에서 추가 |
| <tt>[[pthread_sigmask(3)]]</tt>  | POSIX.1-2008 TC1에서 추가 |
| <tt>[[raise(3)]]</tt>            | |
| <tt>[[read(2)]]</tt>             | |
| <tt>[[readlink(2)]]</tt>         | |
| <tt>[[readlinkat(2)]]</tt>       | POSIX.1-2008에서 추가 |
| <tt>[[recv(2)]]</tt>             | |
| <tt>[[recvfrom(2)]]</tt>         | |
| <tt>[[recvmsg(2)]]</tt>          | |
| <tt>[[rename(2)]]</tt>           | |
| <tt>[[renameat(2)]]</tt>         | POSIX.1-2008에서 추가 |
| <tt>[[rmdir(2)]]</tt>            | |
| <tt>[[select(2)]]</tt>           | |
| <tt>[[sem_post(3)]]</tt>         | |
| <tt>[[send(2)]]</tt>             | |
| <tt>[[sendmsg(2)]]</tt>          | |
| <tt>[[sendto(2)]]</tt>           | |
| <tt>[[setgid(2)]]</tt>           | |
| <tt>[[setpgid(2)]]</tt>          | |
| <tt>[[setsid(2)]]</tt>           | |
| `setsockopt(2)`                  | |
| <tt>[[setuid(2)]]</tt>           | |
| <tt>[[shutdown(2)]]</tt>         | |
| <tt>[[sigaction(2)]]</tt>        | |
| <tt>[[sigaddset(3)]]</tt>        | |
| <tt>[[sigdelset(3)]]</tt>        | |
| <tt>[[sigemptyset(3)]]</tt>      | |
| <tt>[[sigfillset(3)]]</tt>       | |
| <tt>[[sigismember(3)]]</tt>      | |
| <tt>[[siglongjmp(3)]]</tt>       | POSIX.1-2008 TC2에서 추가, 아래 참고 |
| <tt>[[signal(2)]]</tt>           | |
| <tt>[[sigpause(3)]]</tt>         | |
| <tt>[[sigpending(2)]]</tt>       | |
| <tt>[[sigprocmask(2)]]</tt>      | |
| <tt>[[sigqueue(2)]]</tt>         | |
| <tt>[[sigset(3)]]</tt>           | |
| <tt>[[sigsuspend(2)]]</tt>       | |
| <tt>[[sleep(3)]]</tt>            | |
| <tt>[[sockatmark(3)]]</tt>       | POSIX.1-2001 TC2에서 추가 |
| <tt>[[socket(2)]]</tt>           | |
| <tt>[[socketpair(2)]]</tt>       | |
| <tt>[[stat(2)]]</tt>             | |
| <tt>[[stpcpy(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| <tt>[[stpncpy(3)]]</tt>          | POSIX.1-2008 TC2에서 추가 |
| `strcat(3)`                      | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strchr(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strcmp(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strcpy(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| `strcspn(3)`                     | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strlen(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| `strncat(3)`                     | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strncmp(3)]]</tt>          | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strncpy(3)]]</tt>          | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strnlen(3)]]</tt>          | POSIX.1-2008 TC2에서 추가 |
| `strpbrk(3)`                     | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strrchr(3)]]</tt>          | POSIX.1-2008 TC2에서 추가 |
| `strspn(3)`                      | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strstr(3)]]</tt>           | POSIX.1-2008 TC2에서 추가 |
| <tt>[[strtok_r(3)]]</tt>         | POSIX.1-2008 TC2에서 추가 |
| <tt>[[symlink(2)]]</tt>          | |
| <tt>[[symlinkat(2)]]</tt>        | POSIX.1-2008에서 추가 |
| <tt>[[tcdrain(3)]]</tt>          | |
| <tt>[[tcflow(3)]]</tt>           | |
| <tt>[[tcflush(3)]]</tt>          | |
| <tt>[[tcgetattr(3)]]</tt>        | |
| <tt>[[tcgetpgrp(3)]]</tt>        | |
| <tt>[[tcsendbreak(3)]]</tt>      | |
| <tt>[[tcsetattr(3)]]</tt>        | |
| <tt>[[tcsetpgrp(3)]]</tt>        | |
| <tt>[[time(2)]]</tt>             | |
| <tt>[[timer_getoverrun(2)]]</tt> | |
| <tt>[[timer_gettime(2)]]</tt>    | |
| <tt>[[timer_settime(2)]]</tt>    | |
| <tt>[[times(2)]]</tt>            | |
| <tt>[[umask(2)]]</tt>            | |
| <tt>[[uname(2)]]</tt>            | |
| <tt>[[unlink(2)]]</tt>           | |
| <tt>[[unlinkat(2)]]</tt>         | POSIX.1-2008에서 추가 |
| <tt>[[utime(2)]]</tt>            | |
| <tt>[[utimensat(2)]]</tt>        | POSIX.1-2008에서 추가 |
| <tt>[[utimes(2)]]</tt>           | POSIX.1-2008에서 추가 |
| <tt>[[wait(2)]]</tt>             | |
| <tt>[[waitpid(2)]]</tt>          | |
| `wcpcpy(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcpncpy(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wcscat(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcscat(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcschr(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcscmp(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcscpy(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcscspn(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wcslen(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcsncat(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wcsncmp(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wcsncpy(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wcsnlen(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wcspbrk(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wcsrchr(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wcsspn(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcsstr(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wcstok(3)`                      | POSIX.1-2008 TC2에서 추가 |
| `wmemchr(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wmemcmp(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wmemcpy(3)`                     | POSIX.1-2008 TC2에서 추가 |
| `wmemmove(3)`                    | POSIX.1-2008 TC2에서 추가 |
| `wmemset(3)`                     | POSIX.1-2008 TC2에서 추가 |
| <tt>[[write(2)]]</tt>            | |

참고:

* POSIX.1-2001과 POSIX.1-2001 TC2에서는 <tt>[[fpathconf(3)]]</tt>, <tt>[[pathconf(3)]]</tt>, <tt>[[sysconf(3)]]</tt>가 비동기 시그널 안전이기를 요구했지만 POSIX.1-2008에서 이 요구가 제거되었다.

* 안전하지 않은 함수의 실행이 시그널 핸들러로 중단되고, 그 핸들러가 <tt>[[longjmp(3)]]</tt>나 <tt>[[siglongjmp(3)]]</tt> 호출을 통해 끝나고, 이후 프로그램에서 안전하지 않은 함수를 호출하는 경우에 프로그램의 동작 방식이 규정되어 있지 않다.

* POSIX.1-2001 TC1에서는 응용이 시그널 핸들러에서 <tt>[[fork(2)]]</tt>를 호출하고 <tt>[[pthread_atfork(3)]]</tt> 호출로 등록한 포크 핸들러에서 비동기 시그널 안전이 아닌 함수를 호출하는 경우 동작 방식이 규정되어 있지 않음을 분명히 하였다. 그 표준의 향후 리비전에서 <tt>[[fork(2)]]</tt>를 비동기 시그널 안전 함수 목록에서 제거할 가능성이 높다.

* 취소점인 함수를 호출하며 취소 연기 구간 상에 있는 비동기 시그널 핸들러가 취소를 유발할 수 있다. 비동기 취소가 발생한 것처럼 동작하게 되며 그로 인해 응용 상태의 무결성이 깨질 수 있다.

### `errno`

시그널 핸들러에서 진입 시 `errno`를 저장하고 반환 전 그 값을 복원시키면 `errno` 값을 가져오고 설정하는 것이 비동기 시그널 안전이다.

### GNU C 라이브러리의 일탈

GNU C 라이브러리에 다음과 같은 표준 일탈이 있다.

* glibc 2.24 전에서 <tt>[[execl(3)]]</tt>과 <tt>[[execle(3)]]</tt>이 내부적으로 <tt>[[realloc(3)]]</tt>을 이용했고 그로 인해 비동기 시그널 안전이 아니었다. glibc 2.24에서 수정되었다.

* glibc의 <tt>[[aio_suspend(3)]]</tt> 구현이 내부적으로 <tt>[[pthread_mutex_lock(3)]]</tt>을 사용하기 때문에 비동기 시그널 안전이 아니다.

## SEE ALSO

<tt>[[sigaction(2)]]</tt>, <tt>[[signal(7)]]</tt>, <tt>[[standards(7)]]</tt>

----

2021-03-22
