## NAME

posix_fadvise - 파일 데이터 접근 방식을 미리 선언하기

## SYNOPSIS

```c
#include <fcntl.h>

int posix_fadvise(int fd, off_t offset, off_t len, int advice);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`posix_fadvise()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

프로그램에서 `posix_fadvise()`를 사용해 향후 특정 패턴으로 파일 데이터에 접근하겠다는 의도를 선언할 수 있다. 그러면 커널이 적절한 최적화를 수행할 수 있게 된다.

`fd`가 가리키는 파일 내에서 `offset`에서 시작해 `len` 바이트만큼 (`len`이 0이면 파일 끝까지) 이어지는 (꼭 존재해야 하는 것은 아닌) 영역에 `advice`를 적용한다. `advice`에는 구속력이 없으며 그저 응용 입장에서의 예상일 뿐이다.

`advice`에 다음 값이 가능하다.

`POSIX_FADV_NORMAL`
:   지정한 데이터에 대한 접근 패턴에 관해 응용에서 해 줄 조언이 없음을 나타낸다. 열린 파일에 아무 조언도 주지 않으면 기본으로 상정하는 값이다.

`POSIX_FADV_SEQUENTIAL`
:   응용에서 지정한 데이터에 순차적으로 (오프셋이 작은 데이터를 먼저 읽는 식으로) 접근할 예정이다.

`POSIX_FADV_RANDOM`
:   지정한 데이터에 임의 순서로 접근하려 한다.

`POSIX_FADV_NOREUSE`
:   지정한 데이터에 한 번만 접근하려 한다.

    2.6.18 전의 커널에서 `POSIX_FADV_NOREUSE`는 `POSIX_FADV_WILLNEED`와 동작이 같았다. 버그였던 것 같다. 커널 2.6.18부터 이 플래그는 no-op이다.

`POSIX_FADV_WILLNEED`
:   지정한 데이터에 조만간 접근하려 한다.

    `POSIX_FADV_WILLNEED`는 지정한 영역을 페이지 캐시로 읽어 들이는 논블록 동작을 개시한다. 가상 메모리 부하에 따라 읽는 데이터 양을 커널이 줄일 수도 있다. (보통 몇 메가바이트 정도는 완전히 충족되고 그 이상으로는 잘 쓰지 않는다.)

`POSIX_FADV_DONTNEED`
:   지정한 데이터에 당분간은 접근하지 않을 것이다.

    `POSIX_FADV_DONTNEED`는 지정한 영역과 연계된 캐싱 된 페이지들을 해제 시도한다. 예를 들어 큰 파일을 스트리밍 할 때 유용하다. 프로그램에서 이미 사용한 데이터를 캐시에서 해제하라고 주기적으로 커널에게 요청할 수 있고, 그러면 캐시 내의 더 유용한 페이지들이 폐기되지 않을 수 있다.

    페이지 일부만 폐기하는 요청은 무시한다. 불필요한 데이터를 폐기하는 것보다 필요한 데이터를 보존하는 쪽이 중요하다. 데이터 폐기를 고려하게 하려면 `offset`과 `len`이 페이지에 정렬되어 있어야 한다.

    지정 영역 내의 변경된 페이지들을 구현체가 기반 장치로 기록하려 시도할 *수도* 있다. 하지만 이를 보장하지는 않는다. 기록 안 된 변경 페이지가 있으면 해제되지 않을 것이다. 변경된 페이지가 해제되게 하고 싶으면 응용에서 먼저 <tt>[[fsync(2)]]</tt>나 <tt>[[fdatasync(2)]]</tt>를 호출해야 할 것이다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 오류 번호를 반환한다.

## ERRORS

`EBADF`
:   `fd` 인자가 유효한 파일 디스크립터가 아니다.

`EINVAL`
:   `advice`에 유효하지 않은 값을 지정했다.

`ESPIPE`
:   지정한 파일 디스크립터가 파이프나 FIFO를 가리키고 있다. (POSIX에서 명세하는 오류는 `ESPIPE`이지만 커널 버전 2.6.16 전의 리눅스는 이 경우 `EINVAL`을 반환했다.)

## VERSIONS

리눅스 2.5.60에서 커널 지원이 처음 등장했다. 기반 시스템 호출의 이름이 `fadvise64()`였다. glibc 버전 2.2부터 래퍼 함수 `posix_fadvise()`를 통해 라이브러리 지원이 이뤄졌다.

리눅스 3.18부터 기반 시스템 호출 지원이 선택적이다. `CONFIG_ADVISE_SYSCALLS` 구성 옵션 설정에 따라 정해진다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008. 참고로 POSIX.1-2003 TC1에서 `len` 인자의 타입이 `size_t`에서 `off_t`로 바뀌었다.

## NOTES

리눅스에서 `POSIX_FADV_NORMAL`은 미리 읽기 윈도를 기반 장치별 기본값으로 설정한다. `POSIX_FADV_SEQUENTIAL`은 두 배 크기로 만들고 `POSIX_FADV_RANDOM`은 파일 미리 읽기를 완전히 끈다. 이런 변경은 지정한 영역만이 아니라 파일 전체에 영향을 끼친다. (단, 동일 파일에 대한 다른 열린 파일 핸들은 영향을 받지 않는다.)

<tt>[[proc(5)]]</tt>에서 기술하는 `/proc/sys/vm/drop_caches` 인터페이스를 통해 커널 버퍼 캐시 내용을 비울 수 있다.

파일을 열어서 <tt>[[mmap(2)]]</tt>으로 맵 하고서 그 매핑에 <tt>[[mincore(2)]]</tt>를 적용하면 파일의 어느 페이지들이 버퍼 캐시 안에 상주하고 있는지에 대한 스냅샷을 얻을 수 있다.

### C 라이브러리/커널 차이

C 라이브러리의 래퍼 함수 이름이 `posix_fadvise()`이다. 기반 시스템 호출의 이름은 `fadvise64()`이다. (일부 아키텍처에서는 `fadvise64_64()`이다.) 두 시스템 호출의 차이는 전자에서 `len` 인자의 타입이 `size_t`라고 상정하는 반면 후자에서는 `loff_t`를 기대한다는 점이다.

### 아키텍처별 차이

어떤 아키텍처에서는 64비트 인자를 적절한 레지스터 쌍에 맞춰 넣어야 한다. (자세한 내용은 <tt>[[syscall(2)]]</tt> 참고.) 그런 아키텍처에서 SYNOPSIS에 있는 `posix_fadvise()` 호출 시그너처는 `fd`와 `offset` 인자 사이 패딩으로 레지스터 하나를 낭비하게 만들 것이다. 그래서 그런 아키텍처들에서는 인자 순서를 적절히 바꾸고 나머지는 `posix_fadvise()`와 정확하게 동일한 버전을 정의한다.

예를 들어 리눅스 2.6.14부터 ARM에는 다음 시스템 호출이 있다.

```c
long arm_fadvise64_64(int fd, int advice,
                      loff_t offset, loff_t len);
```


일반적으로 응용에게는 이런 아키텍처별 세부 사항이 감춰져 있다. glibc의 `posix_fadvise()` 래퍼 함수에서 적절한 아키텍처별 시스템 호출을 불러 준다.

## BUGS

커널 2.6.6 전에서는 `len`을 0으로 지정하면 이를 "파일 끝까지 전체 바이트"로 해석하지 않고 말 그대로 "0바이트"로 해석했다.

## SEE ALSO

`fincore(1)`, <tt>[[mincore(2)]]</tt>, <tt>[[readahead(2)]]</tt>, <tt>[[sync_file_range(2)]]</tt>, <tt>[[posix_fallocate(3)]]</tt>, <tt>[[posix_madvise(3)]]</tt>

----

2018-03-06
