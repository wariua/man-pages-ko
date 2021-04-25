## NAME

aio_init - 비동기 I/O 초기화

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <aio.h>

void aio_init(const struct aioinit *init);
```

`-lrt`로 링크.

## DESCRIPTION

GNU 전용인 `aio_init()` 함수를 이용해 호출자가 glibc POSIX AIO 구현에 튜닝 힌트를 줄 수 있다. 이 함수 사용은 선택적이며, 효력이 있으려면 POSIX AIO API의 다른 함수 사용 전에 호출해야 한다.

`init` 인자가 가리키는 버퍼로 튜닝 정보를 제공한다. 그 버퍼는 다음 형태의 구조체이다.

```c
struct aioinit {
    int aio_threads;    /* 스레드 최대 개수 */
    int aio_num;        /* 예상 동시 요청 개수 */
    int aio_locks;      /* 사용 안 함 */
    int aio_usedba;     /* 사용 안 함 */
    int aio_debug;      /* 사용 안 함 */
    int aio_numusers;   /* 사용 안 함 */
    int aio_idle_time;  /* 이 시간(초 단위)이 지나면 유휴
                           스레드 종료 (glibc 2.2부터) */
    int aio_reserved;
};
```

`aioinit` 구조체에서 다음 필드들이 쓰인다.

`aio_threads`
:   이 필드는 구현에서 사용할 수 있는 작업 스레드 최대 개수를 지정한다. 미처리 I/O 동작의 수가 이 제한을 초과하면 작업 스레드에 여유가 생길 때까지 초과 작업들을 큐에 넣어 두게 된다. 이 필드에 1보다 작은 값을 지정하면 1 값을 쓴다. 기본값은 20이다.

`aio_num`
:   이 필드는 호출자가 큐에 동시에 넣는 I/O 요청 최대 개수 예상치를 지정한다. 이 필드에 32보다 작은 값을 지정하면 32로 올린다. 기본값은 64이다.

`aio_idle_time`
:   이 필드는 작업 스레드가 이전 요청을 완료하고서 추가 요청을 몇 초나 기다린 후에 종료할지 지정한다. 기본값은 1이다.

## VERSIONS

glibc 2.1부터 `aio_init()` 함수가 사용 가능하다.

## CONFORMING TO

이 함수는 GNU 확장이다.

## SEE ALSO

<tt>[[aio(7)]]</tt>

----

2020-08-13
