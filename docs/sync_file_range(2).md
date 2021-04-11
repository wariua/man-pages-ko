## NAME

sync_file_range - 파일 조각을 디스크로 동기화하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <fcntl.h>

int sync_file_range(int fd, off64_t offset, off64_t nbytes,
                    unsigned int flags);
```

## DESCRIPTION

`sync_file_range()`는 파일 디스크립터 `fd`가 가리키는 열린 파일을 디스크와 동기화하면서 세밀한 제어를 할 수 있게 해 준다.

`offset`은 동기화할 파일 범위의 시작 바이트이다. `nbytes`는 동기화할 범위의 길이를 바이트 단위로 지정한다. `nbytes`가 0이면 `offset`부터 파일 끝까지의 모든 바이트를 동기화한다. 동기화는 시스템 페이지 크기를 단위로 이뤄진다. `offset`을 페이지 경계로 내리고 `(offset+nbytes-1)`을 페이지 경계로 올린다.

`flags` 비트 마스크 인자는 다음 값들을 포함할 수 있다.

`SYNC_FILE_RANGE_WAIT_BEFORE`
:   쓰기를 수행하기 전에 이미 장치 드라이버에 쓰기를 위해 제출돼 있는 지정 범위 내 모든 페이지들의 쓰기가 이뤄지기를 기다린다.

`SYNC_FILE_RANGE_WRITE`
:   현재 쓰기를 위해 제출돼 있지 않은 지정 범위 내 변경 페이지 모두의 쓰기를 개시한다. 요청 큐 크기를 넘게 쓰려고 하면 블록 할 수도 있음에 유의하라.

`SYNC_FILE_RANGE_WAIT_AFTER`
:   쓰기를 수행한 후에 범위 내 모든 페이지들의 쓰기가 이뤄지기를 기다린다.

`flags`를 0으로 지정하는 것이 가능하다. no-op이다.

### 경고

이 시스템 호출은 극히 위험하며 이식성 있는 프로그램에서는 사용하지 말아야 한다. 이 동작들 어느 것도 파일의 메타데이터를 기록하지 않는다. 따라서 응용에서 엄격하게 이미 실체화된 디스크 블록 덮어쓰기만 수행하는 경우가 아니면 크래시 후에 데이터가 사용 가능하다는 보장이 없다. 하지만 쓰기가 순수하게 덮어쓰기인지 알아낼 수 있는 사용자 인터페이스는 없다. 그리고 copy-on-write 동작 방식의 파일 시스템(가령 `btrfs`)에서는 기존에 할당된 블록을 덮어쓰는 게 불가능하다. 또 기할당 공간에 쓰기를 할 때 많은 파일 시스템에서 블록 할당자 호출까지 필요로 하는데 이 시스템 호출은 그 블록을 디스크로 동기화하지 않는다. 이 시스템 호출은 디스크 쓰기 캐시를 플러시 하지 않으며, 따라서 디스크 쓰기 캐시가 휘발성인 시스템에서 데이터 무결성을 제공하지 않는다.

### 몇 가지 세부 사항

`SYNC_FILE_RANGE_WAIT_BEFORE` 및 `SYNC_FILE_RANGE_WAIT_AFTER`는 I/O 오류나 `ENOSPC` 조건이 발생하면 탐지하여 호출자에게 반환해 준다.

유용한 `flags` 비트 조합은 다음과 같다.

`SYNC_FILE_RANGE_WAIT_BEFORE | SYNC_FILE_RANGE_WRITE`
:   `sync_file_range()` 호출 때 지정 범위 내에 있던 변경 페이지들 모두가 쓰기 목록 하에 있도록 한다. "데이터 무결성 위한 쓰기 개시" 동작이다.

`SYNC_FILE_RANGE_WRITE`
:   현재 쓰기 목록 하에 있지 않은 지정 범위 내 변경 페이지들 모두의 쓰기를 시작한다. 비동기적인 "디스크로 플러시" 동작이다. 데이터 무결성 동작에는 적합하지 않다.

`SYNC_FILE_RANGE_WAIT_BEFORE` (또는 `SYNC_FILE_RANGE_WAIT_AFTER`)
:   지정 범위 내 모든 페이지들의 쓰기 완료를 기다린다. 앞서 나온 `SYNC_FILE_RANGE_WAIT_BEFORE | SYNC_FILE_RANGE_WRITE` 동작 후에 사용해서 그 동작의 완료를 기다리고 그 결과를 얻을 수 있다.

`SYNC_FILE_RANGE_WAIT_BEFORE | SYNC_FILE_RANGE_WRITE | SYNC_FILE_RANGE_WAIT_AFTER`
:   `sync_file_range()` 호출 때 지정 범위 내에 있던 변경 페이지들 모두가 디스크로 가도록 하는 "데이터 무결성 위한 쓰기 개시" 동작이다.

## RETURN VALUE

성공 시 `sync_file_range()`는 0을 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니다.

`EINVAL`
:   `flags`에 유효하지 않은 비트를 지정했다. 또는 `offset`이나 `nbytes`가 유효하지 않다.

`EIO`
:   I/O 오류.

`ENOMEM`
:   메모리 부족.

`ENOSPC`
:   디스크 공간 부족.

`ESPIPE`
:   `fd`가 정규 파일, 블록 장치, 디렉터리 아닌 뭔가를 가리키고 있다.

## VERSIONS

리눅스 커널 2.6.17에서 `sync_file_range()`가 처음 등장했다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이므로 이식성 있는 프로그램에서는 피해야 한다.

## NOTES

### `sync_file_range2()`

어떤 아키텍처(가령 PowerPC, ARM)에서는 64비트 인자를 적절한 레지스터 쌍에 맞춰 넣어야 한다. 그런 아키텍처에서 SYNOPSIS에 있는 `sync_file_range()` 호출 시그너처는 `fd`와 `offset` 인자 사이 패딩으로 레지스터 하나를 낭비하게 만들 것이다. (자세한 내용은 <tt>[[syscall(2)]]</tt> 참고.) 그래서 그런 아키텍처들에서는 인자 순서를 적절히 바꾼 다른 시스템 호출을 정의한다.

```c
int sync_file_range2(int fd, unsigned int flags,
                     off64_t offset, off64_t nbytes);
```

이 시스템 호출의 동작은 그 외 부분에서는 `sync_file_range()`와 정확히 같다.

리눅스 2.6.20에서 ARM 아키텍처에 `arm_sync_file_range()`라는 이름으로 이 시그너처의 시스템 호출이 처음 등장했다. 리눅스 2.6.22에서 PowerPC에 비슷한 시스템 호출이 추가되면서 이름이 바뀌었다. glibc 지원이 제공되는 아키텍처에서는 glibc가 `sync_file_range()` 이름 하에 투명하게 `sync_file_range2()`를 감싸 준다.

## SEE ALSO

<tt>[[fdatasync(2)]]</tt>, <tt>[[fsync(2)]]</tt>, <tt>[[msync(2)]]</tt>, <tt>[[sync(2)]]</tt>

----

2021-03-22
