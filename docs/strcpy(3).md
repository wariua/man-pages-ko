## NAME

strcpy, strncpy - 문자열 복사하기

## SYNOPSIS

```c
#include <string.h>

char *strcpy(char *restrict dest, const char *src);
char *strncpy(char *restrict dest, const char *restrict src, size_t n);
```

## DESCRIPTION

`strcpy()` 함수는 `src`가 가리키는 문자열을 (종료용 널 바이트(`'\0'`)를 포함해) `dest`가 가리키는 배열로 복사한다. 두 문자열이 겹칠 수 없으며 대상 문자열 `dest`가 복사를 받아들일 만큼 커야 한다. *버퍼 넘침에 주의하라!* (BUGS 참고.)

`strncpy()` 함수는 그와 비슷하되 `src`에서 최대 `n` 바이트만 복사한다. **경고**: `src`의 처음 `n` 바이트에 널 바이트가 없으면 `dest`에 들어간 문자열이 널로 종료되지 않게 된다.

`src`의 길이가 `n`보다 작으면 `dest`에 널 바이트들을 추가로 채워서 총 `n` 바이트가 기록되게 한다.

`strncpy()`를 단순하게 구현한다면 다음처럼 될 수 있다.

```c
char *
strncpy(char *dest, const char *src, size_t n)
{
    size_t i;

    for (i = 0; i < n && src[i] != '\0'; i++)
        dest[i] = src[i];
    for ( ; i < n; i++)
        dest[i] = '\0';

    return dest;
}
```

## RETURN VALUE

`strcpy()` 및 `strncpy()` 함수는 대상 문자열 `dest`에 대한 포인터를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strcpy()`, `strncpy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

일부 프로그래머들은 `strncpy()`가 비효율적이고 오용되기 쉽다고 본다. `dest`의 크기가 `src` 길이보다 크다는 걸 알고 있다면 (즉 검사 코드를 포함시켰다면) `strcpy()`를 쓸 수 있다.

유효한 (그리고 의도에 맞는) `strcpy()` 용도 하나는 고정 길이 버퍼로 C 문자열을 복사하면서 버퍼가 넘치지 않게 하면서 (가령 그 버퍼를 매체로 기록하거나 프로세스간 통신 기법을 통해 다른 프로세스로 전송하는 경우 정보 누출을 방지하기 위해) 대상 버퍼의 안 쓰는 바이트들을 0으로 덮어 쓰게도 하는 경우다.

`src`의 처음 `n` 개 바이트에 종료 널 바이트가 없으면 `strncpy()`로 인해 `dest`에 종료 안 된 문자열이 들어간다. 길이가 `buflen`인 `buf`가 있다면 다음 같은 식으로 해서 강제로 종료시킬 수 있다.

```c
if (buflen > 0) {
    strncpy(buf, str, buflen - 1);
    buf[buflen - 1]= '\0';
}
```

(물론 위 기법에서는 `src`에 `buflen - 1` 개 넘는 바이트가 있으면 `dest`로 복사하는 과정에서 정보가 유실된다는 점을 무시하고 있다.)

### `strlcpy()`

일부 시스템(BSD 계열, 솔라리스 등)에선 다음 함수를 제공한다.

```c
size_t strlcpy(char *dest, const char *src, size_t size);
```

이 함수는 `strncpy()`와 비슷하되 `dest`로 최대 `size-1` 바이트를 복사하고 항상 종료 널 바이트를 붙인다. 그리고 대상 버퍼를 (추가) 널 바이트들로 채우지 않는다. `strcpy()` 및 `strncpy()`의 일부 문제들이 고쳐지긴 하지만 호출자는 `size`가 너무 작은 경우 데이터 유실 가능성을 여전히 처리해야 한다. 이 함수의 반환 값은 `src`의 길이라서 중간에 잘렸는지 쉽게 알 수 있다. 반환 값이 `size`와 같거나 그보다 크다면 잘린 것이다. 데이터 유실이 문제가 되는 경우라면 *반드시* 호출 전에 인자들을 확인하거나 함수 반환 값을 검사해야 한다. `strlcpy()`는 glibc에 없으며 POSIX에도 표준화돼 있지 않다. 하지만 리눅스에서 `libbsd` 라이브러리를 통해 이용 가능하다.

## BUGS

`strcpy()`의 대상 문자열이 충분히 크지 않다면 어떤 일이든 발생할 수 있다. 고정 길이 문자열 버퍼를 넘치게 하는 건 크래커들이 머신을 장악하는 데 애용하는 기법이다. 프로그램에서 버퍼를 읽거나 버퍼로 데이터를 복사할 때는 먼저 공간히 충분한지 확인할 필요가 있다. 넘치는 게 불가능함을 보일 수 있다면 그럴 필요가 없을 수도 있겠지만, 그래도 조심해야 한다. 시간이 흐르면서 프로그램이 바뀔 수 있고, 그래서 불가능하던 게 가능해질 수 있다.

## SEE ALSO

<tt>[[bcopy(3)]]</tt>, <tt>[[memccpy(3)]]</tt>, <tt>[[memcpy(3)]]</tt>, <tt>[[memmove(3)]]</tt>, <tt>[[stpncpy(3)]]</tt>, <tt>[[strdup(3)]]</tt>, <tt>[[string(3)]]</tt>, `wcscpy(3)`, `wcsncpy(3)`

----

2021-03-22
