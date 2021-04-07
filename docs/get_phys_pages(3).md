## NAME

get_phys_pages, get_avphys_pages - 물리적 페이지 총개수와 가용 개수 얻기

## SYNOPSIS

```c
#include <sys/sysinfo.h>

long int get_phys_pages(void);
long int get_avphys_pages(void);
```

## DESCRIPTION

`get_phys_pages()` 함수는 시스템에서 사용 가능한 물리적 메모리 페이지의 총개수를 반환한다.

`get_avphys_pages()` 함수는 시스템에서 현재 사용 가능한 물리적 메모리 페이지 개수를 반환한다.

## RETURN VALUE

성공 시 이 함수들은 DESCRIPTION의 설명처럼 음수 아닌 값을 반환한다. 실패 시 -1을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>ENOSYS</code></dt>
<dd>필요한 정보를 시스템이 제공하지 못했다. (<code>/proc</code> 파일 시스템이 마운트 되지 않아서일 수 있다.)</dd>
</dl>

## CONFORMING TO

이 함수들은 GNU 확장이다.

## NOTES

이 함수들은 `/proc/meminfo`의 `MemTotal` 및 `MemFree` 필드를 읽어서 필요한 정보를 얻는다.

다음 <tt>[[sysconf(3)]]</tt> 호출은 이 페이지에서 기술하는 함수들과 같은 정보를 얻을 수 있는 이식성 있는 방법이다.

```c
total_pages = sysconf(_SC_PHYS_PAGES);    /* 총 페이지 */
avl_pages = sysconf(_SC_AVPHYS_PAGES);    /* 가용 페이지 */
```

## EXAMPLE

`get_phys_pages()`와 `get_avphys_pages()`를 어떻게 사용할 수 있는지 다음 예가 보여 준다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/sysinfo.h>

int
main(int argc, char *argv[])
{
    printf("This system has %ld pages of physical memory and "
            "%ld pages of physical memory available.\n",
            get_phys_pages(), get_avphys_pages());
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[sysconf(3)]]</tt>

----

2019-03-06
