## NAME

pthread_cleanup_push_defer_np, pthread_cleanup_pop_restore_np - 스레드 취소 정리 핸들러 집어넣고 꺼내면서 취소 가능성 유형도 저장하기

## SYNOPSIS

```c
#include <pthread.h>

void pthread_cleanup_push_defer_np(void (*routine)(void *),
                                   void *arg);
void pthread_cleanup_pop_restore_np(int execute);
```

`-pthread`로 링크.

## DESCRIPTION

이 함수들은 이 페이지에서 언급하는 차이들을 제외하면 <tt>[[pthread_cleanup_push(3)]]</tt> 및 <tt>[[pthread_cleanup_pop(3)]]</tt>과 같다.

`pthread_cleanup_push_defer_np()`는 <tt>[[pthread_cleanup_push(3)]]</tt>처럼 스레드의 취소 정리 핸들러 스택에 `routine`을 집어넣는다. 그에 더해서 스레드의 현재 취소 가능성 유형을 저장하고 취소 가능성 유형을 "연기"로 설정한다. (<tt>[[pthread_setcanceltype(3)]]</tt> 참고.) 이렇게 하면 호출 전에 스레드의 취소 가능성 유형이 "비동기"였던 경우에도 취소 정리가 일어나게 된다.

`pthread_cleanup_pop_restore_np()`는 <tt>[[pthread_cleanup_pop(3)]]</tt>처럼 스레드의 취소 정리 핸들러 스택에서 최상단의 정리 핸들러를 꺼낸다. 그에 더해서 스레드의 취소 가능성 유형을 대응하는 `pthread_cleanup_push_defer_np()` 호출 시점의 값으로 복원한다.

이 함수들의 호출이 같은 함수 안에 있도록, 그리고 같은 문법적 내포 단계에 있도록 해야 한다. <tt>[[pthread_cleanup_push(3)]]</tt>에서 기술하는 다른 제약들이 마찬가지로 적용된다.

다음 호출 열이

```c
pthread_cleanup_push_defer_np(routine, arg);
pthread_cleanup_pop_restore_np(execute);
```

다음과 동등하다. (하지만 더 짧고 효율적이다.)

```c
int oldtype;

pthread_cleanup_push(routine, arg);
pthread_setcanceltype(PTHREAD_CANCEL_DEFERRED, &oldtype);
...
pthread_setcanceltype(oldtype, NULL);
pthread_cleanup_pop(execute);
```

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "\_np"(nonportable: 이식성 없음)가 붙어 있다.

## SEE ALSO

<tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_cleanup_push(3)]]</tt>, <tt>[[pthread_setcancelstate(3)]]</tt>, <tt>[[pthread_testcancel(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
