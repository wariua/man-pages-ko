## NAME

asprintf, vasprintf - 할당된 문자열에 찍기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <stdio.h>

int asprintf(char **restrict strp, const char *restrict fmt, ...);
int vasprintf(char **restrict strp, const char *restrict fmt,
              va_list ap);
```

## DESCRIPTION

`asprintf()` 및 `vasprintf()` 함수는 <tt>[[sprintf(3)]]</tt> 및 <tt>[[vsprintf(3)]]</tt>와 유사하되 종료용 널 바이트(`'\0'`)까지 포함해 출력을 담을 만큼 큰 문자열을 할당하며 첫 번째 인자를 통해 포인터를 반환한다. 그 포인터가 더는 필요치 않을 때 <tt>[[free(3)]]</tt>에 줘서 할당된 저장 공간을 해제해야 한다.

## RETURN VALUE

성공 시 이 함수들은 <tt>[[sprintf(3)]]</tt>와 마찬가지로 찍은 바이트 수를 반환한다. 메모리 할당이 불가능했거나 어떤 다른 오류가 발생한 경우 -1을 반환하며 그때 `strp`의 내용은 규정돼 있지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `asprintf()`, `vasprintf()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

이 함수들은 GNU 확장이며 C나 POSIX에 포함돼 있지 않다. BSD 계열에서도 이용 가능하다. FreeBSD 구현에선 오류 시 `strp`를 NULL로 설정한다.

## SEE ALSO

<tt>[[free(3)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[printf(3)]]</tt>

----

2021-03-22
