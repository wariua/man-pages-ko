## NAME

posix_fallocate - 파일 공간 할당하기

## SYNOPSIS

```c
#include <fcntl.h>

int posix_fallocate(int fd, off_t offset, off_t len);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`posix_fallocate()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`posix_fallocate()` 함수는 파일 디스크립터 `fd`가 가리키는 파일에서 `offset`부터 `len` 바이트만큼 이어지는 범위의 바이트들을 위한 디스크 공간이 할당돼 있도록 만든다. `posix_fallocate()` 호출 성공 후에는 지정 범위 내 바이트들에 대한 쓰기 동작이 디스크 공간 부족 때문에 실패하지 않는 게 보장된다.

파일 크기가 `offset`+`len`보다 작으면 파일이 그 크기로 늘어난다. 아니면 파일 크기가 바뀌지 않는다.

## RETURN VALUE

`posix_fallocate()`는 성공 시 0을 반환하며 실패 시 오류 번호를 반환한다. `errno`를 설정하지 않는 점에 유의하라.

## ERRORS

`EBADF`
:   `fd`가 유효한 파일 디스크립터가 아니거나 쓰기 가능하게 열려 있지 않다.

`EFBIG`
:   `offset+len`이 최대 파일 크기를 초과한다.

`EINTR`
:   실행 중 시그널을 잡았다.

`EINVAL`
:   `offset`이 0보다 작거나, `len`이 0 이하이거나, 하위 파일 시스템에서 그 동작을 지원하지 않는다.

`ENODEV`
:   `fd`가 정규 파일을 가리키고 있지 않다.

`ENOSPC`
:   `fd`가 가리키는 파일을 담은 장치에 충분한 공간이 남아 있지 않다.

`EOPNOTSUPP`
:   `fd`가 가리키는 파일을 포함하는 파일 시스템에서 이 동작을 지원하지 않는다. NOTES에서 설명하는 에뮬레이션을 수행하지 않는 musl libc 같은 C 라이브러리에서 이 오류 코드를 반환할 수 있다.

`ESPIPE`
:   `fd`가 파이프를 가리키고 있다.

## VERSIONS

glibc 2.1.94부터 `posix_fallocate()`가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `posix_fallocate()` | 스레드 안전성 | MT-Safe (하지만 NOTES 참고) |

## CONFORMING TO

POSIX.1-2001.

POSIX.1-2008에 따르면 `len`이 0이거나 `offset`이 0보다 작은 경우 구현체에서 `EINVAL` 오류를 *내놓아야 한다*. POSIX.1-2001에 따르면 `len`이 0보다 작거나 `offset`이 0보다 작은 경우 구현체에서 `EINVAL` 오류를 *내놓아야 하며*, `len`이 0과 같은 경우 오류를 *내놓을 수 있다*.

## NOTES

glibc 구현에서 `posix_fallocate()`는 <tt>[[fallocate(2)]]</tt> 시스템 호출을 써서 구현돼 있으며 그 시스템 호출은 MT-safe이다. 하위 파일 시스템에서 <tt>[[fallocate(2)]]</tt>를 지원하지 않는 경우에는 동작을 에뮬레이션하는데, 몇 가지 유의 사항이 있다.

* 에뮬레이션이 비효율적이다.

* 경쟁 조건이 있어서 다른 스레드나 프로세스에서 동시에 기록한 내용이 널 바이트로 덮어 써질 수도 있다.

* 경쟁 조건이 있어서 다른 스레드나 프로세스에서 동시에 파일 크기를 늘이면 파일 크기가 예상보다 작게 나올 수도 있다.

* `fd`를 `O_APPEND` 플래그나 `O_WRONLY` 플래그로 열었으면 함수가 `EBADF`로 실패한다.

전반적으로 그 에뮬레이션은 MT-safe가 아니다. 응용에서 이런 에뮬레이션 유의 사항들을 용인할 수 없는 경우 리눅스에서는 <tt>[[fallocate(2)]]</tt>를 쓸 수도 있다. 일반적으로 이 방법은 `EOPNOTSUPP` 반환 시 응용 동작을 중단할 생각일 때만 권장할 만하다. 그렇지 않다면 응용 자체에서 대비책을 구현해야 할 텐데, 그러면 glibc의 에뮬레이션와 같은 문제들이 그대로 발생하게 된다.

## SEE ALSO

`fallocate(1)`, <tt>[[fallocate(2)]]</tt>, <tt>[[lseek(2)]]</tt>, <tt>[[posix_fadvise(2)]]</tt>

----

2021-03-22
