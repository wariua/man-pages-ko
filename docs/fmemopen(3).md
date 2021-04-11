## NAME

fmemopen - 메모리를 스트림으로 열기

## SYNOPSIS

```c
#include <stdio.h>

FILE *fmemopen(void *buf, size_t size, const char *mode);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`fmemopen()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

`fmemopen()` 함수는 `mode`로 지정한 방식으로 접근할 수 있는 스트림을 연다. 그 스트림을 통해 `buf`가 가리키는 문자열 내지 메모리 버퍼에 I/O를 수행할 수 있다.

`mode` 인자는 스트림 I/O의 동작 방식을 나타내며 다음 중 하나이다.

| | |
| --- | --- |
| `r`  | 스트림을 읽기용으로 연다. |
| `w`  | 스트림을 쓰기용으로 연다. |
| `a`  | 덧붙이기. 스트림을 쓰기용으로 열고 초기 버퍼 위치를 첫 널 바이트 위치로 설정한다. |
| `r+` | 스트림을 읽기 및 쓰기용으로 연다. |
| `w+` | 스트림을 읽기 및 쓰기용으로 연다. 버퍼 내용물을 잘라낸다. (즉 버퍼의 첫 바이트에 '\0'을 둔다.) |
| `a+` | 덧붙이기. 스트림을 읽기 및 쓰기용으로 열고 초기 버퍼 위치를 첫 널 바이트 위치로 설정한다. |

스트림에 현재 위치, 즉 다음 I/O 동작을 수행할 위치라는 개념이 있다. I/O 동작에서 현재 위치를 묵시적으로 갱신한다. `fseek(3)`으로 명시적으로 갱신할 수 있고 `ftell(3)`로 알아낼 수 있다. 덧붙이기를 제외한 모든 모드에서는 초기 위치를 버퍼 시작점으로 설정한다. 덧붙이기 모드에서 버퍼 내에 널 바이트가 없는 경우에는 초기 위치가 `size+1`이다.

`buf`를 NULL로 지정하면 `fmemopen()`에서 `size` 바이트의 버퍼를 할당한다. 임시 버퍼에 데이터를 쓰고서 다시 읽어 들이고 싶은 응용에서 유용하다. 초기 위치는 버퍼 시작점으로 설정한다. 그 버퍼는 스트림이 닫힐 때 자동으로 해제된다. 참고로 그런 호출로 할당된 임시 버퍼에 대한 포인터를 호출자가 얻을 수 있는 방법은 없다. (<tt>[[open_memstream(3)]]</tt>도 참고.)

`buf`가 NULL이 아니면 호출자가 할당한 최소 `len` 바이트짜리 버퍼를 가리켜야 한다.

쓰기 가능하게 연 스트림을 플러시(<tt>[[fflush(3)]]</tt>) 하거나 닫을(<tt>[[fclose(3)]]</tt>) 때 버퍼 끝에 공간이 있으면 널 바이트를 기록한다. 그렇게 되려면 버퍼에 여유 바이트가 있도록 (그리고 `size`가 그 바이트까지 포함하도록) 호출자가 신경써 줘야 한다.

읽기 가능하게 연 스트림에서 버퍼 내에 널 바이트('\0')가 있어도 읽기 동작이 파일 끝 표시를 반환하지는 않는다. 현재 버퍼 위치가 버퍼 시작점부터 `size` 바이트를 지날 때만 버퍼 읽기가 파일 끝을 알린다.

쓰기 동작은 현재 위치에서 일어나거나 (덧붙이기 외의 모드), 스트림의 현재 크기 위치에서 일어난다 (덧붙이기 모드).

버퍼에 `size` 바이트보다 많이 쓰려고 시도하면 오류가 발생한다. 기본적으로 그런 오류는 그 `stdio` 버퍼를 플러시 했을 때에야 (데이터는 사라진 채) 드러나게 된다. 다음 호출로 버퍼링을 꺼서 출력 동작 시점에 오류를 탐지하는 게 유용할 수 있다.

```c
setbuf(stream, NULL);
```

## RETURN VALUE

성공 완료 시 `fmemopen()`은 `FILE` 포인터를 반환한다. 아니면 NULL을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## VERSIONS

glibc 1.0.x부터 `fmemopen()`이 이미 사용 가능했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `fmemopen()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2008. 이 함수는 POSIX.1-2001에 명세되어 있지 않으며 다른 시스템들에서 널리 사용 가능하지 않다.

POSIX.1-2008에서는 `mode`의 'b'를 무시해야 한다고 명세한다. 하지만 기술 정오표 1에서 그 경우에 구현별 처리를 할 수 있도록 표준을 조정하고 있으며, 그래서 glibc의 'b' 처리 방식이 허용된다.

## NOTES

이 함수가 반환하는 파일 스트림에는 연계된 파일 디스크립터가 없다. (즉 반환된 스트림에 <tt>[[fileno(3)]]</tt>를 호출하면 오류를 반환하게 된다.)

버전 2.22에서 바이너리 모드(아래 참고)를 없애면서 `fmemopen()` 구현의 오래된 버그들이 고쳐졌으며 이 인터페이스를 위한 새 버전 심볼이 생겼다.

### 바이너리 모드

2.9에서 2.21까지 glibc의 `fmemopen()` 구현에서는 "바이너리" 모드를 지원했는데 `mode`에 두 번째 글자로 'b'를 지정해서 모드를 켰다. 이 모드에서는 쓰기 시 묵시적으로 종료용 널 바이트를 추가하지 않으며 `fseek(3)` `SEEK_END`가 현재 문자열 길이가 아니라 버퍼 끝(즉 `size` 인자로 지정한 값)을 기준으로 한다.

바이너리 모드 구현에는 API 버그가 있었는데, 바이너리 모드를 지정하려면 'b'가 `mode`에서 *두 번째* 문자여야 한다. 그래서 예를 들어 "wb+"라고 하면 원하는 효과가 나지만 "w+b"라고 하면 나지 않는다. <tt>[[fopen(3)]]</tt>의 `mode` 처리 방식과 일치하지 않는 방식이다.

glibc 2.22에서 바이너리 모드가 없어졌다. `mode`에 'b'를 지정해도 아무 효과가 없다.

## BUGS

glibc 버전 2.22 전에서, `size`를 0으로 지정하면 `fmemopen()`이 `EINVAL` 오류로 실패한다. 이 경우에 스트림이 성공적으로 생성된 다음 첫 번째 읽기 시도에서 파일 끝이 반환되는 게 더 일관적일 텐데, 버전 2.22부터 glibc 구현이 그런 동작 방식을 제공한다.

glibc 버전 2.22 전에서, `fmemopen()`에 덧붙이기 모드("a"나 "a+")를 지정하면 초기 버퍼 위치를 첫 널 바이트 위치로 설정하기는 하지만 (현재 위치를 스트림 끝이 아닌 위치로 재설정하는 경우) 이후의 쓰기가 스트림 끝에서 이뤄지도록 강제하지는 않는다. glibc 2.22에서 이 버그가 고쳐졌다.

glibc 버전 2.22 전에서, `fmemopen()`의 `mode` 인자가 덧붙이기("a"나 "a+")를 나타내고 `size` 인자가 나타내는 `buf` 내 범위에 널 바이트가 포함되지 않는 경우에 POSIX.1-2008에 따르면 초기 버퍼 위치를 버퍼 끝 다음 바이트 위치로 설정해야 한다. 하지만 이 경우에 glibc의 `fmemopen()`에서는 버퍼 위치를 -1로 설정한다. glibc 2.22에서 이 버그가 고쳐졌다.

glibc 버전 2.22 전에서, `fmemopen()`으로 만든 스트림에 `whence` 값을 `SEEK_END`로 해서 `fseek(3)` 호출을 수행할 때 `offset`을 스트림 끝 위치에 더하는 게 아니라 그 위치에서 *뺐다*. glibc 2.22에서 이 버그가 고쳐졌다.

glibc 2.9에서 `fmemopen()`에 "바이너리" 모드를 추가하면서 ABI가 조용히 바뀌었다. 그 전에는 `fmemopen()`에서 `mode` 내의 'b'를 무시했다.

## EXAMPLE

아래 프로그램에서는 `fmemopen()`을 사용해 입력 버퍼를 열고 <tt>[[open_memstream(3)]]</tt>을 사용해 동적 크기 출력 버퍼를 연다. (프로그램 첫 번째 명령행 인자에서 가져온) 입력 문자열을 탐색해서 정수들을 읽어 들이고, 그 정수의 제곱을 출력 버퍼로 써넣는다. 다음은 이 프로그램의 출력 예이다.

```text
$ ./a.out '1 23 43'
size=11; ptr=1 529 1849
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

#define handle_error(msg) \
    do { perror(msg); exit(EXIT_FAILURE); } while (0)

int
main(int argc, char *argv[])
{
    FILE *out, *in;
    int v, s;
    size_t size;
    char *ptr;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s '<num>...'\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    in = fmemopen(argv[1], strlen(argv[1]), "r");
    if (in == NULL)
        handle_error("open_memstream");

    out = open_memstream(&ptr, &size);
    if (out == NULL)
        handle_error("open_memstream");

    for (;;) {
        s = fscanf(in, "%d", &v);
        if (s <= 0)
            break;

        s = fprintf(out, "%d ", v * v);
        if (s == -1)
            handle_error("fprintf");
    }

    fclose(in);
    fclose(out);

    printf("size=%zu; ptr=%s\n", size, ptr);

    free(ptr);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[fopen(3)]]</tt>, <tt>[[fopencookie(3)]]</tt>, <tt>[[open_memstream(3)]]</tt>

----

2019-03-06
