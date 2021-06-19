## NAME

pread, pwrite - 파일 디스크립터의 지정한 오프셋에서 읽거나 쓰기

## SYNOPSIS

```c
#include <unistd.h>

ssize_t pread(int fd, void *buf, size_t count, off_t offset);
ssize_t pwrite(int fd, const void *buf, size_t count, off_t offset);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pread()`, `pwrite()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `|| /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200809L`

## DESCRIPTION

`pread()`는 파일 디스크립터 `fd`의 (파일 시작점 기준) 오프셋 `offset`에서 최대 `count` 개 바이트를 `buf`가 가리키는 버퍼로 읽어 들인다. 파일 오프셋은 바뀌지 않는다.

`pwrite()`는 `buf`가 가리키는 버퍼의 최대 `count` 개 바이트를 파일 디스크립터 `fd`의 오프셋 `offset`에 기록한다. 파일 오프셋은 바뀌지 않는다.

`fd`가 가리키는 파일에서 seek이 가능해야 한다.

## RETURN VALUE

성공 시 `pread()`는 읽어 들인 바이트 수(0은 파일 끝 표시)를 반환하고 `pwrite()`는 기록한 바이트 수를 반환한다.

성공한 호출에서 요청보다 적은 바이트를 이동한 것이 오류가 아니라는 점에 유의하라. (<tt>[[read(2)]]</tt> 및 <tt>[[write(2)]]</tt> 참고.)

오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`pread()`는 <tt>[[read(2)]]</tt>나 <tt>[[lseek(2)]]</tt>에 명세된 오류로 실패하여 `errno`를 설정할 수 있다. `pwrite()`는 <tt>[[write(2)]]</tt>나 <tt>[[lseek(2)]]</tt>에 명세된 오류로 실패하여 `errno`를 설정할 수 있다.

## VERSIONS

리눅스 버전 2.1.60에서 `pread()` 및 `pwrite()` 시스템 호출이 추가되었다. 그리고 2.1.69에서 i386 시스템 호출 테이블에 항목이 추가되었다. glibc 2.1에서 C 라이브러리 지원(시스템 호출이 없는 구식 커널에서 <tt>[[lseek(2)]]</tt>을 이용한 에뮬레이션 포함)이 추가되었다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`pread()` 및 `pwrite()` 시스템 호출은 다중 스레드 응용에서 특히 유용하다. 이를 통해 다른 스레드가 파일 오프셋을 바꾸는 것에 영향을 받지 않으면서 여러 스레드가 같은 파일 디스크립터에서 I/O를 수행할 수 있다.

### C 라이브러리/커널 차이

리눅스 커널 2.6에서 기반 시스템 호출의 이름이 바뀌었다. `pread()`가 `pread64()`가 되고 `pwrite()`가 `pwrite64()`가 되었다. 시스템 호출 번호는 동일하게 유지되었다. glibc의 `pread()` 및 `pwrite()` 래퍼 함수에서 이 변경 사항을 투명하게 처리해 준다.

<tt>[[syscall(2)]]</tt>에서 설명하는 이유들 때문에 일부 32비트 아키텍처에서는 이 시스템 호출들의 호출 시그너처가 다르다.

## BUGS

POSIX에서는 파일을 `O_APPEND` 플래그로 여는 것이 `pwrite()`에서 데이터를 기록하는 위치에 어떤 영향도 주지 않아야 한다고 요구한다. 하지만 리눅스에서는 파일을 `O_APPEND`로 열면 `offset` 값과 상관없이 `pwrite()`가 파일 끝에 데이터를 덧붙인다.

## SEE ALSO

<tt>[[lseek(2)]]</tt>, <tt>[[read(2)]]</tt>, <tt>[[readv(2)]]</tt>, <tt>[[write(2)]]</tt>

----

2021-03-22
