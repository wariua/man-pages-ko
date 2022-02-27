## NAME

strcat, strncat - 두 문자열 이어 붙이기

## SYNOPSIS

```c
#include <string.h>

char *strcat(char *restrict dest, const char *restrict src);
char *strncat(char *restrict dest, const char *restrict src, size_t n);
```

## DESCRIPTION

`strcat()` 함수는 문자열 `src`를 문자열 `dest`에 덧붙인다. `dest` 끝에 있던 종료 널 문자열(`'\0'`)을 덮어 쓰고 새 종료 널 바이트를 추가한다. 두 문자열이 겹쳐선 안 되며 `dest` 문자열에 결과를 담을 충분한 공간이 있어야 한다. `dest`가 충분히 크지 않을 때 프로그램 동작은 예측 불가능하다. *버퍼 넘침은 보호된 프로그램들을 공격하는 흔한 통로다.*

`strncat()` 함수는 그와 비슷하되,

* `src`에서 최대 `n`개 바이트만 이용하며,

* `src`에 `n`개 이상의 바이트가 있는 경우 널 종료일 필요가 없다.

`strcat()`처럼 `dest`의 결과 문자열은 항상 널로 끝난다.

`src`에 `n`개 이상의 바이트가 있는 경우 `strncat()`은 `n+1`개 바이트(`src`의 `n`개와 종료 널 바이트)를 `dest`에 써넣는다. 따라서 `dest`의 크기가 최소 `strlen(dest)+n+1`이어야 한다.

`strncat()`을 단순하게 구현한다면 다음처럼 될 수 있다.

```c
char *
strncat(char *dest, const char *src, size_t n)
{
    size_t dest_len = strlen(dest);
    size_t i;

    for (i = 0 ; i < n && src[i] != '\0' ; i++)
        dest[dest_len + i] = src[i];
    dest[dest_len + i] = '\0';

    return dest;
}
```

## RETURN VALUE

`strcat()` 및 `strncat()` 함수는 결과 문자열 `dest`에 대한 포인터를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strcat()`, `strncat()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

일부 시스템(BSD 계열, 솔라리스 등)에선 다음 함수를 제공한다.

```c
size_t strlcat(char *dest, const char *src, size_t size);
```

이 함수는 널 종료 문자열 `src`를 문자열 `dest`에 덧붙이되 `src`에서 최대 `size-strlen(dest)-1`만큼 복사하고 결과에 널 종료 바이트를 추가한다. `size`는 `strlen(dest)`보다 작지 않아야 한다. `strcat()`의 버퍼 넘침 문제가 고쳐지긴 하지만 호출자는 `size`가 너무 작은 경우 데이터 유실 가능성을 여전히 처리해야 한다. `strlcat()`에서 만들려고 한 문자열의 길이를 반환한다. 반환 값이 `size`와 같거나 그보다 크다면 데이터 유실이 일어난 것이다. 데이터 유실이 문제가 되는 경우라면 *반드시* 호출 전에 인자들을 확인하거나 함수 반환 값을 검사해야 한다. `strlcat()`은 glibc에 없으며 POSIX에도 표준화돼 있지 않다. 하지만 리눅스에서 `libbsd` 라이브러리를 통해 이용 가능하다.

## EXAMPLES

`strcat()`과 `strncat()`에서 `dest` 문자열 끝의 널 바이트를 찾기 위해 문자열 시작점부터 탐색을 하기 때문에 문자열 `dest`의 길이에 따라 이 함수들의 실행 시간이 증가한다. 아래 프로그램을 실행해서 확인할 수 있다. (한 대상에 많은 문자열을 이어 붙이는 게 목적이라면 각 문자열의 바이트들을 복사하면서 대상 문자열 끝 포인터를 따로 관리하는 방식이 성능이 더 나을 것이다.)

### 프로그램 소스

```c
#include <stdint.h>
#include <string.h>
#include <time.h>
#include <stdio.h>

int
main(int argc, char *argv[])
{
#define LIM 4000000
    char p[LIM + 1];    /* 종료 널 바이트 때문에 +1 */
    time_t base;

    base = time(NULL);
    p[0] = '\0';

    for (int j = 0; j < LIM; j++) {
        if ((j % 10000) == 0)
            printf("%d %jd\n", j, (intmax_t) (time(NULL) - base));
        strcat(p, "a");
    }
}
```

## SEE ALSO

<tt>[[bcopy(3)]]</tt>, <tt>[[memccpy(3)]]</tt>, <tt>[[memcpy(3)]]</tt>, <tt>[[strcpy(3)]]</tt>, <tt>[[string(3)]]</tt>, <tt>[[strncpy(3)]]</tt>, `wcscat(3)`, `wcsncat(3)`

----

2021-03-22
