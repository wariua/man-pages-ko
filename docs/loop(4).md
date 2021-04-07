## NAME

loop, loop-control - 루프 장치

## SYNOPSIS

```c
#include <linux/loop.h>
```

## DESCRIPTION

루프 장치란 데이터 블록을 하드 디스크나 광학 디스크 드라이브 같은 물리적 장치가 아니라 파일 시스템 내 정규 파일의 블록이나 다른 블록 장치로 매핑 하는 블록 장치이다. 예를 들어 파일로 저장된 파일 시스템 이미지에 대한 블록 장치를 만들 수 있을 텐데, 그러면 <tt>[[mount(8)]]</tt> 명령으로 그 이미지를 마운트 할 수 있다. 즉 다음과 같이 할 수 있다.

```
$ dd if=/dev/zero of=file.img bs=1MiB count=10
$ sudo losetup /dev/loop4 file.img
$ sudo mkfs -t ext4 /dev/loop4
$ sudo mkdir /myloopdev
$ sudo mount /dev/loop4 /myloopdev
```

<tt>[[losetup(8)]]</tt>의 또 다른 예 참고.

루프 장치별로 암복호화를 위한 변환 기능을 지정할 수 있다.

루프 블록 장치는 다음 `ioctl(2)` 동작들을 제공한다.

<dl>
<dt><code>LOOP_SET_FD</code></dt>
<dd><code>ioctl(2)</code> (세 번째) 인자로 파일 디스크립터를 준 열린 파일에 루프 장치를 연계한다.</dd>

<dt><code>LOOP_CLR_FD</code></dt>
<dd>루프 장치와 파일 디스크립터의 연계를 없앤다.</dd>

<dt><code>LOOP_SET_STATUS</code></dt>
<dd>

<code>ioctl(2)</code> (세 번째) 인자를 이용해 루프 장치의 상태를 설정한다. 그 인자는 <code>loop_info</code> 구조체에 대한 포인터인데 <code>&lt;linux/loop.h&gt;</code>에 다음처럼 구조체가 정의돼 있다.

```c
struct loop_info {
    int           lo_number;            /* ioctl r/o */
    dev_t         lo_device;            /* ioctl r/o */
    unsigned long lo_inode;             /* ioctl r/o */
    dev_t         lo_rdevice;           /* ioctl r/o */
    int           lo_offset;
    int           lo_encrypt_type;
    int           lo_encrypt_key_size;  /* ioctl w/o */
    int           lo_flags;             /* ioctl r/o */
    char          lo_name[LO_NAME_SIZE];
    unsigned char lo_encrypt_key[LO_KEY_SIZE];
                                        /* ioctl w/o */
    unsigned long lo_init[2];
    char          reserved[4];
};
```

암호화 방식(<code>lo_encrypt_type</code>)은 <code>LO_CRYPT_NONE</code>, <code>LO_CRYPT_XOR</code>, <code>LO_CRYPT_DES</code>, <code>LO_CRYPT_FISH2</code>, <code>LO_CRYPT_BLOW</code>, <code>LO_CRYPT_CAST128</code>, <code>LO_CRYPT_IDEA</code>, <code>LO_CRYPT_DUMMY</code>, <code>LO_CRYPT_SKIPJACK</code>, <code>LO_CRYPT_CRYPTOAPI</code>(리눅스 2.6.0부터) 중 하나여야 한다.

<code>lo_flags</code> 필드는 비트 마스크이며 다음 플래그들 중 0개 이상을 포함할 수 있다.

 <dl>
 <dt><code>LO_FLAGS_READ_ONLY</code></dt>
 <dd>루프백 장치가 읽기 전용이다.</dd>

 <dt><code>LO_FLAGS_AUTOCLEAR</code> (리눅스 2.6.25부터)</dt>
 <dd>마지막으로 닫을 때 루프백 장치가 자동으로 없어진다.</dd>

 <dt><code>LO_FLAGS_PARTSCAN</code> (리눅스 3.2부터)</dt>
 <dd>자동 파티션 탐색을 허용한다.</dd>
 </dl>
</dd>

<dt><code>LOOP_GET_STATUS</code></dt>
<dd>루프 장치의 상태를 얻는다. <code>ioctl(2)</code> (세 번째) 인자가 <code>struct loop_info</code>에 대한 포인터여야 한다.</dd>

<dt><code>LOOP_CHANGE_FD</code> (리눅스 2.6.5부터)</dt>
<dd>루프 장치의 기반 저장소를 정수인 <code>ioctl(2)</code> (세 번째) 인자에 지정한 파일 디스크립터가 나타내는 새 파일로 바꾼다. 루프 장치가 읽기 전용이고 새 기반 저장소가 이전 기반 저장소와 크기 및 종류가 같은 경우에만 이 동작이 가능하다.</dd>

<dt><code>LOOP_SET_CAPACITY</code> (리눅스 2.6.30부터)</dt>
<dd>동작 중인 루프 장치의 크기를 바꾼다. 기반 저장소의 크기를 바꾼 다음에 이 동작을 써서 루프 드라이버가 새 크기를 알아내도록 할 수 있다. 이 동작에는 인자가 없다.</dd>
</dl>

리눅스 2.6부터 두 가지 새로운 `ioctl(2)` 동작이 있다.

<dl>
<dt><code>LOOP_SET_STATUS64</code>, <code>LOOP_GET_STATUS64</code></dt>
<dd>

위에서 설명한 <code>LOOP_SET_STATUS</code> 및 <code>LOOP_GET_STATUS</code>와 유사하되 필드가 몇 개 더 있고 일부 필드가 더 큰 <code>loop_info64</code> 구조체를 사용한다.

```c
struct loop_info64 {
    uint64_t lo_device;                   /* ioctl r/o */
    uint64_t lo_inode;                    /* ioctl r/o */
    uint64_t lo_rdevice;                  /* ioctl r/o */
    uint64_t lo_offset;
    uint64_t lo_sizelimit;/* 바이트 단위, 0 == 가능한 최대 */
    uint32_t lo_number;                   /* ioctl r/o */
    uint32_t lo_encrypt_type;
    uint32_t lo_encrypt_key_size;         /* ioctl w/o */
    uint32_t lo_flags;                    /* ioctl r/o */
    uint8_t  lo_file_name[LO_NAME_SIZE];
    uint8_t  lo_crypt_name[LO_NAME_SIZE];
    uint8_t  lo_encrypt_key[LO_KEY_SIZE]; /* ioctl w/o */
    uint64_t lo_init[2];
};
```
</dd>
</dl>

### `/dev/loop-control`

리눅스 3.1부터 커널에서 `/dev/loop-control` 장치를 제공하는데, 이를 통해 응용에서 동적으로 유휴 장치를 찾아내고 시스템에 루프 장치를 추가 및 제거할 수 있다. 그런 동작들을 수행하려면 먼저 `/dev/loop-control`을 연 다음 다음 `ioctl(2)` 동작들 중 하나를 쓰면 된다.

<dl>
<dt><code>LOOP_CTL_GET_FREE</code></dt>
<dd>사용할 유휴 루프 장치를 찾고 없으면 할당한다. 성공 시 호출 결과로 장치 번호가 반환된다. 이 동작에는 인자가 없다.</dd>

<dt><code>LOOP_CTL_ADD</code></dt>
<dd>새로운 루프 장치를 추가하며 그 장치 번호를 세 번째 <code>ioctl(2)</code> 인자에 long형 정수로 지정한다. 성공 시 호출 결과로 장치 번호가 반환된다. 그 장치가 이미 할당돼 있으면 <code>EEXIST</code> 오류로 호출이 실패한다.</dd>

<dt><code>LOOP_CTL_REMOVE</code></dt>
<dd>루프 장치를 제거하며 그 장치 번호를 세 번째 <code>ioctl(2)</code> 인자에 long형 정수로 지정한다. 성공 시 호출 결과로 장치 번호가 반환된다. 장치가 사용 중이면 <code>EBUSY</code> 오류로 호출이 실패한다.</dd>
</dl>

## FILES

<dl>
<dt><code>/dev/loop*</code></dt>
<dd>루프 블록 특수 장치 파일.</dd>
</dl>

## EXAMPLE

아래 프로그램에서는 `/dev/loop-control` 장치를 사용해 유휴 루프 장치를 찾아내고, 그 루프 장치를 열고, 장치의 기반 저장소로 사용할 파일을 열고, 루프 장치를 기반 저장소에 연계한다. 다음 셸 세션이 프로그램 사용 방식을 보여 준다.

```
$ dd if=/dev/zero of=file.img bs=1MiB count=10
10+0 records in
10+0 records out
10485760 bytes (10 MB) copied, 0.00609385 s, 1.7 GB/s
$ sudo ./mnt_loop file.img
loopname = /dev/loop5
```

### 프로그램 소스

```c
#include <fcntl.h>
#include <linux/loop.h>
#include <sys/ioctl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

int
main(int argc, char *argv[])
{
    int loopctlfd, loopfd, backingfile;
    long devnr;
    char loopname[4096];

    if (argc != 2) {
        fprintf(stderr, "Usage: %s backing-file\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    loopctlfd = open("/dev/loop-control", O_RDWR);
    if (loopctlfd == -1)
        errExit("open: /dev/loop-control");

    devnr = ioctl(loopctlfd, LOOP_CTL_GET_FREE);
    if (devnr == -1)
        errExit("ioctl-LOOP_CTL_GET_FREE");

    sprintf(loopname, "/dev/loop%ld", devnr);
    printf("loopname = %s\n", loopname);

    loopfd = open(loopname, O_RDWR);
    if (loopfd == -1)
        errExit("open: loopname");

    backingfile = open(argv[1], O_RDWR);
    if (backingfile == -1)
        errExit("open: backing-file");

    if (ioctl(loopfd, LOOP_SET_FD, backingfile) == -1)
        errExit("ioctl-LOOP_SET_FD");

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[losetup(8)]]</tt>, <tt>[[mount(8)]]</tt>

----

2019-03-06
