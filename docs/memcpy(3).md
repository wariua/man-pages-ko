## NAME

memcpy - 메모리 영역 복사하기

## SYNOPSIS

```c
#include <string.h>

void *memcpy(void *restrict dest, const void *restrict src, size_t n);
```

## DESCRIPTION

`memcpy()` 함수는 메모리 영역 `src`에서 메모리 영역 `dest`로 `n`개 바이트를 복사한다. 두 메모리 영역이 겹쳐선 안 된다. 메모리 영역이 겹친다면 <tt>[[memmove(3)]]</tt>를 쓰면 된다.

## RETURN VALUE

`memcpy()` 함수는 `dest` 포인터를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `memcpy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

메모리 영역이 겹쳐선 안 된다는 요건을 따르지 못한 것이 여러 중요한 버그들의 원인이 되었다. (POSIX 및 C 표준에선 겹치는 영역에 `memcpy()` 사용 시 규정 안 된 동작이 발생한다고 분명히 밝히고 있다.) 특히 주목받았던 건 glibc 2.13에서 (x86-64를 포함한) 일부 플랫폼의 `memcpy()` 성능 최적화를 하면서 `src`에서 `dest`로 바이트들을 복사하는 순서가 바뀐 일이다.

그 변화 때문에 겹치는 영역으로 복사를 수행하던 여러 응용들의 결함이 드러났다. 이전 구현 하에선 바이트들을 복사하던 순서 때문에 우연히 버그가 감춰지다가 복사 순서가 바뀌면서 드러난 것이다. glibc 2.14에서 버전 붙은 심볼이 추가돼서 구식인 (즉 2.14 전 glibc 버전들에 링크된) 바이너리들이 버퍼가 겹친 경우를 안전하게 다루는 `memcpy()` 구현을 이용하게 했다. (<tt>[[memmove(3)]]</tt>의 별칭 형태로 "구식" `memcpy()` 구현을 제공했다.)

## SEE ALSO

<tt>[[bcopy(3)]]</tt>, <tt>[[bstring(3)]]</tt>, <tt>[[memccpy(3)]]</tt>, <tt>[[memmove(3)]]</tt>, <tt>[[mempcpy(3)]]</tt>, <tt>[[strcpy(3)]]</tt>, <tt>[[strncpy(3)]]</tt>, `wmemcpy(3)`

----

2021-03-22
