## NAME

etext, edata, end - 프로그램 세그먼트들의 끝

## SYNOPSIS

```c
extern etext;
extern edata;
extern end;
```

## DESCRIPTION

이 심볼들의 주소는 다양한 프로그램 세그먼트들의 끝을 나타낸다.

`etext`
:   텍스트 세그먼트(프로그램 코드)의 끝 다음의 첫 주소이다.

`edata`
:   초기화 된 데이터 세그먼트의 끝 다음의 첫 주소이다.

`end`
:   (BSS 세그먼트라고도 하는) 비초기화 데이터 세그먼트의 끝 다음의 첫 주소이다.

## CONFORMING TO

대부분의 유닉스 시스템에서 오랫동안 이 심볼들을 제공했지만 표준화되어 있지는 않다. 조심해서 사용하라.

## NOTES

프로그램에서 이 심볼들을 명시적으로 선언해야 한다. 어떤 헤더 파일에도 정의되어 있지 않다.

일부 시스템에서는 이 심볼들의 이름 앞에 밑줄이 붙어서 `_etext`, `_edata`, `_end`이다. 리눅스에서 컴파일 하는 프로그램에게는 이 심볼들도 정의되어 있다.

프로그램 실행이 시작될 때 프로그램 단절점(break)은 `&end` 근처 어딘가(아마 다음 페이지의 시작점)일 것이다. 하지만 <tt>[[brk(2)]]</tt>나 <tt>[[malloc(3)]]</tt>을 통해 메모리를 할당하면서 그 단절점이 바뀌게 된다. 인자 0으로 <tt>[[sbrk(2)]]</tt>를 사용하면 현재의 프로그램 단절점 값을 알아낼 수 있다.

## EXAMPLE

실행 시 아래 프로그램은 다음과 같은 출력을 내놓는다.

```
$ ./a.out
First address past:
    program text (etext)       0x8048568
    initialized data (edata)   0x804a01c
    uninitialized data (end)   0x804a024
```

### 프로그램 소스

```c
#include <stdio.h>
#include <stdlib.h>

extern char etext, edata, end; /* 심볼들에 뭔가 타입을 줘야 한다.
                                  그래야 "gcc -Wall"이 조용해진다. */

int
main(int argc, char *argv[])
{
    printf("First address past:\n");
    printf("    program text (etext)      %10p\n", &etext);
    printf("    initialized data (edata)  %10p\n", &edata);
    printf("    uninitialized data (end)  %10p\n", &end);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[objdump(1)]]</tt>, <tt>[[readelf(1)]]</tt>, <tt>[[sbrk(2)]]</tt>, <tt>[[elf(5)]]</tt>

----

2019-03-06
