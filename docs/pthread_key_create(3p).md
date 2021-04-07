## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_key_create - 스레드별 데이터 키 생성

## SYNOPSIS

```c
#include <pthread.h>

int pthread_key_create(pthread_key_t *key, void (*destructor)(void*));
```

## DESCRIPTION

`pthread_key_create()` 함수는 프로세스 내 모든 스레드에게 보이는 스레드별 데이터 키를 생성한다. `pthread_key_create()`가 내놓는 키 값은 스레드별 데이터 위치를 얻는 데 쓰는 불투명 객체이다. 여러 스레드에서 같은 키 값을 사용할 수 있으며, `pthread_setspecific()`으로 키에 결속한 값은 스레드별로 관리되고 호출 스레드의 수명 동안 유지된다.

키 생성 시 모든 활성 스레드에서 새 키에 NULL 값이 연계된다. 스레드 생성 시 새 스레드에서 모든 정의된 키에 NULL 값이 연계된다.

각 키 값에 선택적으로 소멸자 함수를 연계할 수 있다. 스레드 종료 시 키 값에 NULL 아닌 소멸자 포인터가 연계되어 있으면 키의 값을 NULL로 설정한 다음에 이전에 연계되어 있던 값을 유일한 인자로 하여 그 함수를 호출한다. 스레드 종료 시 여러 소멸자가 존재하는 경우에 소멸자 호출 순서는 명세되어 있지 않다.

연계 소멸자 있는 모든 NULL 아닌 값들에 대해 소멸자를 모두 호출한 후에도 연계 소멸자 있는 NULL 아닌 값이 있으면 그 과정을 반복한다. 미처리 상태인 NULL 아닌 값들에 대한 소멸자 호출 과정을 최소 `PTHREAD_DESTRUCTOR_ITERATIONS`번 반복한 후에도 연계 소멸자 있는 NULL 아닌 값이 있는 경우 구현에서는 소멸자 호출을 중단할 수도 있고, 또는 무한 루프를 유발할 수 있더라도 연계 소멸자 있는 NULL 아닌 값이 더는 존재하지 않을 때까지 소멸자 호출을 계속할 수도 있다.

## RETURN VALUE

성공 시 `pthread_key_create()` 함수는 새로 생성한 키 값을 `*key`에 저장하고 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_key_create()` 함수가 실패한다.

<dl>
<dt><code>EAGAIN</code></dt>
<dd>시스템에서 스레드별 데이터 키를 하나 더 만드는 데 필요한 자원이 부족하거나, 프로세스당 키 총개수에 대한 시스템 제한인 <code>PTHREAD_KEYS_MAX</code>를 초과했다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>키를 생성하기에 충분한 메모리가 없다.</dd>
</dl>

`pthread_key_create()` 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

다음 예의 함수는 최초 호출 때 스레드별 데이터 키를 초기화 하며, 필요시 초기화를 해서 스레드별 객체를 호출 스레드에 연결해 준다.

```c
static pthread_key_t key;
static pthread_once_t key_once = PTHREAD_ONCE_INIT;

static void
make_key()
{
    (void) pthread_key_create(&key, NULL);
}

func()
{
    void *ptr;

    (void) pthread_once(&key_once, make_key);
    if ((ptr = pthread_getspecific(key)) == NULL) {
        ptr = malloc(OBJECT_SIZE);
        ...
        (void) pthread_setspecific(key, ptr);
    }
    ...
}
```

참고로 `pthread_getspecific()`이나 `pthread_setspecific()`을 쓰기 전에 키를 초기화 해야 한다. 모듈 초기화 루틴에서 명시적으로 `pthread_key_create()` 호출을 할 수도 있고 이 예에서처럼 최초 호출에서 묵시적으로 할 수도 있다. 초기화 전에 키를 사용하려는 시도는 모두 프로그래밍 오류이며, 그래서 아래 코드는 잘못된 것이다.

```c
static pthread_key_t key;

func()
{
    void *ptr;

    /* 키 초기화 안 됐음! 동작 안 함! */
    if ((ptr = pthread_getspecific(key)) == NULL &&
        pthread_setspecific(key, NULL) != 0) {
        pthread_key_create(&key, NULL);
        ...
    }
}
```

## APPLICATION USAGE

없음.

## RATIONALE

### 소멸자 함수

보통 특정 스레드에 대해 키에 결속되는 값은 그 호출 스레드를 위해 동적으로 할당한 저장 공간을 가리키는 포인터이다. `pthread_key_create()`에 지정하는 소멸자 함수는 스레드가 끝날 때 그 공간을 해제하기 위한 것이다. 여기에 스레드 취소 정리 핸들러를 이용할 수는 없는데, 취소 정리 핸들러가 동작하는 문법적 스코프 밖에서 스레드별 데이터가 계속 존재할 수도 있기 때문이다.

스레드 동작 기간 중에 어떤 키에 연계된 값을 갱신해야 하는 경우 새 값을 결속하기 전에 이전 값에 연계된 저장 공간을 해제해야 할 수도 있다. 이를 `pthread_setspecific()` 함수에서 자동으로 해 줄 수도 있겠지만 그로 인한 복잡성 추가를 정당화 할 만큼 자주 필요한 기능이 아니다. 대신 프로그래머가 이전 공간 해제를 책임진다.

```c
pthread_getspecific(key, &old);
new = allocate();
destructor(old);
pthread_setspecific(key, new);
```

**주의**: 비동기 취소를 켜고서 실행하면 위 예에서 저장 공간 누수가 일어날 수도 있다. get과 set 사이에 어떤 취소점도 없다면 기본 취소 상태에서 그런 문제가 발생하지 않는다.

소멸자에 안전한 함수라는 개념은 없다. 응용이 시그널 핸들러에서 `pthread_exit()`를 호출하지 않는다면, 또는 핸들러에서 `pthread_exit()`를 호출할 수도 있는 시그널을 비동기 안전하지 않은 함수를 호출하는 동안 막아 둔다면 소멸자에서 모든 함수들을 안전하게 호출할 수 있다.

### 비멱등 데이터 키 생성

`pthread_key_create()`을 어떤 `key` 주소 매개변수에 대해 멱등이게 만들자는 요청이 있었다. 그러면 응용에서 어떤 `key` 주소에 대해 `pthread_key_create()`을 여러 번 호출할 수 있게 되고 키가 한 개만 생성된다고 보장된다. 그렇게 하려면 키 값을 미리 (아마도 컴파일 시점에) 널 값으로 초기화 해 두어야 할 것이다. 그리고 키가 정확히 한 개만 생성되도록 보장하기 위해 `key` 매개변수의 주소와 내용에 따라 묵시적으로 상호 배제를 수행해야 할 것이다.

불행히도 묵시적 상호 배제는 `pthread_key_create()`에 국한되지 않을 것이다. 많은 구현에서는 저장 도중이거나 아직 보이지 않는 키 값 사용을 막기 위해 `pthread_getspecific()` 및 `pthread_setspecific()`에서도 묵시적 상호 배제를 수행해야 할 것이다. 이 때문에 중요한 작업, 특히 `pthread_getspecific()`의 비용이 상당히 올라갈 수 있을 것이다.

그래서 그 제안을 거부하였다. `pthread_key_create()` 함수에서는 어떤 묵시적 동기화도 수행하지 않는다. 키마다 정확히 한 번, 키 사용 전에 그 함수가 호출되도록 하는 것은 프로그래머의 책임이다. 이를 위해 사용할 수 있는 메커니즘들이 이미 다양하게 있는데, 명시적인 모듈 초기화 함수 호출하기, 뮤텍스 사용하기, `pthread_once()` 사용하기 등이 포함된다. 이렇게 하는 것이 프로그래머에게 전혀 큰 부담이 되지 않으며, 혼동을 줄 수도 있는 *임시변통* 묵시적 동기화 메커니즘을 전혀 도입하지 않으며, 자주 쓰는 스레드별 데이터 작업들이 더 효율적으로 될 가능성이 생긴다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_getspecific()|pthread_getspecific(3p)]]</tt>, <tt>[[pthread_key_delete()|pthread_key_delete(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
