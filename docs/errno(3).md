## NAME

errno - 최근 오류 번호

## SYNOPSIS

```c
#include <errno.h>
```

## DESCRIPTION

`<errno.h>` 헤더 파일에서 정수 변수 `errno`를 정의한다. 시스템 호출들과 일부 라이브러리 함수들에서 오류 발생 시에 무엇이 잘못됐는지 나타내도록 그 변수를 설정한다.

### `errno`

호출의 반환 값이 오류를 나타낼 때만 (즉 대부분 시스템 호출에서 -1, 대부분 라이브러리 함수에서 -1 또는 NULL일 때만) `errno`의 값이 의미가 있다. 성공하는 함수에서 `errno`를 바꾸는 것이 *허용된다*. 어떤 시스템 호출이나 라이브러리 호출에서도 절대 `errno` 값을 0으로 설정하지는 않는다.

어떤 시스템 호출과 라이브러리 함수(가령 <tt>[[getpriority(2)]]</tt>)에서는 -1이 성공 시의 유효한 반환 값이다. 그런 경우에는 호출 전에 `errno`를 0으로 설정해서 성공 반환과 오류 반환을 구별할 수 있다. 호출이 반환한 상태 값이 오류가 발생했을 수도 있다는 뜻이라면 `errno`가 0 아닌 값인지 확인해 보면 된다.

`errno`는 ISO C에서 `int` 타입의 변경 가능한 lvalue라고 규정돼 있으며 명시적으로 선언해서는 안 된다. `errno`가 매크로일 수도 있기 때문이다. `errno`는 스레드 로컬이다. 그래서 한 스레드에서 그 값을 설정해도 다른 스레드의 값에 영향을 주지 않는다.

### 오류 번호와 이름

유효한 오류 번호들은 모두 양수다. `errno`에 등장할 수 있는 오류 번호 각각의 심볼 이름이 `<errno.h>` 헤더 파일에 정의돼 있다.

POSIX.1에서 명세하는 오류 이름들은 모두 서로 다른 값을 가져야 한다. 단, 예외로 `EAGAIN`과 `EWOULDBLOCK`은 같을 수도 있다. 리눅스에서 이 둘은 모든 아키텍처에서 값이 같다.

각 심볼 이름에 대응하는 오류 번호는 유닉스 시스템 종류에 따라 다르며 심지어 리눅스에서 아키텍처에 따라 다르기도 하다. 그래서 아래의 오류 이름 목록에는 번호 값이 포함돼 있지 않다. <tt>[[perror(3)]]</tt> 및 <tt>[[strerror(3)]]</tt> 함수를 쓰면 그 이름을 대응하는 텍스트 오류 메시지로 변환할 수 있다.

어떤 리눅스 시스템에서든 (`moreutils` 패키지에 포함된) `errno(1)` 명령을 쓰면 모든 오류 심볼 이름과 대응하는 오류 번호들의 목록을 얻을 수 있다.

```text
$ errno -l
EPERM 1 Operation not permitted
ENOENT 2 No such file or directory
ESRCH 3 No such process
EINTR 4 Interrupted system call
EIO 5 Input/output error
...
```

`errno(1)` 명령을 이용해 다음과 같이 개별 오류 번호와 이름을 찾아보고 오류 설명의 문자열로 오류를 검색할 수도 있다.

```text
$ errno 2
ENOENT 2 No such file or directory
$ errno ESRCH
ESRCH 3 No such process
$ errno -s permission
EACCES 13 Permission denied
```

### 오류 이름 목록

아래의 오류 심볼 이름 목록에서 여러 이름들에 다음 표시가 되어 있다.

* *POSIX.1-2001*: 그 이름을 POSIX.1-2001에서, 그리고 달리 표시돼 있지 않으면 이후 POSIX.1 버전들에서도 규정하고 있다.

* *POSIX.1-2008*: 그 이름을 POSIX.1-2008에서 규정하고 있으며 그 전의 POSIX.1 표준에는 없다.

* *C99*: 그 이름을 C99에서 규정하고 있다.

다음은 리눅스에서 정의돼 있는 오류 심볼 이름들의 목록이다.

| | |
| - | - |
| `E2BIG` | 인자 목록이 너무 긺. (POSIX.1-2001) |
| `EACCES` | 권한 거부됨. (POSIX.1-2001) |
| `EADDRINUSE` | 주소가 이미 사용 중. (POSIX.1-2001) |
| `EADDRNOTAVAIL` | 사용 가능 주소 없음. (POSIX.1-2001) |
| `EAFNOSUPPORT` | 지원하지 않는 주소 패밀리. (POSIX.1-2001) |
| `EAGAIN` | 자원이 일시적으로 사용 불가. (`EWOULDBLOCK`과 같은 값일 수도 있음.) (POSIX.1-2001) |
| `EALREADY` | 연결이 이미 진행 중. (POSIX.1-2001) |
| `EBADE` | 유효하지 않은 교환. |
| `EBADF` | 잘못된 파일 디스크립터. (POSIX.1-2001) |
| `EBADFD` | 잘못된 상태의 파일 디스크립터. |
| `EBADMSG` | 잘못된 메시지. (POSIX.1-2001) |
| `EBADR` | 유효하지 않은 요청 디스크립터. |
| `EBADRQC` | 유효하지 않은 요청 코드. |
| `EBADSLT` | 유효하지 않은 슬롯. |
| `EBUSY` | 장치 또는 자원이 사용 중. (POSIX.1-2001) |
| `ECANCELED` | 동작이 취소됐음. (POSIX.1-2001) |
| `ECHILD` | 자식 프로세스가 없음. (POSIX.1-2001) |
| `ECHRNG` | 범위를 벗어난 채널 번호. |
| `ECOMM` | 송신 중 통신 오류. |
| `ECONNABORTED` | 연결 중단됨. (POSIX.1-2001) |
| `ECONNREFUSED` | 연결 거부됨. (POSIX.1-2001) |
| `ECONNRESET` | 연결 리셋됨. (POSIX.1-2001) |
| `EDEADLK` | 자원 교착 회피했음. (POSIX.1-2001) |
| `EDEADLOCK` | 대부분 시스템에서 `EDEADLK`의 동의어. 일부 시스템(가령 리눅스의 MIPS, PowerPC, SPARC)에선 별도의 오류 코드 "파일 락킹 교착 오류"다. |
| `EDESTADDRREQ` | 목적 주소 필요함. (POSIX.1-2001) |
| `EDOM` | 수학 인자가 함수 정의역을 벗어남. (POSIX.1, C99) |
| `EDQUOT` | 디스크 쿼터 초과했음. (POSIX.1-2001) |
| `EEXIST` | 파일이 존재함. (POSIX.1-2001) |
| `EFAULT` | 잘못된 주소. (POSIX.1-2001) |
| `EFBIG` | 파일이 너무 큼. (POSIX.1-2001) |
| `EHOSTDOWN` | 호스트가 내려가 있음. |
| `EHOSTUNREACH` | 호스트에 도달 불가능함. (POSIX.1-2001) |
| `EHWPOISON` | 메모리 페이지에 하드웨어 오류가 있음. |
| `EIDRM` | 식별자가 제거됐음. (POSIX.1-2001) |
| `EILSEQ` | <p>다중 바이트 내지 확장 문자가 유효하지 않거나 불완전함. (POSIX.1, C99)</p><p>이는 glibc의 오류 설명임. POSIX.1에선 이 오류를 "유효하지 않은 바이트 열"이라고 기술함.</p> |
| `EINPROGRESS` | 동작 진행 중. (POSIX.1-2001) |
| `EINTR` | 함수 호출이 중단됐음. (POSIX.1-2001) <tt>[[signal(7)]]</tt> 참고. |
| `EINVAL` | 잘못된 인자. (POSIX.1-2001) |
| `EIO` | 입출력 오류. (POSIX.1-2001) |
| `EISCONN` | 소켓이 연결돼 있음. (POSIX.1-2001) |
| `EISDIR` | 디렉터리임. (POSIX.1-2001) |
| `EISNAM` | 이름 있는 타입 파일임. |
| `EKEYEXPIRED` | 키가 만료됐음. |
| `EKEYREJECTED` | 키가 서비스에서 거부됐음. |
| `EKEYREVOKED` | 키가 폐기됐음. |
| `EL2HLT` | 2계층 중단. |
| `EL2NSYNC` | 2계층 동기화 안 됨. |
| `EL3HLT` | 3계층 중단. |
| `EL3RST` | 3계층 리셋. |
| `ELIBACC` | 필요한 공유 라이브러리에 접근할 수 없음. |
| `ELIBBAD` | 접근하려는 공유 라이브러리에 오류. |
| `ELIBMAX` | 너무 많은 공유 라이브러리를 링크하려 시도. |
| `ELIBSCN` | a.out의 .lib 섹션에 오류. |
| `ELIBEXEC` | 공유 라이브러리를 직접 실행할 수 없음. |
| `ELNRANGE` | 링크 수가 범위를 초과. |
| `ELOOP` | 너무 긴 심볼릭 링크 단계. (POSIX.1-2001) |
| `EMEDIUMTYPE` | 잘못된 매체 유형. |
| `EMFILE` | 열린 파일 너무 많음. (POSIX.1-2001) 보통 <tt>[[getrlimit(2)]]</tt>에 설명된 `RLIMIT_NOFILE` 자원 제한 초과 때문이다. `/proc/sys/fs/nr_open`에 지정된 제한 초과 때문일 수도 있다. |
| `EMLINK` | 링크 너무 많음. (POSIX.1-2001) |
| `EMSGSIZE` | 메시지가 너무 긺. (POSIX.1-2001) |
| `EMULTIHOP` | 멀티홉 시도했음. (POSIX.1-2001) |
| `ENAMETOOLONG` | 파일 이름이 너무 긺. (POSIX.1-2001) |
| `ENETDOWN` | 네트워크가 내려가 있음. (POSIX.1-2001) |
| `ENETRESET` | 연결이 네트워크에 의해 중단됨. (POSIX.1-2001) |
| `ENETUNREACH` | 네트워크에 도달 불가능. (POSIX.1-2001) |
| `ENFILE` | 시스템에 열린 파일 너무 많음. (POSIX.1-2001) 리눅스에선 아마 `/proc/sys/fs/file-max` 제한(<tt>[[proc(5)]]</tt> 참고)에 걸려서. |
| `ENOANO` | anode 없음. |
| `ENOBUFS` | 가용 버퍼 공간 없음. (POSIX.1 (XSI STREAMS 옵션)) |
| `ENODATA` | STREAM 헤드 읽기 큐에 가용 메시지 없음. (POSIX.1-2001) |
| `ENODEV` | 그런 장치 없음. (POSIX.1-2001) |
| `ENOENT` | <p>그런 파일 내지 디렉터리 없음. (POSIX.1-2001)</p><p>보통 지정한 경로명이 존재하지 않거나, 경로명 디렉터리 선두부의 어느 부분이 존재하지 않거나, 지정한 경로명이 깨진 심볼릭 링크일 때 이 오류가 발생한다.</p> |
| `ENOEXEC` | 실행 파일 형식 오류. (POSIX.1-2001) |
| `ENOKEY` | 필요한 키가 없음. |
| `ENOLCK` | 쓸 수 있는 락 없음. (POSIX.1-2001) |
| `ENOLINK` | 링크가 손상됐음. (POSIX.1-2001) |
| `ENOMEDIUM` | 매체를 찾지 못했음. |
| `ENOMEM` | 공간 부족/메모리 할당할 수 없음. (POSIX-1.2001) |
| `ENOMSG` | 원하는 유형의 메시지 없음. (POSIX.1-2001) |
| `ENONET` | 머신이 네트워크 상에 있지 않음. |
| `ENOPKG` | 패키지 설치 안 됐음. |
| `ENOPROTOOPT` | 가용 프로토콜 없음. (POSIX.1-2001) |
| `ENOSPC` | 장치에 남은 공간 없음. (POSIX.1-2001) |
| `ENOSR` | STREAM 자원 없음. (POSIX.1 (XSI STREAMS 옵션)) |
| `ENOSTR` | STREAM 아님. (POSIX.1 (XSI STREAMS 옵션)) |
| `ENOSYS` | 기능 구현 안 돼 있음. (POSIX.1-2001) |
| `ENOTBLK` | 블록 장치 필요함. |
| `ENOTCONN` | 소켓 연결 안 돼 있음. (POSIX.1-2001) |
| `ENOTDIR` | 디렉터리 아님. (POSIX.1-2001) |
| `ENOTEMPTY` | 디렉터리가 비어 있지 않음. (POSIX.1-2001) |
| `ENOTRECOVERABLE` | 상태 복원 불가능. (POSIX.1-2008) |
| `ENOTSOCK` | 소켓 아님. (POSIX.1-2001) |
| `ENOTSUP` | 지원 안 되는 동작. (POSIX.1-2001) |
| `ENOTTY` | 부적절한 I/O 제어 동작. (POSIX.1-2001) |
| `ENOTUNIQ` | 이름이 네트워크에서 유일하지 않음. |
| `ENXIO` |  그런 장치 내지 주소 없음. (POSIX.1-2001) |
| `EOPNOTSUPP` | <p>소켓에서 지원 안 되는 동작. (POSIX.1-2001)</p><p>(리눅스에선 `ENOTSUP`과 `EOPNOTSUPP`의 값이 같지만 POSIX.1에 따르면 두 오류 값이 구별돼야 한다.)</p> |
| `EOVERFLOW` | 값이 너무 커서 데이터 타입에 저장 못함. (POSIX.1-2001) |
| `EOWNERDEAD` | 소유자가 죽었음. (POSIX.1-2008) |
| `EPERM` | 동작이 허용되지 않음. (POSIX.1-2001) |
| `EPFNOSUPPORT` | 지원하지 않은 프로토콜 패밀리. |
| `EPIPE` | 파이프 깨졌음. (POSIX.1-2001) |
| `EPROTO` | 프로토콜 오류. (POSIX.1-2001) |
| `EPROTONOSUPPORT` | 지원하지 않는 프로토콜. (POSIX.1-2001) |
| `EPROTOTYPE` | 소켓에 맞지 않는 프로토콜 타입. (POSIX.1-2001) |
| `ERANGE` | 결과가 너무 큼. (POSIX.1, C99) |
| `EREMCHG` | 원격 주소가 바뀌었음. |
| `EREMOTE` | 객체가 원격에 있음. |
| `EREMOTEIO` | 원격 I/O 오류. |
| `ERESTART` | 중단된 시스템 호출을 재시작해야 함. |
| `ERFKILL` | RF-kill 때문에 동작 불가능. |
| `EROFS` | 읽기 전용 파일 시스템. (POSIX.1-2001) |
| `ESHUTDOWN` | 전송 종단 shutdown 후 보낼 수 없음. |
| `ESPIPE` | 위치 이동 불가능. (POSIX.1-2001) |
| `ESOCKTNOSUPPORT` | 지원하지 않는 소켓 유형. |
| `ESRCH` | 그런 프로세스 없음. (POSIX.1-2001) |
| `ESTALE` | <p>파일 핸들이 더는 유효하지 않음. (POSIX.1-2001)</p><p>NFS 등의 파일 시스템에서 이 오류가 발생할 수 있다.</p> |
| `ESTRPIPE` | 스트림 파이프 오류. |
| `ETIME` | <p>타이머 만료됨. (POSIX.1 (XSI STREAMS 옵션))</p><p>(POSIX.1에선 "STREAM <tt>[[ioctl(2)]]</tt> 타임아웃"이라고 함.</p> |
| `ETIMEDOUT` | 연결이 타임아웃됨. (POSIX.1-2001) |
| `ETOOMANYREFS` | 참조가 너무 많음. splice 할 수 없음. |
| `ETXTBSY` | 텍스트 파일 사용 중. (POSIX.1-2001) |
| `EUCLEAN` | 구조 정리 필요. |
| `EUNATCH` | 프로토콜 드라이버 붙어 있지 않음. |
| `EUSERS` | 사용자가 너무 많음. |
| `EWOULDBLOCK` | 동작이 블록하게 됨. (`EAGAIN`과 같은 값일 수도 있음.) (POSIX.1-2001) |
| `EXDEV` | 잘못된 링크. (POSIX.1-2001) |
| `EXFULL` | 교환기에 여유 없음. |

## NOTES

많이 하는 실수로 다음과 같은 게 있다.

```c
if (somecall() == -1) {
    printf("somecall() failed\n");
    if (errno == ...) { ... }
}
```

여기서 `errno`가 반드시 `somecall()` 함수 반환 시의 값을 가지고 있는 게 아니다. (<tt>[[printf(3)]]</tt>에 의해 바뀌었을 수도 있다.) 라이브러리 호출을 거치면서 `errno` 값이 유지돼야 한다면 저장을 해야 한다.

```c
if (somecall() == -1) {
    int errsv = errno;
    printf("somecall() failed\n");
    if (errsv == ...) { ... }
}
```

POSIX 스레드 API에서는 오류 시 `errno`를 설정하지 *않는다는* 점에 유의하라. 실패 시 함수 결과로 오류 번호를 반환한다. 그 오류 번호는 다른 API에서 `errno`로 반환하는 오류 번호와 의미가 같다.

일부 아주 오래된 시스템에서는 `<errno.h>`가 존재하지 않거나 `errno`를 선언해 주지 않았으며, 그래서 `errno`를 직접 선언(`extern int errno`)해야 했다. 지금은 **그렇게 해선 안 된다.** 오래 전부터 그럴 필요가 없게 됐으며 최근의 C 라이브러리 버전들에서 문제를 일으키게 된다.

## SEE ALSO

`errno(1)`, <tt>[[err(3)]]</tt>, <tt>[[error(3)]]</tt>, <tt>[[perror(3)]]</tt>, <tt>[[strerror(3)]]</tt>

----

2021-03-22
