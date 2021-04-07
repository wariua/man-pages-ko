## NAME

stpcpy - 문자열을 복사하고 그 끝에 대한 포인터 반환하기

## SYNOPSIS

```c
#include <string.h>

char *stpcpy(char *dest, const char *src);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>stpcpy()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.10부터:</dt>
 <dd><code>_POSIX_C_SOURCE >= 200809L</code></dd>
 <dt>glibc 2.10 전:</dt>
 <dd><code>_GNU_SOURCE</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`stpcpy()` 함수는 `src`가 가리키는 문자열을 (종료용 널 바이트(`'\0'`)를 포함해) `dest`가 가리키는 배열로 복사한다. 두 문자열이 겹칠 수 없으며 대상 문자열 `dest`가 복사를 받아들일 만큼 커야 한다.

## RETURN VALUE

`stpcpy()`는 문자열 `dest`의 시작이 아니라 **끝**에 대한 포인터를 (즉, 종료용 널 바이트의 주소를) 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `stpcpy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 POSIX.1-2008에 추가되었다. 그 전에는 C나 POSIX.1 표준의 일부가 아니었고 유닉스 시스템에 통상 있는 것도 아니었다. 처음 등장한 것은 늦어도 1986년에 Lattice C Amiga-DOS 컴파일러에서였으며, 이후 1989년에 GNU fileutils와 GNU textutils에, 그리고 1992년에 GNU C 라이브러리에 등장했다. BSD 계열에도 있다.

## BUGS

이 함수는 버퍼 `dest`를 넘치게 할 수 있다.

## EXAMPLE

예를 들어 이 프로그램은 `stpcpy()`으로 **foo**와 **bar**를 이어 붙여서 **foobar**를 만들어 낸 다음 출력한다.

```c
#define _GNU_SOURCE
#include <string.h>
#include <stdio.h>

int
main(void)
{
    char buffer[20];
    char *to = buffer;

    to = stpcpy(to, "foo");
    to = stpcpy(to, "bar");
    printf("%s\n", buffer);
}
```

## SEE ALSO

`bcopy(3)`, <tt>[[memccpy(3)]]</tt>, `memcpy(3)`, `memmove(3)`, <tt>[[stpncpy(3)]]</tt>, `strcpy(3)`, <tt>[[string(3)]]</tt>, `wcpcpy(3)`

----

2019-03-06
