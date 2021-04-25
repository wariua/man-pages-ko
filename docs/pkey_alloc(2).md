## NAME

pkey_alloc, pkey_free - 보호 키 할당하고 해제하기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) */
#include <sys/mman.h>

int pkey_alloc(unsigned int flags, unsigned int access_rights);
int pkey_free(int pkey);
```

## DESCRIPTION

`pkey_alloc()`은 보호 키(pkey)를 할당한다. 그 키를 <tt>[[pkey_mprotect(2)]]</tt>에 전달할 수 있다.

`pkey_alloc()`의 `flags`는 향후 용도를 위해 예약되어 있으며 현재는 항상 0으로 지정해야 한다.

`pkey_alloc()`의 `access_rights` 인자에는 0개 이상의 비활성화 동작을 담을 수 있다.

`PKEY_DISABLE_ACCESS`
:   반환된 보호 키의 대상 메모리에 모든 데이터 접근을 못 하게 한다.

`PKEY_DISABLE_WRITE`
:   반환된 보호 키의 대상 메모리에 쓰기 접근을 못 하게 한다.

`pkey_free()`는 보호 키를 해제하고 이후 할당에 쓸 수 있게 만든다. 보호 키를 해제한 후에는 어떤 보호 키 관련 작업에도 더는 사용할 수 없다.

<tt>[[pkey_mprotect(2)]]</tt>에 의해 어느 주소 범위로 할당되어서 아직 사용 중인 보호 키에 대해 응용에서 `pkey_free()`를 호출하지 말아야 한다. 이 경우의 동작은 규정되어 있지 않으며 오류를 유발할 수도 있다.

## RETURN VALUE

성공 시 `pkey_alloc()`은 양수 보호 키 값을 반환한다. 성공 시 `pkey_free()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `pkey`, `flags`, `access_rights`가 유효하지 않다.

`ENOSPC`
:   (`pkey_alloc()`) 현재 프로세스에서 쓸 수 있는 모든 보호 키가 이미 할당되었다. 사용 가능한 키 개수는 아키텍처 및 구현에 따라 달라지며 커널 내부의 특정 키 사용 때문에 줄어들 수도 있다. 현재 x86에서는 사용자 프로그램에서 15개 키를 사용할 수 있다.

    프로세서나 운영 체제에서 보호 키를 지원하지 않는 경우에도 이 오류가 반환된다. 응용의 통제 범위 밖 요인들 때문에 사용 가능한 pkey 개수가 줄어들 수 있으므로 응용에서는 언제나 이 오류를 다룰 준비가 되어 있어야 할 것이다.

## VERSIONS

리눅스 커널 4.9에서 `pkey_alloc()`과 `pkey_free()`가 추가되었다. glibc 2.27에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`pkey_alloc()` 및 `pkey_free()` 시스템 호출은 리눅스 전용이다.

## NOTES

운영 체제가 보호 키를 지원하는지 여부와 상관없이 언제나 `pkey_alloc()`은 호출해도 안전하다. 이를 pkey 지원 탐지를 위한 다른 메커니즘 대신 사용할 수 있으며, 운영 체제에 pkey 지원이 없으면 그냥 `ENOSPC` 오류로 실패하게 된다.

커널에서는 할당된 보호 키들에 대해서만 하드웨어 권한 레지스터(PKRU)의 내용물이 보존된다고 보장한다. 키가 할당되어 있지 않을 때는 (`pkey_alloc()` 호출이 키를 반환하기 전이나 `pkey_free()`를 통해 키를 해제한 후에는) 언제든 그 키의 접근권에 영향을 주는 권한 레지스터의 내용을 커널이 임의로 바꿀 수도 있다.

## EXAMPLES

<tt>[[pkey(7)]]</tt> 참고.

## SEE ALSO

<tt>[[pkey_mprotect(2)]]</tt>, <tt>[[pkeys(7)]]</tt>

----

2021-03-22
