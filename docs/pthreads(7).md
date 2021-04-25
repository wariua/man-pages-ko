## NAME

pthreads - POSIX 스레드

## DESCRIPTION

POSIX.1에서는 스레드 방식 프로그래밍을 위한 일군의 인터페이스(함수, 헤더 파일)를 명세하는데, 보통 POSIX 스레드 내지 Pthreads라고 한다. 한 프로세스 안에 여러 스레드가 있을 수 있어서 모두 동일 프로그램을 실행한다. 그 스레드들은 동일한 전역 메모리(데이터 및 힙 세그먼트)를 공유하지만 스레드마다 각자의 스택(자동 변수)이 있다.

POSIX.1에서는 스레드들이 다양한 다른 속성들을 공유하기를 요구한다. (즉 이 속성들은 스레드 한정이 아니라 프로세스 전역이다.)

- 프로세스 ID

- 부모 프로세스 ID

- 프로세스 그룹 ID 및 세션 ID

- 제어 터미널

- 사용자 ID 및 그룹 ID

- 열린 파일 디스크립터

- 레코드 락 (<tt>[[fcntl(2)]]</tt>)

- 시그널 처리 방식

- 파일 모드 생성 마스크 (<tt>[[umask(2)]]</tt>)

- 현재 디렉터리 (<tt>[[chdir(2)]]</tt>) 및 루트 디렉터리 (<tt>[[chroot(2)]]</tt>)

- 간격 타이머 (<tt>[[setitimer(2)]]</tt>) 및 POSIX 타이머 (<tt>[[timer_create(2)]]</tt>)

- 나이스 값 (<tt>[[setpriority(2)]]</tt>)

- 자원 제한 (<tt>[[setrlimit(2)]]</tt>)

- CPU 시간 (<tt>[[times(2)]]</tt>) 및 자원 (<tt>[[getrusage(2)]]</tt>) 소모 측정치

POSIX.1에서는 스택뿐 아니라 여러 다른 속성들이 스레드별로 구분되어 있다고 명세한다.

- 스레드 ID (`pthread_t` 데이터 타입)

- 시그널 마스크 (<tt>[[pthread_sigmask(3)]]</tt>)

- `errno` 변수

- 대체 시그널 스택 (<tt>[[sigaltstack(2)]]</tt>)

- 실시간 스케줄링 정책과 우선순위 (<tt>[[sched(7)]]</tt>)

다음 리눅스 한정 속성들 역시 스레드별이다.

- 역능 (<tt>[[capabilities(7)]]</tt>)

- CPU 친화성 (<tt>[[sched_setaffinity(2)]]</tt>)

### pthreads 함수 반환 값

pthreads 함수 대부분은 성공 시 0을 반환하고 실패 시 오류 번호를 반환한다. 반환될 수 있는 오류 번호들은 전통적 시스템 호출 및 C 라이브러리 함수들에서 `errno`로 반환하는 오류 번호와 의미가 같다. pthreads 함수들이 `errno`를 설정하지 않는다는 점에 유의해야 한다. 오류를 반환할 수 있는 pthreads 함수 각각에 대해 POSIX.1-2001에서는 그 함수가 절대 `EINTR` 오류로 실패할 수 없다고 명세하고 있다.

### 스레드 ID

프로세스 내의 각 스레드에는 (`pthread_t` 타입에 저장하는) 고유의 스레드 식별자가 있다. <tt>[[pthread_create(3)]]</tt> 호출자에게 이 식별자가 반환되며 <tt>[[pthread_self(3)]]</tt>로 자기 스레드 식별자를 얻을 수 있다.

스레드 ID는 프로세스 내에서만 유일성이 보장된다. (스레드 ID를 인자로 받는 pthreads 함수 모두에서 그 ID는 정의상 호출자와 같은 프로세스 내의 스레드를 가리킨다.)

종료된 스레드가 합류되거나 분리된 스레드가 종료된 후에 시스템이 스레드 ID를 재사용할 수도 있다. POSIX에서는 "수명이 끝난 스레드 ID를 응용에서 사용하려 시도하는 경우의 동작 방식은 규정되어 있지 않다"고 한다.

### 스레드 안전 함수

스레드 안전(thread-safe) 함수란 동시에 여러 스레드에서 호출해도 안전한 (즉 어느 경우든 같은 결과를 내놓는) 함수이다.

POSIX.1-2001과 POSIX.1-2008에서는 다음 함수들을 제외하고 그 표준에서 명세하는 함수 모두가 스레드 안전이어야 한다고 요구한다.

```text
asctime()
basename()
catgets()
crypt()
ctermid() NULL 아닌 인자 준 경우
ctime()
dbm_clearerr()
dbm_close()
dbm_delete()
dbm_error()
dbm_fetch()
dbm_firstkey()
dbm_nextkey()
dbm_open()
dbm_store()
dirname()
dlerror()
drand48()
ecvt() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
encrypt()
endgrent()
endpwent()
endutxent()
fcvt() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
ftw()
gcvt() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
getc_unlocked()
getchar_unlocked()
getdate()
getenv()
getgrent()
getgrgid()
getgrnam()
gethostbyaddr() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
gethostbyname() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
gethostent()
getlogin()
getnetbyaddr()
getnetbyname()
getnetent()
getopt()
getprotobyname()
getprotobynumber()
getprotoent()
getpwent()
getpwnam()
getpwuid()
getservbyname()
getservbyport()
getservent()
getutxent()
getutxid()
getutxline()
gmtime()
hcreate()
hdestroy()
hsearch()
inet_ntoa()
l64a()
lgamma()
lgammaf()
lgammal()
localeconv()
localtime()
lrand48()
mrand48()
nftw()
nl_langinfo()
ptsname()
putc_unlocked()
putchar_unlocked()
putenv()
pututxline()
rand()
readdir()
setenv()
setgrent()
setkey()
setpwent()
setutxent()
strerror()
strsignal() [POSIX-1.2008에서 추가]
strtok()
system() [POSIX-1.2008에서 추가]
tmpnam() NULL 아닌 인자 준 경우
ttyname()
unsetenv()
wcrtomb() 마지막 인자가 NULL인 경우
wcsrtombs() 마지막 인자가 NULL인 경우
wcstombs()
wctomb()
```

### 비동기 취소 안전 함수

비동기 취소 안전(async-cancel-safe) 함수란 비동기 취소 가능성이 켜져 있는 응용에서 호출해도 안전한 함수이다. (<tt>[[pthread_setcancelstate(3)]]</tt> 참고.)

POSIX.1-2001과 POSIX.1-2008에서는 다음 함수들만 비동기 취소 안전이기를 요구한다.

```text
pthread_cancel()
pthread_setcancelstate()
pthread_setcanceltype()
```

### 취소점

POSIX.1에서는 어떤 함수들이 취소점이어야 한다고, 그리고 어떤 다른 함수들은 취소점일 수도 있다고 명세한다. 스레드가 취소 가능하고, 취소 가능성 유형이 연기이며, 스레드에 취소 요청이 대기 중이면 취소점인 함수를 호출할 때 스레드가 취소된다.

POSIX.1-2001 및/또는 POSIX.1-2008에서 다음 함수들이 취소점이기를 요구한다.

```text
accept()
aio_suspend()
clock_nanosleep()
close()
connect()
creat()
fcntl() F_SETLKW
fdatasync()
fsync()
getmsg()
getpmsg()
lockf() F_LOCK
mq_receive()
mq_send()
mq_timedreceive()
mq_timedsend()
msgrcv()
msgsnd()
msync()
nanosleep()
open()
openat() [POSIX-1.2008에서 추가]
pause()
poll()
pread()
pselect()
pthread_cond_timedwait()
pthread_cond_wait()
pthread_join()
pthread_testcancel()
putmsg()
putpmsg()
pwrite()
read()
readv()
recv()
recvfrom()
recvmsg()
select()
sem_timedwait()
sem_wait()
send()
sendmsg()
sendto()
sigpause() [POSIX.1-2001 한정 (POSIX.1-2008에서 "가능" 목록으로 이동)]
sigsuspend()
sigtimedwait()
sigwait()
sigwaitinfo()
sleep()
system()
tcdrain()
usleep() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
wait()
waitid()
waitpid()
write()
writev()
```

POSIX.1-2001 및/또는 POSIX.1-2008에 따르면 다음 함수들이 취소점일 수도 있다.

```text
access()
asctime()
asctime_r()
catclose()
catgets()
catopen()
chmod() [POSIX-1.2008에서 추가]
chown() [POSIX-1.2008에서 추가]
closedir()
closelog()
ctermid()
ctime()
ctime_r()
dbm_close()
dbm_delete()
dbm_fetch()
dbm_nextkey()
dbm_open()
dbm_store()
dlclose()
dlopen()
dprintf() [POSIX-1.2008에서 추가]
endgrent()
endhostent()
endnetent()
endprotoent()
endpwent()
endservent()
endutxent()
faccessat() [POSIX-1.2008에서 추가]
fchmod() [POSIX-1.2008에서 추가]
fchmodat() [POSIX-1.2008에서 추가]
fchown() [POSIX-1.2008에서 추가]
fchownat() [POSIX-1.2008에서 추가]
fclose()
fcntl() (모든 cmd 인자 값에 대해)
fflush()
fgetc()
fgetpos()
fgets()
fgetwc()
fgetws()
fmtmsg()
fopen()
fpathconf()
fprintf()
fputc()
fputs()
fputwc()
fputws()
fread()
freopen()
fscanf()
fseek()
fseeko()
fsetpos()
fstat()
fstatat() [POSIX-1.2008에서 추가]
ftell()
ftello()
ftw()
futimens() [POSIX-1.2008에서 추가]
fwprintf()
fwrite()
fwscanf()
getaddrinfo()
getc()
getc_unlocked()
getchar()
getchar_unlocked()
getcwd()
getdate()
getdelim() [POSIX-1.2008에서 추가]
getgrent()
getgrgid()
getgrgid_r()
getgrnam()
getgrnam_r()
gethostbyaddr() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
gethostbyname() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
gethostent()
gethostid()
gethostname()
getline() [POSIX-1.2008에서 추가]
getlogin()
getlogin_r()
getnameinfo()
getnetbyaddr()
getnetbyname()
getnetent()
getopt() (opterr가 0이 아닌 경우)
getprotobyname()
getprotobynumber()
getprotoent()
getpwent()
getpwnam()
getpwnam_r()
getpwuid()
getpwuid_r()
gets()
getservbyname()
getservbyport()
getservent()
getutxent()
getutxid()
getutxline()
getwc()
getwchar()
getwd() [POSIX.1-2001 한정 (POSIX.1-2008에서 제거)]
glob()
iconv_close()
iconv_open()
ioctl()
link()
linkat() [POSIX-1.2008에서 추가]
lio_listio() [POSIX-1.2008에서 추가]
localtime()
localtime_r()
lockf() [POSIX-1.2008에서 추가]
lseek()
lstat()
mkdir() [POSIX-1.2008에서 추가]
mkdirat() [POSIX-1.2008에서 추가]
mkdtemp() [POSIX-1.2008에서 추가]
mkfifo() [POSIX-1.2008에서 추가]
mkfifoat() [POSIX-1.2008에서 추가]
mknod() [POSIX-1.2008에서 추가]
mknodat() [POSIX-1.2008에서 추가]
mkstemp()
mktime()
nftw()
opendir()
openlog()
pathconf()
pclose()
perror()
popen()
posix_fadvise()
posix_fallocate()
posix_madvise()
posix_openpt()
posix_spawn()
posix_spawnp()
posix_trace_clear()
posix_trace_close()
posix_trace_create()
posix_trace_create_withlog()
posix_trace_eventtypelist_getnext_id()
posix_trace_eventtypelist_rewind()
posix_trace_flush()
posix_trace_get_attr()
posix_trace_get_filter()
posix_trace_get_status()
posix_trace_getnext_event()
posix_trace_open()
posix_trace_rewind()
posix_trace_set_filter()
posix_trace_shutdown()
posix_trace_timedgetnext_event()
posix_typed_mem_open()
printf()
psiginfo() [POSIX-1.2008에서 추가]
psignal() [POSIX-1.2008에서 추가]
pthread_rwlock_rdlock()
pthread_rwlock_timedrdlock()
pthread_rwlock_timedwrlock()
pthread_rwlock_wrlock()
putc()
putc_unlocked()
putchar()
putchar_unlocked()
puts()
pututxline()
putwc()
putwchar()
readdir()
readdir_r()
readlink() [POSIX-1.2008에서 추가]
readlinkat() [POSIX-1.2008에서 추가]
remove()
rename()
renameat() [POSIX-1.2008에서 추가]
rewind()
rewinddir()
scandir() [POSIX-1.2008에서 추가]
scanf()
seekdir()
semop()
setgrent()
sethostent()
setnetent()
setprotoent()
setpwent()
setservent()
setutxent()
sigpause() [POSIX-1.2008에서 추가]
stat()
strerror()
strerror_r()
strftime()
symlink()
symlinkat() [POSIX-1.2008에서 추가]
sync()
syslog()
tmpfile()
tmpnam()
ttyname()
ttyname_r()
tzset()
ungetc()
ungetwc()
unlink()
unlinkat() [POSIX-1.2008에서 추가]
utime() [POSIX-1.2008에서 추가]
utimensat() [POSIX-1.2008에서 추가]
utimes() [POSIX-1.2008에서 추가]
vdprintf() [POSIX-1.2008에서 추가]
vfprintf()
vfwprintf()
vprintf()
vwprintf()
wcsftime()
wordexp()
wprintf()
wscanf()
```

표준에서 명세하지 않은 다른 함수들을 구현에서 취소점으로 둘 수도 있다. 특히 블록 할 수 있는 비표준 함수가 있다면 아마 구현에서 취소점으로 둘 것이다. (파일을 건드릴 수 있는 함수들 대부분이 여기 포함된다.)

응용에서 비동기 취소를 이용하지 않고 있더라도 위 목록에 있는 함수를 비동기 시그널 핸들러에서 호출하면 비동기 취소와 동등한 결과를 유발할 수 있다는 점에 유의할 필요가 있다. 사용자 코드에서 비동기 취소를 고려하지 않고 있을 수 있고, 그래서 사용자 데이터 상태의 무결성이 깨질 수 있다. 따라서 취소 연기 구간에 진입할 때는 시그널을 조심해서 이용해야 한다.

### 리눅스에서 컴파일 하기

리눅스에서 Pthreads API를 사용하는 프로그램은 `cc -pthread`라고 컴파일 해야 한다.

### POSIX 스레드의 리눅스 구현

그간 리눅스의 GNU C 라이브러리에서 두 가지 스레딩 구현을 제공했다.

LinuxThreads
:   원래 Pthreads 구현이다. glibc 2.4부터는 이 구현을 더이상 지원하지 않는다.

NPTL (Native POSIX Threads Library)
:   신식 Pthreads 구현이다. LinuxThreads와 비교할 때 NPTL은 POSIX.1 명세 요구 사항들을 더 가깝게 준수하며 스레드를 다수 생성할 때 성능이 더 좋다. glibc 2.3.2부터 NPTL을 사용할 수 있으며 리눅스 2.6 커널에 있는 기능들을 필요로 한다.

둘 다 소위 1:1 방식 구현이다. 즉 각 스레드가 커널 스케줄링 항목 하나로 사상된다. 두 가지 스레딩 구현 모두 리눅스의 <tt>[[clone(2)]]</tt> 시스템 호출을 이용한다. NPTL에서는 리눅스의 <tt>[[futex(2)]]</tt> 시스템 호출을 이용해 스레드 동기화 요소들(뮤텍스, 스레드 합류 등)을 구현한다.

### LinuxThreads

이 구현의 주요 특징은 다음과 같다.

- 메인 (최초) 스레드와 프로그램에서 <tt>[[pthread_create(3)]]</tt>로 만드는 스레드에 더해서 구현에서 "관리자" 스레드를 생성한다. 이 스레드가 스레드 생성과 종료를 처리한다. (잘못해서 그 스레드를 죽이면 문제가 생길 수 있다.)

- 구현 내부에서 시그널을 이용한다. 리눅스 2.2와 이후에서는 처음 세 개 실시간 시그널을 쓴다. (<tt>[[signal(7)]]</tt> 참고.) 그 전의 리눅스 커널에서는 `SIGUSR1`과 `SIGUSR2`를 쓴다. 응용에서는 구현에서 이용하는 어떤 시그널도 사용을 피해야 한다.

- 스레드들이 프로세스 ID를 공유하지 않는다. (실질적으로 LinuxThreads의 스레드는 평상시보다 더 많은 정보를 공유하지만 공통 프로세스 ID를 공유하지는 않는 프로세스들로 구현돼 있다.) `ps(1)`에서 LinuxThreads의 스레드들(관리자 스레드 포함)은 별개 프로세스들로 보인다.

LinuxThreads 구현은 다음을 포함한 여러 부분에서 POSIX.1 명세와 다르다.

- <tt>[[getpid(2)]]</tt>를 호출하면 스레드마다 다른 값을 반환한다.

- 메인 스레드 외의 스레드에서 <tt>[[getppid(2)]]</tt>를 호출하면 관리자 스레드의 프로세스 ID를 반환한다. 원래는 그 스레드들에서의 <tt>[[getppid(2)]]</tt>가 메인 스레드에서의 <tt>[[getppid(2)]]</tt>와 같은 값을 반환해야 한다.

- 한 스레드에서 <tt>[[fork(2)]]</tt>로 새 자식 프로세스를 만드는 경우에 어느 스레드든 그 자식에 <tt>[[wait(2)]]</tt> 할 수 있어야 한다. 하지만 이 구현에서는 자식을 만든 스레드만 <tt>[[wait(2)]]</tt> 할 수 있다.

- 스레드에서 <tt>[[execve(2)]]</tt>를 호출하는 경우에 (POSIX.1의 요구대로) 다른 스레드들이 모두 종료된다. 하지만 그래서 남는 프로세스가 <tt>[[execve(2)]]</tt>를 호출한 스레드가 같은 PID를 가진다. 원래는 메인 스레드와 같은 PID를 가져야 한다.

- 스레드들이 사용자 및 그룹 ID를 공유하지 않는다. set-user-ID 프로그램에서 복잡한 문제가 생길 수 있으며 응용에서 <tt>[[seteuid(2)]]</tt> 내지 유사 함수로 크리덴셜을 바꾸면 Pthreads 함수들이 실패하게 될 수 있다.

- 스레드들이 공통 세션 ID 및 프로세스 그룹 ID를 공유하지 않는다.

- 스레드들이 <tt>[[fcntl(2)]]</tt>로 생성한 레코드 락을 공유하지 않는다.

- <tt>[[times(2)]]</tt>와 <tt>[[getrusage(2)]]</tt>가 반환하는 정보가 프로세스 전역이 아니라 스레드 한정이다.

- 스레드들이 세마포어 작업 취소 값을 공유하지 않는다. (<tt>[[semop(2)]]</tt> 참고.)

- 스레드들이 간격 타이머를 공유하지 않는다.

- 스레드들이 공통 나이스 값을 공유하지 않는다.

- POSIX.1에서는 프로세스 전체로 가는 시그널과 개별 스레드로 가는 시그널을 구별한다. POSIX.1에 따르면 프로세스 내에서 임의로 고른 한 스레드가 (가령 <tt>[[kill(2)]]</tt>로 보낸) 프로세스로 향한 시그널을 처리해야 한다. LinuxThreads에서는 프로세스로 향하는 시그널이라는 개념을 지원하지 않는다. 즉 특정 스레드로만 시그널을 보낼 수 있다.

- 스레드들에 별개의 대체 시그널 스택 설정이 있다. 하지만 새 스레드의 대체 시그널 스택 설정을 생성한 스레드에게서 복사하며, 그래서 처음에는 스레드들이 대체 시그널 스택을 공유한다. (새 스레드가 대체 시그널 스택이 정의되지 않은 채로 시작해야 한다. 두 스레드가 공유하는 대체 시그널 스택에서 동시에 시그널을 처리하면 예측 불가능한 프로그램 장애가 발생하기 쉽다.)

### NPTL

NPTL에서는 프로세스 내 모든 스레드들이 같은 스레드 그룹에 들어간다. 그리고 스레드 그룹의 모든 구성원은 같은 PID를 공유한다. NPTL에서는 관리자 스레드를 쓰지 않는다.

NPTL 내부적으로 처음 두 개 실시간 시그널을 이용한다. 즉 그 시그널들을 응용에서 사용할 수 없다. 더 자세한 내용은 <tt>[[nptl(7)]]</tt>을 보라.

NPTL에도 POSIX.1 부적합 사항이 최소 한 가지 있다.

- 스레드들이 공통 나이스 값을 공유하지 않는다.

몇 가지 NPTL 부적합 사항들은 구식 커널에서만 발생한다.

- <tt>[[times(2)]]</tt>와 <tt>[[getrusage(2)]]</tt>가 반환하는 정보가 프로세스 전역이 아니라 스레드 한정이다. (커널 2.6.9에서 수정)

- 스레드들이 자원 제한을 공유하지 않는다. (커널 2.6.10에서 수정)

- 스레드들이 간격 타이머를 공유하지 않는다. (커널 2.6.12에서 수정)

- <tt>[[setsid(2)]]</tt>로 새 세션을 시작하는 게 메인 스레드에게만 허용된다. (커널 2.6.16에서 수정)

- <tt>[[setpgid(2)]]</tt>로 프로세스를 프로세스 그룹 리더로 만드는 게 메인 스레드에게만 허용된다. (커널 2.6.16에서 수정)

- 스레드들에 별개의 대체 시그널 스택 설정이 있다. 하지만 새 스레드의 대체 시그널 스택 설정을 생성한 스레드에게서 복사하며, 그래서 처음에는 스레드들이 대체 시그널 스택을 공유한다. (커널 2.6.16에서 수정)

NPTL 구현의 다음 추가 사항에 유의하라.

- 스택 크기 연성 자원 제한(<tt>[[setrlimit(2)]]</tt>의 `RLIMIT_STACK` 설명 참고)을 *무제한* 외의 값으로 설정하면 그 값이 새 스레드의 기본 스택 크기를 규정한다. 효과가 있으려면 프로그램 실행 전에 제한을 설정해야 하는데, 셸 내장 명령 `ulimit -s`를 (C 셸에서는 `limit stacksize`를) 쓸 수 있다.

### 스레딩 구현 알아내기

glibc 2.3.2부터는 `getconf(1)` 명령을 사용해 시스템의 스레딩 구현을 알아낼 수 있다.

```text
bash$ getconf GNU_LIBPTHREAD_VERSION
NPTL 2.3.4
```

그 전의 glibc 버전에서는 다음 정도 명령이면 기본 스레딩 구현을 알아내기에 충분할 것이다.

```text
bash$ $( ldd /bin/ls | grep libc.so | awk '{print $3}' ) | \
                egrep -i 'threads|nptl'
        Native POSIX Threads Library by Ulrich Drepper et al
```

### 스레딩 구현 선택하기: `LD_ASSUME_KERNEL`

LinuxThreads와 NPTL을 모두 지원하는 glibc(즉 glibc 2.3.x)가 있는 시스템에서는 환경 변수 `LD_ASSUME_KERNEL`을 이용해 동적 링커가 선택한 기본 스레딩 구현을 바꿀 수 있다. 이 변수가 있으면 특정 커널 버전 상에서 돌고 있다고 동적 링커에서 가정하게 된다. 따라서 NPTL에 필요한 지원 사항을 제공하지 않는 커널 버전을 지정하면 LinuxThreads 사용을 강제할 수 있다. (이렇게 할 가장 가능성 높은 이유는 LinuxThreads의 어떤 비준수 동작 방식에 의존하는 (잘못된) 응용을 돌리기 위해서일 것이다.)

```text
bash$ $( LD_ASSUME_KERNEL=2.2.5 ldd /bin/ls | grep libc.so | \
                awk '{print $3}' ) | egrep -i 'threads|nptl'
        linuxthreads-0.10 by Xavier Leroy
```

## SEE ALSO

<tt>[[clone(2)]]</tt>, <tt>[[fork(2)]]</tt>, <tt>[[futex(2)]]</tt>, <tt>[[gettid(2)]]</tt>, <tt>[[proc(5)]]</tt>, <tt>[[attributes(7)]]</tt>, <tt>[[futex(7)]]</tt>, <tt>[[nptl(7)]]</tt>, <tt>[[sigevent(7)]]</tt>, <tt>[[signal(7)]]</tt>

다양한 Pthreads 매뉴얼 페이지: <tt>[[pthread_atfork(3)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_cleanup_push(3)]]</tt>, <tt>[[pthread_cond_signal(3)]]</tt>, <tt>[[pthread_cond_wait(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_detach(3)]]</tt>, <tt>[[pthread_equal(3)]]</tt>, <tt>[[pthread_exit(3)]]</tt>, <tt>[[pthread_key_create(3)]]</tt>, <tt>[[pthread_kill(3)]]</tt>, <tt>[[pthread_mutex_lock(3)]]</tt>, <tt>[[pthread_mutex_unlock(3)]]</tt>, <tt>[[pthread_mutexattr_destroy(3)]]</tt>, <tt>[[pthread_mutexattr_init(3)]]</tt>, <tt>[[pthread_once(3)]]</tt>, <tt>[[pthread_spin_init(3)]]</tt>, <tt>[[pthread_spin_lock(3)]]</tt>, <tt>[[pthread_rwlockattr_setkind_np(3)]]</tt>, <tt>[[pthread_setcancelstate(3)]]</tt>, <tt>[[pthread_setcanceltype(3)]]</tt>, <tt>[[pthread_setspecific(3)]]</tt>, <tt>[[pthread_sigmask(3)]]</tt>, <tt>[[pthread_sigqueue(3)]]</tt>, <tt>[[pthread_testcancel(3)]]</tt>

----

2021-03-22
