## NAME

gnu_get_libc_version, gnu_get_libc_release - glibc 버전과 릴리스 얻기

## SYNOPSIS

```c
#include <gnu/libc-version.h>

const char *gnu_get_libc_version(void);
const char *gnu_get_libc_release(void);
```

## DESCRIPTION

`gnu_get_libc_version()` 함수는 시스템에서 사용 가능한 glibc 버전을 나타내는 문자열을 반환한다.

`gnu_get_libc_release()` 함수는 시스템에서 사용 가능한 glibc 버전의 릴리스 상태를 나타내는 문자열을 반환한다. `stable` 같은 문자열이다.

## VERSIONS

glibc 버전 2.1에서 이 함수들이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `gnu_get_libc_version()`,<br>`gnu_get_libc_release()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 glibc 전용이다.

## EXAMPLE

실행 시 아래 프로그램은 다음과 같은 출력을 내놓게 된다.

```text
$ ./a.out
GNU libc version: 2.8
GNU libc release: stable
```

### 프로그램 소스

```c
#include <gnu/libc-version.h>
#include <stdlib.h>
#include <stdio.h>

int
main(int argc, char *argv[])
{
    printf("GNU libc version: %s\n", gnu_get_libc_version());
    printf("GNU libc release: %s\n", gnu_get_libc_release());
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[confstr(3)]]</tt>

----

2019-03-06
