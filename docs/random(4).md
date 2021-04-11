## NAME

random, urandom - 커널 난수 원천 장치

## SYNOPSIS

```c
#include <linux/random.h>

int ioctl(fd, RNDrequest, param);
```

## DESCRIPTION

(리눅스 1.3.30부터 존재하는) 문자 특수 파일 `/dev/random`과 `/dev/urandom`은 커널의 난수 생성기에 대한 인터페이스를 제공한다. `/dev/random` 파일은 장치 주번호가 1이고 장치 부번호가 8이다. `/dev/urandom` 파일은 장치 주번호가 1이고 장치 부번호가 9이다.

난수 생성기는 장치 드라이버 및 기타 원천들로부터 환경 잡음을 수집하여 엔트로피 풀에 넣는다. 생성기는 또한 엔트로피 풀 내의 잡음 비트 수 추정치를 유지한다. 이 엔트로피 풀로부터 난수들을 생성한다.

리눅스 3.17 및 이후 버전에서는 더 단순하고 안전한 <tt>[[getrandom(2)]]</tt> 인터페이스를 제공한다. 여기에는 어떤 특수 파일도 필요하지 않다. 자세한 내용은 <tt>[[getrandom(2)]]</tt> 매뉴얼 페이지를 보라.

`/dev/urandom` 장치를 읽어들이면 엔트로피 풀을 시드로 쓰는 유사 난수 생성기를 이용하여 난수 바이트를 반환한다. 이 장치를 읽는 것은 블록 하지 않는다. (즉, CPU를 양보하지 않는다.) 하지만 많은 양의 데이터를 요청할 때에는 주목할 만한 지연이 발생할 수 있다.

부팅 초반에는 엔트로피 풀이 초기화 되기 전에 `/dev/urandom`이 데이터를 반환할 수도 있다. 응용에서 이 점이 염려된다면 대신 <tt>[[getrandom(2)]]</tt>이나 `/dev/random`을 사용하라.

`/dev/random` 장치는 `/dev/urandom` 구현에 쓰는 암호학적 요소를 널리 신뢰하지 않던 시기까지 거슬러 올라가는 구식 인터페이스이다. 엔트로피 풀 내의 신선한 잡음 비트 추정량 내에서만 난수 바이트를 반환하게 되며 필요하면 블록 한다. `/dev/random`은 품질 높은 난수성이 필요하면서 불확정한 지연을 감수할 수 있는 응용들에 적합하다.

엔트로피 풀이 비어 있을 때 `/dev/random`을 읽으면 추가로 환경 잡음을 수집할 때까지 블록 하게 된다. `/dev/random`에 대한 <tt>[[open(2)]]</tt>을 `O_NONBLOCK` 플래그와 함께 호출하면 후속 `read(2)`에서 요청한 수만큼의 바이트가 사용 가능하지 않은 경우에 블록 하지 않게 된다. 대신 사용 가능한 바이트들을 반환한다. 사용 가능한 바이트가 없으면 `read(2)`가 -1을 반환하며 `errno`를 `EAGAIN`으로 설정하게 된다.

`/dev/urandom`을 열 때는 `O_NONBLOCK` 플래그의 효과가 없다. `/dev/urandom` 장치에 `read(2)`를 호출할 때 256개까지의 바이트를 읽으면 요청한 만큼의 바이트들을 반환하며 시그널 핸들러에 의해 중단되지 않게 된다. 이 제한을 넘는 버퍼로 읽기를 하면 시그널 핸들러에 의해 중단되는 경우 요청한 바이트 수보다 적게 반환하거나 `EINTR` 오류로 실패할 수도 있다.

리눅스 3.16부터는 `/dev/urandom`에서 `read(2)` 하는 것이 최대 32MB를 반환하게 된다. `/dev/random`에서 `read(2)` 하는 것은 최대 512바이트를 (리눅스 커널 버전 2.6.12 전에서는 340바이트를) 반환하게 된다.

`/dev/random`이나 `/dev/urandom`에 쓰기를 하면 써넣은 데이터로 엔트로피 풀을 갱신하게 되지만 엔트로피 양이 올라가지는 않는다. 즉, 두 파일에서 읽게 되는 내용물에는 영향을 주지만 `/dev/random`에서 읽는 것이 더 빨라지지는 않는다.

### 사용법

`/dev/random` 인터페이스는 구식 인터페이스로 본다. 모든 용도에 `/dev/urandom`이 좋고 또 충분하다. 이에 대한 예외가 부팅 초기 동안 난수성이 필요한 응용들인데, 그런 응용들에서는 대신 <tt>[[getrandom(2)]]</tt>을 사용해야 한다. 그 함수는 엔트로피 풀이 초기화 될 때까지 블록 하게 된다.

아래에서 권장하듯 재부팅을 거칠 때 시드 파일을 저장하면 (최소 2000년부터 모든 주요 리눅스 배포판들이 이렇게 한다.) 부팅 과정에서 재적재하는 즉시 로컬 루트 접근권 없는 공격자에 대해 출력이 암호학적으로 안전해진다. 네트워크 암호화 세션 키에 더할 바 없이 적합하다. `/dev/random` 읽기가 블록 할 수도 있으므로 일반적으로 사용자는 논블로킹 모드로 열고서 (또는 타임아웃을 주어 읽기를 수행하고서) 원하는 엔트로피가 즉시 사용 가능하지 않으면 어떤 종류의 사용자 알림을 제공하기를 바랄 것이다.

### 구성

시스템에 `/dev/random` 및 `/dev/urandom`이 이미 생성되어 있지 않다면 다음 명령으로 생성할 수 있다.

```sh
mknod -m 666 /dev/random c 1 8
mknod -m 666 /dev/urandom c 1 9
chown root:root /dev/random /dev/urandom
```

운용자와의 상호작용이 크게 없이 리눅스 시스템이 시작할 때는 엔트로피 풀이 꽤 예측 가능한 상태일 수도 있다. 이렇게 되면 엔트로피 풀 내의 실제 잡음 양이 추산치보다 내려간다. 이 효과에 대응하려면 시스템 정지와 시작에 걸쳐서 엔트로피 풀 정보가 이어지게 하는 것이 도움이 된다. 리눅스 시스템 시작 과정에 실행되는 적절한 스크립트에 다음 행들을 추가하면 된다.

```sh
echo "Initializing random number generator..."
random_seed=/var/run/random-seed
# 시작에서 시작으로 난수 시드 이월하기
# 엔트로피 풀 전체를 적재한 다음 저장
if [ -f $random_seed ]; then
    cat $random_seed >/dev/urandom
else
    touch $random_seed
fi
chmod 600 $random_seed
poolfile=/proc/sys/kernel/random/poolsize
[ -r $poolfile ] && bits=$(cat $poolfile) || bits=4096
bytes=$(expr $bits / 8)
dd if=/dev/urandom of=$random_seed count=1 bs=$bytes
```

그리고 리눅스 시스템 정지 과정에 실행되는 절적한 스크립트에 다음 행들을 추가하면 된다.

```sh
# 정지에서 시작으로 난수 시드 이월하기
# 엔트로피 풀 전체를 저장
echo "Saving random seed..."
random_seed=/var/run/random-seed
touch $random_seed
chmod 600 $random_seed
poolfile=/proc/sys/kernel/random/poolsize
[ -r $poolfile ] && bits=$(cat $poolfile) || bits=4096
bytes=$(expr $bits / 8)
dd if=/dev/urandom of=$random_seed count=1 bs=$bytes
```

위 예시들에서는 리눅스 2.6.0 내지 이후 버전을 가정한다. 그 버전들에서는 `/proc/sys/kernel/random/poolsize`이 엔트로피 풀의 크기를 비트 단위로 반환한다 (아래 참고).

### `/proc` 인터페이스

(2.3.16부터 있는) `/proc/sys/kernel/random` 디렉터리 내의 파일들이 `/dev/random` 장치에 대한 추가 정보를 제공한다.

`entropy_avail`
:   이 읽기 전용 파일은 사용 가능 엔트로피의 양을 비트 단위로 알려 준다. 0에서 4096까지 범위의 숫자일 것이다.

`poolsize`
:   이 파일은 엔트로피 풀의 크기를 알려 준다. 커널 버전에 따라 이 파일의 해석 방식이 다르다.

    리눅스 2.4:
    :   이 파일은 *바이트* 단위로 엔트로피 풀의 크기를 알려 준다. 보통은 512 값을 가지게 되는데, 파일이 쓰기 가능하므로 가용 알고리듬이 있는 어떤 값으로도 바꿀 수 있다. 선택 가능한 값은 32, 64, 128, 256, 512, 1024, 2048이다.

    리눅스 2.6 및 이후:
    :   이 파일은 읽기 전용이며 *비트* 단위로 엔트로피 풀의 크기를 알려 준다. 4096 값을 담고 있다.

`read_wakeup_threshold`
:   이 파일은 `/dev/random`에게서 엔트로피를 기다리며 잠들어 있는 프로세스들을 깨우기 위해 필요한 엔트로피 비트 수를 담고 있다. 기본값은 64이다.

`write_wakeup_threshold`
:   이 파일은 `/dev/random` 쓰기 접근을 <tt>[[select(2)]]</tt> 내지 <tt>[[poll(2)]]</tt> 하는 프로세스들을 깨우는 엔트로피 비트 수 기준을 담고 있다. 이 기준 아래이면 깨운다. 파일에 쓰기를 해서 이 값들을 바꿀 수 있다.

`uuid` 및 `boot_id`
:   이 읽기 전용 파일들은 6fd5a44b-35f4-4ad4-a9b9-6b9be13e1fe9 같은 난수열을 담고 있다. 전자는 읽을 때마다 새로 생성하며 후자는 한 번만 생성한다.

### `ioctl(2)` 인터페이스

`/dev/random` 내지 `/dev/urandom`에 연결된 파일 디스크립터에서 다음 `ioctl(2)` 요청들이 정의되어 있다. 모든 수행 요청들은 `/dev/random`과 `/dev/urandom` 모두에 영향을 주는 입력 엔트로피 풀과 상호작용 하게 된다. `RNDGETENTCNT`를 제외한 모든 요청에는 `CAP_SYS_ADMIN` 역능이 필요하다.

`RNDGETENTCNT`
:   입력 풀의 엔트로피 양을 가져온다. 그 내용물은 proc 아래의 `entropy_avail`과 같게 된다. 인자가 가리키는 int에 결과가 저장된다.

`RNDADDTOENDCNT`
:   인자가 가리키는 값만큼 입력 풀의 엔트로피 양을 올리거나 내린다.

`RNDGETPOOL`
:   리눅스 2.6.9에서 제거됨.

`RNDADDENTROPY`
:   입력 풀에 어떤 추가 엔트로피를 더하여 엔트로피 양을 늘인다. `/dev/random`이나 `/dev/urandom`에 쓰기를 하는 것과는 다른데, 거기선 어떤 데이터를 추가하지만 엔트로피 양을 늘이지는 않는다. 다음 구조체를 사용한다.

        struct rand_pool_info {
            int    entropy_count;
            int    buf_size;
            __u32  buf[0];
        };

    여기서 `entropy_count`는 엔트로피 양에 더하는 (또는 빼는) 값이고, `buf`는 엔트로피 풀에 추가되는 `buf_size` 크기 버퍼이다.

`RNDZAPENTCNT`, `RNDCLEARPOOL`
:   모든 풀의 엔트로피 양을 0으로 만들고 그 풀들에 어떤 (시계 시간 같은) 시스템 데이터를 추가한다.

## FILES

* `/dev/random`
* `/dev/urandom`

## NOTES

난수를 얻는 데 사용할 수 있는 다양한 인터페이스들의 소개와 비교는 <tt>[[random(7)]]</tt>을 보라.

## BUGS

부팅 초반에는 엔트로피 풀이 초기화 되기 전에 `/dev/urandom` 읽기가 데이터를 반환할 수도 있다.

## SEE ALSO

`mknod(1)`, <tt>[[getrandom(2)]]</tt>, <tt>[[random(7)]]</tt>

RFC 1750, "보안을 위한 난수성 요구 사항"

----

2017-09-15
