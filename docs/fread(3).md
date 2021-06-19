## NAME

fread, fwrite - 이진 스트림 입출력

## SYNOPSIS

```c
#include <stdio.h>

size_t fread(void *restrict ptr, size_t size, size_t nmemb,
             FILE *restrict stream);
size_t fwrite(const void *restrict ptr, size_t size, size_t nmemb,
             FILE *restrict stream);
```

## DESCRIPTION

`fread()` 함수는 `stream`이 가리키는 스트림에서 각기 `size` 바이트 길이인 데이터 항목 `nmemb` 개를 읽어서 `ptr`로 지정한 위치에 저장한다.

`fwrite()` 함수는 `ptr`로 지정한 위치에서 얻은 각기 `size` 바이트 길이인 데이터 항목 `nmemb` 개를 `stream`이 가리키는 스트림에 써 넣는다.

대응하는 논블로킹 버전은 <tt>[[unlocked_stdio(3)]]</tt> 참고.

## RETURN VALUE

성공 시 `fread()`와 `fwrite()`는 읽거나 쓴 항목 개수를 반환한다. `size`가 1인 경우에만 그 수가 이동된 바이트 수와 같다. 오류가 발생했거나 파일 끝에 도달했으면 지정한 항목 개수에 모자란 값을 (또는 0을) 반환한다.

성공적으로 읽거나 쓴 바이트 수만큼 스트림의 파일 위치 표시가 전진한다.

`fread()`에서 파일 끝과 오류를 구별하지 않으므로 호출자가 <tt>[[feof(3)]]</tt>와 <tt>[[ferror(3)]]</tt>를 써서 어느 경우인지 알아내야 한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `fread()`, `fwrite()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89.

## EXAMPLES

아래 프로그램은 `fread()` 사용 방식을 보여 주는데, ELF 실행 파일 /bin/sh을 바이너리 모드로 파싱해서 매직 값과 유형을 찍는다.

```text
$ ./a.out
ELF magic: 0x7f454c46
Class: 0x02
```

### 프로그램 소스

```c
#include <stdio.h>
#include <stdlib.h>

#define ARRAY_SIZE(arr) (sizeof(arr) / sizeof((arr)[0]))

int
main(void)
{
    FILE *fp = fopen("/bin/sh", "rb");
    if (!fp) {
        perror("fopen");
        return EXIT_FAILURE;
    }

    unsigned char buffer[4];

    size_t ret = fread(buffer, sizeof(*buffer), ARRAY_SIZE(buffer), fp);
    if (ret != ARRAY_SIZE(buffer)) {
        fprintf(stderr, "fread() failed: %zu\n", ret);
        exit(EXIT_FAILURE);
    }

    printf("ELF magic: %#04x%02x%02x%02x\n", buffer[0], buffer[1],
           buffer[2], buffer[3]);

    ret = fread(buffer, 1, 1, fp);
    if (ret != 1) {
        fprintf(stderr, "fread() failed: %zu\n", ret);
        exit(EXIT_FAILURE);
    }

    printf("Class: %#04x\n", buffer[0]);

    fclose(fp);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[read(2)]]</tt>, <tt>[[write(2)]]</tt>, <tt>[[feof(3)]]</tt>, <tt>[[ferror(3)]]</tt>, <tt>[[unlocked_stdio(3)]]</tt>

----

2021-03-22
