## NAME

open_memstream, open_wmemstream - 동적 메모리 버퍼 스트림 열기

## SYNOPSIS

```c
#include <stdio.h>

FILE *open_memstream(char **ptr, size_t *sizeloc);

#include <wchar.h>

FILE *open_wmemstream(wchar_t **ptr, size_t *sizeloc);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`open_memstream()`, `open_wmemstream()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

`open_memstream()` 함수는 메모리 버퍼에 쓰기를 하기 위한 스트림을 연다. 함수에서 버퍼를 동적으로 할당하며 필요에 따라 버퍼가 자동으로 커진다. 처음에는 버퍼 크기가 0이어야 한다. 스트림을 닫은 후에 호출자가 그 버퍼를 <tt>[[free(3)]]</tt> 해야 한다.

`ptr`과 `sizeloc`이 가리키는 위치를 통해 버퍼의 현재 위치와 크기를 알려 준다. 이 포인터들이 가리키는 위치는 스트림이 플러시 될 때(<tt>[[fflush(3)]]</tt>)마다, 그리고 스트림이 닫힐 때(<tt>[[fclose(3)]]</tt>) 갱신된다. 호출자가 스트림에 추가로 출력을 수행하지 않는 동안만 그 값들이 유효하다. 추가로 출력을 수행한 경우에 그 값들에 접근하려면 먼저 스트림을 다시 플러시 해야 한다.

버퍼 끝에 널 바이트를 유지한다. 이 바이트는 `sizeloc`에 저장되는 크기 값에 포함되지 *않는다*.

스트림에 현재 위치 개념이 있으며 처음에는 0(버퍼 시작점)이다. 각 쓰기 동작에서 그 버퍼 위치를 묵시적으로 조정한다. `fseek(3)` 내지 `fseeko(3)`로 스트림의 버퍼 위치를 명시적으로 바꿀 수 있다. 지금까지 기록한 데이터의 끝 너머로 버퍼 위치를 옮기면 그 사이 공간을 널 문자로 채운다.

`open_wmemstream()`은 `open_memstream()`과 비슷하되 바이트 대신 확장 문자에서 동작한다.

## RETURN VALUE

성공 완료 시 `open_memstream()`과 `open_wmemstream()`은 `FILE` 포인터를 반환한다. 아니면 NULL을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## VERSIONS

`open_memstream()`은 glibc 1.0.x부터 이미 사용 가능했다. `open_wmemstream()`은 glibc 2.4부터 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `open_memstream()`,<br>`open_wmemstream()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2008. 이 함수들은 POSIX.1-2001에 명세되어 있지 않으며 다른 시스템들에서 널리 사용 가능하지 않다.

## NOTES

이 함수들이 반환하는 파일 스트림에는 연계된 파일 디스크립터가 없다. (즉 반환된 스트림에 <tt>[[fileno(3)]]</tt>를 호출하면 오류를 반환하게 된다.)

## BUGS

glibc 버전 2.7 전에서는 `open_memstream()`으로 만든 스트림 끝 너머로 이동을 하면 버퍼가 확장되지 않는다. 대신 `fseek(3)` 호출이 실패하고 -1을 반환한다.

## EXAMPLES

<tt>[[fmemopen(3)]]</tt> 참고.

## SEE ALSO

<tt>[[fmemopen(3)]]</tt>, <tt>[[fopen(3)]]</tt>, <tt>[[setbuf(3)]]</tt>

----

2021-03-22
