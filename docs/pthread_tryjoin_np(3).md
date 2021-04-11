## NAME

pthread_tryjoin_np, pthread_timedjoin_np - 종료한 스레드와 합류 시도하기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <pthread.h>

int pthread_tryjoin_np(pthread_t thread, void **retval);

int pthread_timedjoin_np(pthread_t thread, void **retval,
                         const struct timespec *abstime);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

이 함수들은 이 페이지에서 기술하는 차이들을 제외하면 <tt>[[pthread_join(3)]]</tt>과 같은 방식으로 동작한다.

`pthread_tryjoin_np()` 함수는 스레드 `thread`와 논블로킹 합류를 수행하여 그 스레드의 종료 상태를 `*retval`에 반환한다. `thread`가 아직 종료하지 않았으면 <tt>[[pthread_join(3)]]</tt>처럼 블록 하지 않고 호출이 오류를 반환한다.

`pthread_timedjoin_np()` 함수는 타임아웃 있는 합류를 수행한다. `thread`가 아직 종료하지 않았으면 `abstime`에 지정한 최대 시간까지 호출이 블록 한다. `thread`가 종료하기 전에 타임아웃이 만료하면 호출이 오류를 반환한다. `abstime` 인자는 다음 형태의 구조체이며 에포크 기준으로 측정한 절대 시간을 나타낸다. (<tt>[[time(2)]]</tt> 참고.)

```c
struct timespec {
    time_t tv_sec;     /* 초 */
    long   tv_nsec;    /* 나노초 */
};
```

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 오류 번호를 반환한다.

## ERRORS

이 함수들은 <tt>[[pthread_join(3)]]</tt>과 같은 오류로 실패할 수 있다. 더불어 `pthread_tryjoin_np()`가 다음 오류로 실패할 수 있다.

`EBUSY`
:   호출 시점에 `thread`가 아직 종료하지 않았다.

더불어 `pthread_timedjoin_np()`가 다음 오류로 실패할 수 있다.

`ETIMEDOUT`
:   `thread`가 종료하기 전에 호출 시한이 넘어갔다.

`EINVAL`
:   `abstime` 값이 유효하지 않다. (`tv_sec`이 0보다 작거나 `tv_nsec`이 1e9보다 크다.)

`pthread_timedjoin_np()`는 절대 `EINTR` 오류를 반환하지 않는다.

## VERSIONS

glibc 버전 2.3.3에서 이 함수들이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_tryjoin_np()`,<br>`pthread_timedjoin_np()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "\_np"(nonportable: 이식성 없음)가 붙어 있다.

## EXAMPLE

다음 코드는 최대 5초 동안 합류를 기다린다.

```c
struct timespec ts;
int s;

...

if (clock_gettime(CLOCK_REALTIME, &ts) == -1) {
    /* 오류 처리 */
}

ts.tv_sec += 5;

s = pthread_timedjoin_np(thread, NULL, &ts);
if (s != 0) {
    /* 오류 처리 */
}
```

## SEE ALSO

<tt>[[clock_gettime(2)]]</tt>, <tt>[[pthread_exit(3)]]</tt>, <tt>[[pthread_join(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
