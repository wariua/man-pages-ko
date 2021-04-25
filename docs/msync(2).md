## NAME

msync - 파일을 메모리 맵과 동기화 하기

## SYNOPSIS

```c
#include <sys/mman.h>

int msync(void *addr, size_t length, int flags);
```

## DESCRIPTION

`msync()`는 <tt>[[mmap(2)]]</tt>으로 메모리로 맵 한 파일의 코어 내 사본에 이뤄진 변경 내용을 파일 시스템으로 내린다. 이 호출을 쓰지 않으면 <tt>[[munmap(2)]]</tt>을 호출하기 전에는 변경 내용이 기록된다는 보장이 없다. 더 정확하게는 `addr`에서 시작하고 길이가 `length`인 메모리 영역에 대응하는 파일 부분을 갱신한다.

`flags` 인자에는 `MS_ASYNC`와 `MS_SYNC` 중 딱 하나를 지정해야 하며, 선택적으로 `MS_INVALIDATE` 비트까지 포함시킬 수 있다. 이 비트들의 의미는 다음과 같다.

`MS_ASYNC`
:   갱신을 예약하게 하고 호출은 즉시 반환한다.

`MS_SYNC`
:   갱신을 요청하고 완료되기를 기다린다.

`MS_INVALIDATE`
:   같은 파일의 다른 매핑들을 무효화하도록 요청한다. (그래서 방금 써넣은 최신 값들로 갱신될 수 있게 한다.)

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBUSY`
:   `flags`에 `MS_INVALIDATE`를 지정했는데 지정한 주소 범위에 메모리 잠금이 존재한다.

`EINVAL`
:   `addr`이 `PAGESIZE`의 배수가 아니다. `flags`에 `MS_ASYNC | MS_INVALIDATE | MS_SYNC` 외의 비트가 설정돼 있다. `flags`에 `MS_SYNC`와 `MS_ASYNC`가 함께 설정돼 있다.

`ENOMEM`
:   지정한 메모리가 (또는 그 일부가) 맵 되어 있지 않다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

리눅스 1.3.21에서 이 호출이 도입했을 때는 `ENOMEM` 대신 `EFAULT`를 썼다. 리눅스 2.4.19에서 POSIX 값인 `ENOMEM`으로 바뀌었다.

`msync()`가 사용 가능한 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_MAPPED_FILES`와 `_POSIX_SYNCHRONIZED_IO`가 0보다 큰 값으로 정의되어 있다. (<tt>[[sysconf(3)]]</tt>도 참고.)

## NOTES

POSIX에 따르면 `flags`에 `MS_SYNC`와 `MS_ASYNC` 중 하나를 지정해야 하며 실제로 일부 시스템에서는 그 플래그들 중 하나를 포함시키지 않으면 `msync()`가 실패하게 된다. 하지만 리눅스에서는 둘 중 어느 쪽도 지정하지 않은 `msync()` 호출을 허용하며, 그 (현재) 의미는 `MS_ASYNC`를 지정한 것과 동등하다. (리눅스 2.6.19부터는 커널에서 변경 페이지들을 올바로 추적해서 필요하면 저장소로 내려 주기 때문에 `MS_ASYNC`가 실제로는 no-op이다.) 그런 리눅스의 동작 방식에도 불구하고, 이식 가능하고 미래를 대비하는 응용에서는 `flags`에 `MS_SYNC`와 `MS_ASYNC` 중 하나를 꼭 지정해야 할 것이다.

## SEE ALSO

<tt>[[mmap(2)]]</tt>

B.O. Gallmeister, POSIX.4, O.Reilly, 128-129쪽 및 389-391쪽.

----

2021-03-22
