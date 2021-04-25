## NAME

stdarg, va_start, va_arg, va_end, va_copy - 가변 인자 목록

## SYNOPSIS

```c
#include <stdarg.h>

void va_start(va_list ap, last);
type va_arg(va_list ap, type);
void va_end(va_list ap);
void va_copy(va_list dest, va_list src);
```

## DESCRIPTION

가변 타입의 가변 개수 인자로 함수를 호출할 수 있다. 포함 파일 `<stdarg.h>`에서 타입 `va_list`를 선언하고 세 가지 매크로를 정의하는데, 이를 이용해 피호출 함수에게 개수와 타입이 알려져 있지 않은 인자들의 목록을 순회할 수 있다.

피호출 함수에서 `va_list` 타입 객체를 선언해야 하며 그 객체를 매크로 `va_start()`, `va_arg()`, `va_end()`에서 사용한다.

### `va_start()`

`va_start()` 매크로는 이후 `va_arg()` 및 `va_end()`가 쓸 수 있게 `ap`를 초기화 하며, 따라서 가장 먼저 호출해야 한다.

인자 `last`는 가변 인자 목록 전의 마지막 인자, 즉 호출 함수에서 타입을 알고 있는 마지막 인자의 이름이다.

이 인자의 주소를 `va_start()` 매크로 내에서 사용할 수도 있으므로 레지스터 변수로 선언되어 있거나 함수나 배열 타입으로 선언되어 있어선 안 된다.

### `va_arg()`

`va_arg()` 매크로는 호출의 다음 인자의 타입과 값을 가진 식으로 확장된다. 인자 `ap`는 `va_start()`로 초기화 한 `va_list ap`이다. `va_arg()`를 호출할 때마다 `ap`를 변경해서 다음 호출 때 다음 인자를 반환하도록 한다. 인자 `type`은 타입 이름으로, `type`에 \*만 붙이면 지정한 타입의 객체에 대한 포인터 타입을 얻을 수 있도록 지정한다.

`va_start()` 매크로 다음으로 처음 `va_arg()` 매크로를 쓰면 `last` 다음 인자를 반환한다. 이어지는 호출이 나머지 인자들의 값을 반환한다.

다음 인자가 없거나 `type`이 (기본 인자 승격 방식에 따라 승격된) 실제 다음 인자의 타입과 호환되지 않는 경우 정해져 있지 않은 오류가 발생하게 된다.

`ap`를 어떤 함수로 전달하고 그 함수에서 `va_arg(ap,type)`을 사용하는 경우에 함수 반환 후 `ap`의 값은 규정되어 있지 않다.

### `va_end()`

각 `va_start()` 호출에는 같은 함수 내에 대응하는 `va_end()` 호출이 있어야 한다. `va_end(ap)` 호출 후 변수 `ap`는 규정되어 있지 않다. `va_start()`와 `va_end()`로 각각 감싸서 목록을 여러 번 순회하는 것이 가능하다. `va_end()`가 매크로일 수도 있고 함수일 수도 있다.

### `va_copy()`

`va_copy()` 매크로는 (앞서 초기화 한) 가변 인자 목록 `src`를 `dest`로 복사한다. `dest`에 같은 `last` 인자로 `va_start()`를 적용하고 이어서 현재 `src` 상태에 도달하기까지 한 것과 같은 횟수의 `va_arg()` 호출을 적용한 것처럼 동작한다.

단순 명백한 구현 방식에서는 `va_list`가 가변 인자 함수의 스택 프레임에 대한 포인터일 것이다. 그런 (가장 흔한) 방식에서는 다음 할당이 아무 문제가 없을 것이다.

```c
va_list aq = ap;
```

하지만 안타깝게도 `va_list`를 포인터의 (길이 1인) 배열로 만드는 시스템도 있으며, 거기선 다음과 같이 해야 한다.

```c
va_list aq;
*aq = *ap;
```

그리고 레지스터로 인자를 전달하는 시스템에서는 `va_start()`에서 메모리를 할당해서 인자들을 저장하고 `va_arg()`가 목록을 순회할 수 있도록 다음 인자가 뭔지 표시도 해 두어야 할 수도 있다. 그럼 그 할당 메모리를 `va_end()`에서 다시 해제할 수 있다. 이런 상황에 대응하기 위해 C99에서는 매크로 `va_copy()`를 추가하여 위 할당을 다음 코드로 대체할 수 있도록 한다.

```c
va_list aq;
va_copy(aq, ap);
...
va_end(aq);
```

각 `va_copy()` 호출에는 같은 함수 내에 대응하는 `va_end()` 호출이 있어야 한다. 일부 시스템에서는 `va_copy()`를 제공하지 않고 대신 `__va_copy`가 있는데, 제안 초안에서 사용했던 이름이다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `va_start()`, `va_end()`,<br>`va_copy()` | 스레드 안전성 | MT-Safe |
| `va_arg()` | 스레드 안전성 | MT-Safe race:ap |

## CONFORMING TO

매크로 `va_start()`, `var_arg()`, `va_end()`는 C89를 따른다. C99에서 `va_copy()` 매크로를 정의한다.

## BUGS

과거의 **varargs** 매크로와 달리 **stdarg** 매크로에서는 고정 인자 없는 함수를 작성하는 것이 불가능하다. 이 문제가 일을 만드는 건 주로 **varargs** 코드를 **stdarg** 코드로 변환할 때지만 `va_list` 인자를 받는 <tt>[[vfprintf(3)]]</tt> 같은 함수로 모든 인자를 전달하려 하는 가변 인자 함수에서도 어려움이 생긴다.

## EXAMPLES

함수 `foo`는 서식 문자열을 받아서 타입에 따라 각 서식 문자에 연계된 인자를 찍는다.

```c
#include <stdio.h>
#include <stdarg.h>

void
foo(char *fmt, ...)   /* '...'은 C의 가변 함수 문법 */
{
    va_list ap;
    int d;
    char c;
    char *s;

    va_start(ap, fmt);
    while (*fmt)
        switch (*fmt++) {
        case 's':              /* 문자열 */
            s = va_arg(ap, char *);
            printf("string %s\n", s);
            break;
        case 'd':              /* int */
            d = va_arg(ap, int);
            printf("int %d\n", d);
            break;
        case 'c':              /* char */
            /* 완전히 승격된 타입을 va_arg가
               받으므로 캐스트 필요 */
            c = (char) va_arg(ap, int);
            printf("char %c\n", c);
            break;
        }
    va_end(ap);
}
```

## SEE ALSO

<tt>[[vprintf(3)]]</tt>, <tt>[[vscanf(3)]]</tt>, <tt>[[vsyslog(3)]]</tt>

----

2021-03-22
