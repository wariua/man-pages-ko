## NAME

mincore - 페이지가 메모리에 상주하고 있는지 확인하기

## SYNOPSIS

```c
#include <unistd.h>
#include <sys/mman.h>

int mincore(void *addr, size_t length, unsigned char *vec);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`mincore()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

`mincore()`는 호출 프로세스 가상 메모리의 페이지들이 코어(RAM)에 상주하고 있어서 참조 시 디스크 접근(페이지 폴트)을 유발하지 않는지 여부를 나타내는 벡터를 반환한다. 주소 `addr`에서 시작해서 `length` 바이트만큼 이어지는 페이지들에 대한 상주 정보를 커널이 반환한다.

`addr` 인자는 시스템 페이지 크기의 배수여야 한다. `length` 인자는 페이지 크기의 배수일 필요가 없다. 하지만 페이지 단위로 상주 정보를 반환하기 때문에 실질적으로 `length`가 페이지 크기의 다음 배수로 올림 된다. `sysconf(_SC_PAGESIZE)`로 페이지 크기(`PAGE_SIZE`)를 얻을 수 있다.

`vec` 인자는 적어도 `(length+PAGE_SIZE-1) / PAGE_SIZE` 바이트를 담은 배열의 포인터여야 한다. 반환 시에, 페이지가 현재 메모리에 상주하고 있으면 대응하는 바이트의 최하위 비트가 설정되고, 아니면 해제된다. (각 바이트의 다른 비트들의 설정은 규정되어 있지 않다. 그 비트들은 향후 용도를 위해 예비되어 있다.) 당연하지만 `vec`에 반환된 정보는 스냅샷일 뿐이다. 메모리에 고정되어 있지 않은 페이지는 언제든 들어오고 나갈 수 있으므로 호출 반환 시점에 `vec`의 내용이 이미 낡은 것일 수도 있다.

## RETURN VALUE

성공 시 `mincore()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EAGAIN`
:   커널에 일시적으로 자원이 부족하다.

`EFAULT`
:   `vec`이 유효하지 않은 주소를 가리키고 있다.

`EINVAL`
:   `addr`이 페이지 크기의 배수가 아니다.

`ENOMEM`
:   `length`가 `(TASK_SIZE - addr)`보다 크다. (`length`에 음수 값을 지정하면 그 값을 큰 부호 없는 정수로 해석하기 때문에 이렇게 될 수 있다.) 리눅스 2.6.11 및 이전에서는 이 경우에 `EINVAL` 오류를 반환했다.

`ENOMEM`
:   `addr`에서 `addr + length`까지에 맵 안 된 메모리가 포함되어 있다.

## VERSIONS

리눅스 2.3.99pre1 및 glibc 2.2부터 사용 가능하다.

## CONFORMING TO

`mincore()`는 POSIX.1에 명세되어 있지 않으며, 모든 유닉스 구현에서 사용 가능하지는 않다.

## BUGS

커널 2.6.21 전에서 `MAP_PRIVATE` 매핑이나 (<tt>[[remap_file_pages(2)]]</tt>로 수립한) 비선형 매핑에 대해 `mincore()`가 올바른 정보를 반환하지 않았다.

## SEE ALSO

`fincore(1)`, <tt>[[madvise(2)]]</tt>, <tt>[[mlock(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[posix_fadvise(2)]]</tt>, <tt>[[posix_madvise(3)]]</tt>

----

2021-03-22
