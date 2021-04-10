## NAME

posix_madvise - 메모리 사용 패턴에 대한 조언 주기

## SYNOPSIS

```c
#include <sys/mman.h>

int posix_madvise(void *addr, size_t len, int advice);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`posix_madvise()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

응용에서 `posix_madvise()` 함수를 사용해 `addr`에서 시작해 `len` 바이트만큼 이어지는 주소 범위 내 메모리의 예상 사용 패턴에 대해 시스템에게 조언할 수 있다. 시스템에서 자유로이 그 조언을 이용해 메모리 접근 성능을 향상시킬 수 (또는 조언을 완전히 무시할 수) 있되, `posix_madvise()` 호출이 지정 범위 내 메모리 접근의 동작 결과에는 영향을 주지 않는다.

`advice` 인자는 다음 중 하나이다.

`POSIX_MADV_NORMAL`
:   지정한 주소 범위의 메모리 사용 패턴에 관해 응용에서 해 줄 특별한 조언이 없다. 기본 동작 방식이다.

`POSIX_MADV_SEQUENTIAL`
:   응용에서 지정한 주소 범위에 순차적으로, 낮은 주소부터 높은 주소로 접근할 예정이다.

`POSIX_MADV_RANDOM`
:   응용에서 지정한 주소 범위에 임의 순서로 접근할 예정이다. 따라서 미리 읽기가 평상시보다 쓸모가 없을 수 있다.

`POSIX_MADV_WILLNEED`
:   응용에서 지정한 주소 범위에 조만간 접근할 예정이다. 따라서 미리 읽기가 이득일 수 있다.

`POSIX_MADV_DONTNEED`
:   응용에서 지정한 주소 범위에 당분간은 접근하지 않을 예정이다.

## RETURN VALUE

성공 시 `posix_madvise()`는 0을 반환한다. 실패 시 양수 오류 번호를 반환한다.

## ERRORS

`EINVAL`
:   `addr`이 시스템 페이지 크기의 배수가 아니거나 `len`이 음수이다.

`EINVAL`
:   `advice`가 유효하지 않다.

`ENOMEM`
:   지정한 범위 내의 주소 일부 내지 전체가 호출자 주소 공간 밖에 있다.

## VERSIONS

glibc 버전 2.2에서 `posix_madvise()` 지원이 처음 등장했다.

## CONFORMING TO

POSIX.1-2001.

## NOTES

POSIX.1에서는 `len`이 0일 때 구현에서 오류를 내놓는 것을 허용한다. 리눅스에서는 `len`을 0으로 지정하는 것이 (성공을 반환하는 no-op로) 허용된다.

glibc에서는 <tt>[[madvise(2)]]</tt>를 이용해 이 함수를 구현한다. 그런데 glibc 2.6부터 `POSIX_MADV_DONTNEED`를 no-op로 처리한다. 대응하는 <tt>[[madvise(2)]]</tt> 값 `MADV_DONTNEED`의 동작이 파괴적 방식이기 때문이다.

## SEE ALSO

<tt>[[madvise(2)]]</tt>, <tt>[[posix_fadvise(2)]]</tt>

----

2017-09-15
