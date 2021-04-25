## NAME

mallopt - 메모리 할당 매개변수 설정하기

## SYNOPSIS

```c
#include <malloc.h>

int mallopt(int param, int value);
```

## DESCRIPTION

`mallopt()` 함수는 메모리 할당 함수들(<tt>[[malloc(3)]]</tt> 참고)의 동작을 제어하는 매개변수들을 조정한다. `param` 인자는 변경할 매개변수를 나타내고 `value`는 그 매개변수의 새 값을 나타낸다.

`param`에 다음 값을 지정할 수 있다.

`M_ARENA_MAX`
:   값이 0이 아니면 이 매개변수는 만들 수 있는 아레나 최대 개수에 대한 경성 제한을 규정한다. 아레나(arena)는 <tt>[[malloc(3)]]</tt> (및 유사) 호출이 할당 요청을 처리하는 데 사용할 수 있는 메모리 풀을 나타낸다. 아레나는 스레드에 대해 안전하고 그래서 동시에 메모리 요청이 여러 개 있을 수 있다. 스레드 수와 아레나 수 간에는 상충 관계가 있다. 아레나가 많을수록 스레드 간 경쟁이 줄어들지만 대신 메모리 사용량이 올라간다.

    이 매개변수의 기본값은 0인데, `M_ARENA_TEST` 설정에 따라 아레나 개수 제한을 정한다는 뜻이다.

    이 매개변수는 glibc 2.10부터 `--enable-experimental-malloc`을 통해서, 그리고 glibc 2.15부터는 기본적으로 사용 가능하다. 어떤 할당자 버전들(가령 CentOS 5, RHEL 5)에서는 생성 아레나 수에 제한이 없었다.

    이후의 glibc 버전을 사용하고 있을 때 어떤 경우에 응용에서 아레나 접근 시 높은 경쟁을 보일 수 있다. 그런 경우에 스레드 수에 맞게 `M_ARENA_MAX`를 높이는 것이 유익할 수 있다. 이는 동작 방식 면에서 tcmalloc과 jemalloc이 취하는 전략(스레드별 할당 풀)과 비슷하다.

`M_ARENA_TEST`
:   이 매개변수는 시스템 설정을 확인해서 생성 아레나 수에 대한 경성 제한을 알아낼 시점을 생성 아레나 수 단위로 지정한다. (아레나의 정의는 `M_ARENA_MAX`를 보라.)

    아레나 경성 제한 계산 방식은 구현별로 규정하는데 일반적으로 가용 CPU 수의 배수로 계산한다. 경성 제한을 한번 계산하고 나면 그 결과가 최종 값이 되어 아레나의 총개수를 제한한다.

    `M_ARENA_TEST` 매개변수의 기본값은 `sizeof(long)`이 4인 시스템에서는 2이다. 그 외 경우에서는 기본값이 8이다.

    이 매개변수는 glibc 2.10부터 `--enable-experimental-malloc`을 통해서, 그리고 glibc 2.15부터는 기본적으로 사용 가능하다.

    `M_ARENA_MAX`가 0 아닌 값을 가질 때는 `M_ARENA_TEST`의 값을 사용하지 않는다.

`M_CHECK_ACTION`
:   이 매개변수 설정은 다양한 프로그래밍 오류들(가령 같은 포인터 두 번 해제하기)을 탐지했을 때 glibc가 어떻게 대응하는지를 제어한다. 이 매개변수에 할당한 값의 하위 3개 (0번, 1번, 2번) 비트가 다음과 같이 glibc 동작을 결정한다.

    0번 비트
    :   이 비트가 설정되어 있으면 오류 상세 내용을 담은 한 줄짜리 메시지를 `stderr`로 찍는다. 메시지는 "\*\*\* glibc detected ***"로 시작하며, 다음으로 프로그램 이름과 오류를 탐지한 메모리 할당 함수 이름, 짧은 오류 설명, 그리고 오류를 탐지한 메모리 주소가 온다.

    1번 비트
    :   이 비트가 설정되어 있으면 0번 비트에서 상술한 오류 메시지를 찍은 후에 프로그램이 <tt>[[abort(3)]]</tt>를 호출해서 종료한다. 그리고 glibc 버전 2.4부터는 0번 비트가 함께 설정돼 있으면 오류 메시지 출력과 실행 중단 사이에서 프로그램이 <tt>[[backtrace(3)]]</tt> 방식으로 스택 트레이스를 찍고 `/proc/[pid]/maps` 방식(<tt>[[proc(5)]]</tt> 참고)으로 프로세스의 메모리 매핑을 찍는다.

    2번 비트 (glibc 2.4부터)
    :   0번 비트가 함께 설정돼 있을 때만 이 비트가 효과가 있다. 이 비트가 설정되어 있으면 오류를 설명하는 한 줄 메시지가 간단해져서 오류를 탐지한 함수 이름과 짧은 오류 설명만 담는다.

    `value`의 나머지 비트들은 무시한다.

    위 내용을 종합하면 `M_CHECK_ACTION`에서 다음 숫자 값들이 의미가 있다.

    | | |
    | --- | --- |
    | 0 | 오류 조건을 무시한다. (규정되지 않은 결과로) 실행을 계속한다. |
    | 1 | 자세한 오류 메시지를 찍고 실행을 계속한다. |
    | 2 | 프로그램 실행을 중단한다. |
    | 3 | 자세한 오류 메시지와 스택 트레이스, 메모리 매핑을 찍고 프로그램 실행을 중단한다. |
    | 5 | 간단한 오류 메시지를 찍고 실행을 계속한다. |
    | 7 | 간단한 오류 메시지와 스택 트레이스, 메모리 매핑을 찍고 프로그램 실행을 중단한다. |

    glibc 2.3.4부터는 `M_CHECK_ACTION` 매개변수의 기본값이 3이다. glibc 버전 2.3.3 및 이전에서는 기본값이 1이다.

    `M_CHECK_ACTION`에 0 아닌 값을 쓰는 게 유용할 수 있는 건 안 그러면 크래시가 훨씬 나중에 발생해서 문제의 진짜 원인을 잡아내기가 아주 어려울 수도 있기 때문이다.

`M_MMAP_MAX`
:   이 매개변수는 <tt>[[mmap(2)]]</tt>으로 동시에 대응할 수 있는 할당 요청 최대 개수를 나타낸다. 이 매개변수가 존재하는 이유는 일부 시스템에서 <tt>[[mmap(2)]]</tt>에 쓰는 내부 테이블 수가 한정돼 있어서 여러 개를 사용할 때 성능이 떨어질 수 있기 때문이다.

    기본값은 65,536이다. 특별한 의미가 있는 값은 아니고 안전 장치 역할일 뿐이다. 이 매개변수를 0으로 설정하면 큰 할당 요청을 <tt>[[mmap(2)]]</tt>으로 처리하는 동작이 꺼진다.

`M_MMAP_THRESHOLD`
:   `M_MMAP_THRESHOLD`가 나타내는 (바이트 단위) 제한 이상이고 유휴 목록으로 처리할 수 없는 요청에 대해 메모리 할당 함수들은 <tt>[[sbrk(2)]]</tt>로 프로그램 단절점을 올리는 대신 <tt>[[mmap(2)]]</tt>을 사용한다.

    <tt>[[mmap(2)]]</tt>을 이용한 메모리 할당 방식에는 할당 메모리 블록을 언제든 독립적으로 시스템으로 해제할 수 있다는 큰 장점이 있다. (그에 반해 힙은 상단에서 메모리를 해제할 때만 줄일 수 있다.) 한편으로 <tt>[[mmap(2)]]</tt> 사용에는 단점도 있는데, 할당 해제한 공간을 향후 할당에서 재사용하도록 유휴 목록에 두지 않으며, <tt>[[mmap(2)]]</tt> 할당이 페이지에 정렬돼 있어야 하므로 메모리가 낭비될 수 있으며, <tt>[[mmap(2)]]</tt>을 통해 할당하는 메모리를 0으로 채우는 비싼 작업을 커널이 수행해야 한다. 이런 인자들 사이에서 균형을 잡은 결과가 `M_MMAP_THRESHOLD` 매개변수 기본 설정인 128\*1024이다.

    이 매개변수의 하한은 0이다. 상한은 `DEFAULT_MMAP_THRESHOLD_MAX`인데, 32비트 시스템에서는 512\*1024이고 64비트 시스템에서는 `4*1024*1024*sizeof(long)`이다.

    *참고*: 요즘 glibc에서는 기본적으로 동적인 mmap 문턱값을 사용한다. 문턱값이 처음에는 128\*1024이지만 현재 문턱보다 크고 `DEFAULT_MMAP_THRESHOLD_MAX` 이하인 블록을 해제할 때 문턱값을 해제 블록 크기에 가깝게 조정한다. 동적인 mmap 문턱값을 사용하고 있을 때는 힙 줄이기 문턱값 역시도 동적 mmap 문턱값의 두 배가 되도록 동적으로 조정한다. `M_TRIM_THRESHOLD`, `M_TOP_PAD`, `M_MMAP_THRESHOLD`, `M_MMAP_MAX` 매개변수 중 하나라도 설정하면 mmap 문턱값 자동 조정이 꺼진다.

`M_MXFAST` (glibc 2.3부터)
:   "패스트빈(fastbin)"을 이용해 처리할 메모리 할당 요청의 상한을 설정한다. (이 매개변수의 단위는 바이트이다.) 패스트빈은 해제된 같은 크기 메모리 블록들을 인접 블록과 병합하지 않고 유지해 두는 저장 영역이다. 이후에 같은 크기 블록을 다시 할당하면 패스트빈을 이용해 아주 빨리 처리할 수 있다. 다만 메모리 단편화와 프로그램의 전반적 메모리 사용량이 증가할 수 있다.

    이 매개변수의 기본값은 `64*sizeof(size_t)/4`(32비트 아키텍처에서는 64)이다. 이 매개변수의 범위는 0에서 `80*sizeof(size_t)/4`까지이다. `M_MXFAST`를 0으로 설정하면 패스트빈 사용을 끈다.

`M_PERTURB` (glibc 2.4부터)
:   이 매개변수를 0 아닌 값으로 설정하면 (<tt>[[calloc(3)]]</tt>을 통한 할당은 제외하고) 할당된 메모리의 바이트들을 `value`의 최하위 바이트 값의 보수로 초기화 하며, 할당 메모리를 <tt>free(3)</tt>를 이용해 해제할 때 해제된 바이트들을 `value`의 최하위 바이트로 설정한다. 프로그램에서 할당 메모리가 0으로 초기화 되어 있다고 잘못 상정하거나 이미 해제된 메모리의 값을 재사용하는 오류를 탐지하는 데 도움이 될 수 있다.

    이 매개변수의 기본값은 0이다.

`M_TOP_PAD`
:   이 매개변수는 <tt>[[sbrk(2)]]</tt>를 호출해 프로그램 단절점을 변경할 때 적용할 패딩의 양을 규정한다. (이 매개변수의 단위는 바이트이다.) 다음 경우에 이 매개변수가 효과가 있다.

    * 프로그램 단절점이 올라갈 때 <tt>[[sbrk(2)]]</tt> 요청에 `M_TOP_PAD` 바이트를 추가한다.

    * <tt>[[free(3)]]</tt> 호출의 결과로 힙을 줄일 때 (`M_TRIM_THRESHOLD` 설명 참고) 힙 상단에 이만큼의 유휴 공간을 남겨둔다.

    어느 경우이든 패딩 양을 항상 시스템 페이지 경계로 올림 한다.

    `M_TOP_PAD` 변경은 (매개변수를 낮게 설정했을 때) 시스템 호출 횟수가 늘어나는 것과 (매개변수를 높게 설정했을 때) 힙 상단의 안 쓰는 메모리를 낭비하는 것 사이의 타협이다.

    이 매개변수의 기본값은 128\*1024이다.

`M_TRIM_THRESHOLD`
:   힙 상단의 연속된 유휴 메모리가 충분히 커지면 <tt>[[free(3)]]</tt>에서 <tt>[[sbrk(2)]]</tt>를 사용해 그 메모리를 시스템으로 되돌려준다. (상당한 양의 메모리를 해제한 후에도 오랫동안 실행되는 프로그램에서 이 동작이 유용할 수 있다.) `M_TRIM_THRESHOLD` 매개변수는 <tt>[[sbrk(2)]]</tt>를 이용해 힙을 줄이게 되려면 유휴 메모리 블록이 도달해야 하는 최소 크기(바이트 단위)를 지정한다.

    이 매개변수의 기본값은 128\*1024이다. `M_TRIM_THRESHOLD`를 -1로 설정하면 줄이기를 완전히 끈다.

    `M_TRIM_THRESHOLD` 변경은 (매개변수를 낮게 설정했을 때) 시스템 호출 횟수가 늘어나는 것과 (매개변수를 높게 설정했을 때) 힙 상단의 안 쓰는 메모리를 낭비하는 것 사이의 타협이다.

### 환경 변수

여러 환경 변수들을 정의하여 `mallopt()`로 제어하는 것과 같은 매개변수들을 변경할 수 있다. 환경 변수 사용은 프로그램 소스 코드를 바꿀 필요가 없다는 장점이 있다. 효과가 있으려면 첫 번째 메모리 할당 함수 호출 전에 이 변수들이 정의돼 있어야 한다. (같은 매개변수를 `mallopt()`를 통해 조정하는 경우에는 `mallopt()` 설정이 우선시된다.) 보안적 이유 때문에 set-user-ID 및 set-group-ID 프로그램에서는 이 변수들이 무시된다.

그 환경 변수들은 다음과 같다. (일부 환경 변수 이름 끝에 밑줄이 붙어 있다는 것에 유의하라.)

`MALLOC_ARENA_MAX`
:   `mallopt()` `M_ARENA_MAX`와 같은 매개변수를 제어한다.

`MALLOC_ARENA_TEST`
:   `mallopt()` `M_ARENA_TEST`와 같은 매개변수를 제어한다.

`MALLOC_CHECK_`
:   이 환경 변수는 `mallopt()` `M_CHECK_ACTION`과 같은 매개변수를 제어한다. 이 변수가 0 아닌 값으로 설정돼 있으면 특수하게 구현된 메모리 할당 함수들을 쓴다. (<tt>[[malloc_hook(3)]]</tt> 기능을 이용해 그렇게 한다.) 그 구현에서 추가적인 오류 검사를 수행하며 표준 메모리 할당 함수들보다 느리다. (그 구현이 가능한 모든 오류를 탐지하지는 않는다. 메모리 누수는 여전히 발생할 수 있다.)

    이 환경 변수에 할당하는 값은 한 자리 숫자여야 하며, 그 의미는 `M_CHECK_ACTION`에서 설명하는 대로이다. 그 첫 번째 숫자 이후의 문자들은 무시된다.

    보안적 이유 때문에 set-user-ID 및 set-group-ID 프로그램에는 기본적으로 `MALLOC_CHECK_`의 효과가 꺼진다. 하지만 `/etc/suid-debug`라는 파일이 존재하면 (파일의 내용물은 상관없음) set-user-ID 및 set-group-ID 프로그램에도 `MALLOC_CHECK_`가 효과가 있다.

`MALLOC_MMAP_MAX_`
:   `mallopt()` `M_MMAP_MAX`와 같은 매개변수를 제어한다.

`MALLOC_MMAP_THRESHOLD_`
:   `mallopt()` `M_MMAP_THRESHOLD`와 같은 매개변수를 제어한다.

`MALLOC_PERTURB_`
:   `mallopt()` `M_PERTURB`와 같은 매개변수를 제어한다.

`MALLOC_TRIM_THRESHOLD_`
:   `mallopt()` `M_TRIM_THRESHOLD`와 같은 매개변수를 제어한다.

`MALLOC_TOP_PAD_`
:   `mallopt()` `M_TOP_PAD`와 같은 매개변수를 제어한다.

## RETURN VALUE

성공 시 `mallopt()`는 1을 반환한다. 오류 시 0을 반환한다.

## ERRORS

오류 시 `errno`를 설정하지 *않는다*.

## CONFORMING TO

이 함수는 POSIX나 C 표준에 명세되어 있지 않다. 여러 시스템 V 파생 시스템에 비슷한 함수가 존재하지만 `param` 값의 범위가 시스템에 따라 다르다. SVID에서 `M_MXFAST`, `M_NLBLKS`, `M_GRAIN`, `M_KEEP` 옵션을 정의했지만 이 중 첫 번째만 glibc에 구현되어 있다.

## BUGS

`param`에 유효하지 않은 값을 지정해도 오류가 발생하지 않는다.

glibc 구현 내의 계산 오류 때문에 다음과 같이 호출 시

```c
mallopt(M_MXFAST, n)
```

`n`까지 크기의 모든 할당에 패스트빈을 사용하게 되지 않는다. 원하는 결과를 보장하려면 `n`을 다음 번 `(2k+1)*sizeof(size_t)`로 올림 해야 한다 (`k`는 정수).

`mallopt()`를 사용해 `M_PERTURB`를 설정하면 기대하는 대로 할당 메모리의 바이트들이 `value`의 바이트의 보수로 초기화 되고 메모리를 해제할 때 그 영역의 바이트들이 `value`에 지정한 바이트로 초기화 된다. 그런데 구현에 `sizeof(size_t)` 차이의 오류가 있다. `free(p)` 호출로 해제하는 메모리 블록을 정확히 초기화 하지 않고 `p+sizeof(size_t)`에서 시작하는 블록을 초기화 한다.

## EXAMPLES

아래 프로그램은 `M_CHECK_ACTION` 사용 방식을 보여 준다. 프로그램에 (정수) 명령행 인자를 주면 그 인자로 `M_CHECK_ACTION` 매개변수를 설정한다. 그러고서 프로그램은 메모리 블록을 할당해서 이를 두 번 해제한다 (오류).

다음 셸 세션은 glibc 하에서 `M_CHECK_ACTION` 기본값으로 이 프로그램을 실행할 때 어떻게 되는지 보여 준다.

```text
$ ./a.out
main(): returned from first free() call
*** glibc detected *** ./a.out: double free or corruption (top): 0x09d30008 ***
======= Backtrace: =========
/lib/libc.so.6(+0x6c501)[0x523501]
/lib/libc.so.6(+0x6dd70)[0x524d70]
/lib/libc.so.6(cfree+0x6d)[0x527e5d]
./a.out[0x80485db]
/lib/libc.so.6(__libc_start_main+0xe7)[0x4cdce7]
./a.out[0x8048471]
======= Memory map: ========
001e4000-001fe000 r-xp 00000000 08:06 1083555    /lib/libgcc_s.so.1
001fe000-001ff000 r--p 00019000 08:06 1083555    /lib/libgcc_s.so.1
[일부 행 생략]
b7814000-b7817000 rw-p 00000000 00:00 0
bff53000-bff74000 rw-p 00000000 00:00 0          [stack]
Aborted (core dumped)
```

다음 실행 예는 `M_CHECK_ACTION`에 다른 값을 사용할 때의 결과를 보여 준다.

```text
$ ./a.out 1             # 오류 진단 및 실행 계속
main(): returned from first free() call
*** glibc detected *** ./a.out: double free or corruption (top): 0x09cbe008 ***
main(): returned from second free() call
$ ./a.out 2             # 오류 메시지 없이 실행 중단
main(): returned from first free() call
Aborted (core dumped)
$ ./a.out 0             # 오류 무시하고 실행 계속
main(): returned from first free() call
main(): returned from second free() call
```

다음 실행 예는 같은 매개변수를 `MALLOC_CHECK_` 환경 변수를 이용해 설정하는 방법을 보여 준다.

```text
$ MALLOC_CHECK_=1 ./a.out
main(): returned from first free() call
*** glibc detected *** ./a.out: free(): invalid pointer: 0x092c2008 ***
main(): returned from second free() call
```

### 프로그램 소스

```c
#include <malloc.h>
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
    char *p;

    if (argc > 1) {
        if (mallopt(M_CHECK_ACTION, atoi(argv[1])) != 1) {
            fprintf(stderr, "mallopt() failed");
            exit(EXIT_FAILURE);
        }
    }

    p = malloc(1000);
    if (p == NULL) {
        fprintf(stderr, "malloc() failed");
        exit(EXIT_FAILURE);
    }

    free(p);
    printf("main(): returned from first free() call\n");

    free(p);
    printf("main(): returned from second free() call\n");

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[mmap(2)]]</tt>, <tt>[[sbrk(2)]]</tt>, <tt>[[mallinfo(3)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[malloc_hook(3)]]</tt>, <tt>[[malloc_info(3)]]</tt>, <tt>[[malloc_stats(3)]]</tt>, <tt>[[malloc_trim(3)]]</tt>, <tt>[[mcheck(3)]]</tt>, <tt>[[mtrace(3)]]</tt>, <tt>[[posix_memalign(3)]]</tt>

----

2021-03-22
