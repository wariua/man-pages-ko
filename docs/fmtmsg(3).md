## NAME

fmtmsg - 형식 있는 오류 메시지 출력하기

## SYNOPSIS

```c
#include <fmtmsg.h>

int fmtmsg(long classification, const char *label,
           int severity, const char *text,
           const char *action, const char *tag);
```

## DESCRIPTION

이 함수는 그 인자들이 기술하는 메시지를 `classification` 인자로 나타낸 장치(들)에 표시한다. `stderr`로 찍는 메시지는 `MSGVERB` 환경 변수에 따라 형식이 달라진다.

`label` 인자는 메시지의 원천을 식별해 준다. 문자열은 콜론으로 구분된 두 부분으로 되어 있어야 한다. 첫 번째 부분은 10글자를 넘을 수 없고 두 번째 부분은 14글자를 넘을 수 없다.

`text` 인자는 오류의 상태를 기술한다.

`action` 인자는 오류를 복구하기 위해 할 수 있는 조치를 기술한다. 이를 찍는 경우 앞에 "TO FIX: "가 붙는다.

`tag` 인자는 추가 정보를 찾을 수 있는 온라인 문서에 대한 참조이다. `label`의 값과 고유한 식별 번호를 담아야 할 것이다.

### 더미 인자

인자 각각이 더미 값을 가질 수 있다. 더미 분류 값인 `MM_NULLMC`(`0L`)는 어떤 출력도 나타내지 않고, 그래서 아무 것도 찍히지 않는다. 더미 심각도 값인 `NO_SEV`(`0`)는 어떤 심각도도 제공하지 않는다는 말이다. `MM_NULLLBL`, `MM_NULLTXT`, `MM_NULLACT`, `MM_NULLTAG`는 `((char *) 0)`, 즉 빈 문자열과 동의어이며 `MM_NULLSEV`는 `NO_SEV`와 동의어이다.

### `classification` 인자

`classification` 인자는 4가지 종류의 정보를 기술하는 값들을 합친 것이다.

첫 번째 값은 출력 채널을 규정한다.

<dl>
<dt><code>MM_PRINT</code></dt><dd><code>stderr</code>로 출력.</dd>
<dt><code>MM_CONSOLE</code></dt><dd>시스템 콘솔로 출력.</dd>
<dt><code>MM_PRINT | MM_CONSOLE</code></dt><dd>둘 모두로 출력.</dd>
</dl>

두 번째 값은 오류의 원천이다.

<dl>
<dt><code>MM_HARD</code></dt><dd>하드웨어 오류가 발생했음.</dd>
<dt><code>MM_FIRM</code></dt><dd>펌웨어 오류가 발생했음.</dd>
<dt><code>MM_SOFT</code></dt><dd>소프트웨어 오류가 발생했음.</dd>
</dl>

세 번째 값은 어디서 문제를 탐지했는지 기록한다.

<dl>
<dt><code>MM_APPL</code></dt><dd>응용이 탐지했음.</dd>
<dt><code>MM_UTIL</code></dt><dd>유틸리티가 탐지했음.</dd>
<dt><code>MM_OPSYS</code></dt><dd>운영 체제가 탐지했음.</dd>
</dl>

네 번째 값은 사건의 심각도를 보여 준다.

<dl>
<dt><code>MM_RECOVER</code></dt><dd>복구 가능한 오류임.</dd>
<dt><code>MM_NRECOV</code></dt><dd>복구 불가능한 오류임.</dd>
</dl>

### `severity` 인자

`severity` 인자는 다음 값들 중 하나를 받을 수 있다.

<dl>
<dt><code>MM_NOSEV</code></dt><dd>심각도를 찍지 않음.</dd>
<dt><code>MM_HALT</code></dt><dd>이 값은 `HALT`라고 찍음.</dd>
<dt><code>MM_ERROR</code></dt><dd>이 값은 `ERROR`라고 찍음.</dd>
<dt><code>MM_WARNING</code></dt><dd>이 값은 `WARNING`이라고 찍음.</dd>
<dt><code>MM_INFO</code></dt><dd>이 값은 `INFO`라고 찍음.</dd>
</dl>

숫자 값으로는 0에서 4까지이다. <tt>[[addseverity(3)]]</tt>나 환경 변수 `SEV_LEVEL`을 사용하면 수준과 찍을 문자열을 더 추가할 수 있다.

## RETURN VALUE

함수가 4가지 값을 반환할 수 있다.

<dl>
<dt><code>MM_OK</code></dt><dd>모든 게 잘 돌아갔음.</dd>
<dt><code>MM_NOTOK</code></dt><dd>완전한 실패.</dd>
<dt><code>MM_NOMSG</code></dt><dd><code>stderr</code>에 쓰는 중에 오류.</dd>
<dt><code>MM_NOCON</code></dt><dd> 콘솔에 쓰는 중 오류.</dd>
</dl>

## ENVIRONMENT

환경 변수 `MSGVERB`("message verbosity")를 이용해 `stderr`로의 출력 일부를 숨길 수 있다. (콘솔로의 출력에는 영향을 주지 않는다.) 이 변수가 정의되어 있고, NULL이 아니고, 유효한 키워드들의 콜론 구분 목록이면 메시지에서 그 키워드들에 대응하는 부분들만 찍힌다. 유효한 키워드는 "`label`", "`severity`", "`text`", "`action`", "`tag`"이다.

환경 변수 `SEV_LEVEL`을 이용해 새로운 심각도 수준을 도입할 수 있다. 기본적으로는 위에서 기술한 다섯 가지 심각도 수준만 사용할 수 있다. 다른 숫자 값을 사용하면 `fmtmsg()`가 아무 것도 찍지 않을 것이다. 하지만 사용자가 첫 번째 `fmtmsg()` 호출 전에 프로세스의 환경에 다음과 같은 형식으로 `SEV_LEVEL`을 넣어 주면 `fmtmsg()`가 (표준 수준 0~4에 더해서) 지정한 값들도 받아들이게 되어 그 수준의 오류 발생 시 지정한 출력 문자열을 사용하게 된다.

```
SEV_LEVEL=[description[:description[:...]]]
```

`description` 각각은 다음 형식이다.

```
severity-keyword,level,printstring
```

`severity-keyword` 부분은 `fmtmsg()`에서 사용하지 않지만 그래도 있어야 한다. `level` 부분은 숫자를 문자열로 표현한 것이다. 그 숫자 값은 4보다 커야 한다. 이 수준을 선택하려면 `fmtmsg()`의 `severity` 인자로 이 값을 사용해야 한다. 이미 정의된 수준들을 덮어쓰는 것은 불가능하다. `printstring`은 `fmtmsg()`에서 이 수준의 메시지를 처리할 때 찍히는 문자열이다.

## VERSIONS

glibc 버전 2.1부터 `fmtmsg()`를 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `fmtmsg()` | 스레드 안전성 | glibc >= 2.16: MT-Safe<br>glibc < 2.16: MT-Unsafe |

glibc 2.16 전에서 `fmtmsg()` 함수는 보호가 되지 않는 정적 변수를 사용하며, 그래서 스레드 안전하지 않다.

glibc 2.16부터 `fmtmsg()` 함수는 락을 사용해 그 정적 변수를 보호하며, 그래서 스레드 안전하다.

## CONFORMING TO

함수 `fmtmsg()` 및 <tt>[[addseverity(3)]]</tt>, 그리고 환경 변수 `MSGVERB` 및 `SEV_LEVEL`은 시스템 V에서 온 것이다.

함수 `fmtmsg()`와 환경 변수 `MSGVERB`를 POSIX.1-2001 및 POSIX.1-2008에서 기술하고 있다.

## NOTES

시스템 V 및 UnixWare 맨 페이지에서는 이 함수들이 "`pfmt` 및 `addsev()`" 내지 "`pfmt()`, `vpfmt()`, `lfmt()`, `vlfmt()`"으로 대체되었으며 나중에 제거될 것이라고 얘기한다.

## EXAMPLE

```c
#include <stdio.h>
#include <stdlib.h>
#include <fmtmsg.h>

int
main(void)
{
    long class = MM_PRINT | MM_SOFT | MM_OPSYS | MM_RECOVER;
    int err;

    err = fmtmsg(class, "util-linux:mount", MM_ERROR,
                "unknown mount option", "See mount(8).",
                "util-linux:mount:017");
    switch (err) {
    case MM_OK:
        break;
    case MM_NOTOK:
        printf("Nothing printed\n");
        break;
    case MM_NOMSG:
        printf("Nothing printed to stderr\n");
        break;
    case MM_NOCON:
        printf("No console output\n");
        break;
    default:
        printf("Unknown error from fmtmsg()\n");
    }
    exit(EXIT_SUCCESS);
}
```

출력이 다음과 같을 것이다:

```
util-linux:mount: ERROR: unknown mount option
TO FIX: See mount(8).  util-linux:mount:017
```

다음과 같이 한 후에는:

```sh
MSGVERB=text:action; export MSGVERB
```

출력이 다음과 같이 된다:

```
unknown mount option
TO FIX: See mount(8).
```

## SEE ALSO

<tt>[[addseverity(3)]]</tt>, <tt>[[perror(3)]]</tt>

----

2017-09-15
