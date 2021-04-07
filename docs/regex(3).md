## NAME

regcomp, regexec, regerror, regfree - POSIX 정규 표현식 함수

## SYNOPSIS

```c
#include <sys/types.h>
#include <regex.h>

int regcomp(regex_t *preg, const char *regex, int cflags);

int regexec(const regex_t *preg, const char *string, size_t nmatch,
            regmatch_t pmatch[], int eflags);

size_t regerror(int errcode, const regex_t *preg, char *errbuf,
                size_t errbuf_size);

void regfree(regex_t *preg);
```

## DESCRIPTION

### POSIX 정규 표현식 컴파일

`regcomp()`를 사용해 이어지는 `regexec()` 검색에 적합한 형태로 정규 표현식을 컴파일 한다.

`regcomp()`에 주는 `preg`는 패턴 버퍼 저장 공간에 대한 포인터이고, `regex`는 널 종료 문자열에 대한 포인터, `cflags`는 컴파일 방식을 결정하는 플래그들이다.

모든 정규 표현식 검색은 컴파일 된 패턴 버퍼를 통해 이뤄져야 한다. 따라서 `regexec()`에는 항상 `regcomp()`로 초기화 한 패턴 버퍼의 주소를 줘야 한다.

`cflags`에는 다음을 0개 이상 비트 OR 할 수 있다.

<dl>
<dt><code>REG_EXTENDED</code></dt>
<dd><code>regex</code>를 해석할 때 <strong>POSIX</strong> 확장 정규 표현식 문법을 쓴다. 설정돼 있지 않으면 <strong>POSIX</strong> 기본 정규 표현식 문법을 쓴다.</dd>

<dt><code>REG_ICASE</code></dt>
<dd>대소문자를 구별하지 않는다. 이 패턴 버퍼를 쓰는 후속 <code>regexec()</code>에서 대소문자를 무시하게 된다.</dd>

<dt><code>REG_NOSUB</code></dt>
<dd>일치 위치를 알려 주지 않는다. 이 플래그를 설정해서 컴파일 한 패턴 버퍼를 쓰면 <code>regexec()</code>에서 <code>nmatch</code> 및 <code>pmatch</code> 인자를 무시한다.</dd>

<dt><code>REG_NEWLINE</code></dt>
<dd>

임의 문자 일치 연산자가 개행에 일치하지 않는다.

개행을 포함하지 않은 비일치 목록(`[^...]`)이 개행에 일치하지 않는다.

`regexec()` 실행 플래그 `eflags`에 `REG_NOTBOL`이 있는지와 무관하게 행 시작 일치 연산자(`^`)가 개행 바로 다음의 빈 문자열에 일치한다.

`eflags`에 `REG_NOTEOL`이 있는지와 무관하게 행 종료 연산자(`$`)가 개행 바로 전의 빈 문자열에 일치한다.
</dd>
</dl>

### POSIX 정규 표현식 일치 검사

`regexec()`를 사용해 미리 컴파일 한 패턴 버퍼 `preg`에 널 종료 문자열을 맞춰 본다. `nmatch`와 `pmatch`를 통해 일치 위치들에 대한 정보를 제공한다. `eflags`에는 `REG_NOTBOL`과 `REG_NOTEOL`을 비트 OR 할 수 있는데 일치 방식이 아래 설명처럼 바뀐다.

<dl>
<dt><code>REG_NOTBOL</code></dt>
<dd>행 시작 일치 연산자가 항상 일치에 실패한다. (하지만 위의 컴파일 플래그 <code>REG_NEWLINE</code> 참고.) 문자열을 부분 부분씩 <code>regexec()</code>로 전달하기 때문에 문자열 시작을 행의 시작으로 해석하지 말아야 할 때 이 플래그를 쓸 수 있다.</dd>

<dt><code>REG_NOTEOL</code></dt>
<dd>행 종료 일치 연산자가 항상 일치에 실패한다. (하지만 위의 컴파일 플래그 <code>REG_NEWLINE</code> 참고.)</dd>
</dl>

### 바이트 위치

패턴 버퍼를 컴파일 할 때 `REG_NOSUB`를 설정한 경우가 아니면 일치 위치 정보를 얻는 게 가능하다. `pmatch`는 최소 `nmatch` 개 항목이 들어가는 크기여야 한다. `regexec()`에서 `pmatch`에 부분열 일치 위치를 채운다. `i` 번째 열린 괄호에서 시작하는 부분식의 위치가 `pmatch[i]`에 저장된다. 그리고 전체 정규 표현식의 일치 위치가 `pmatch[0]`에 저장된다. (하위식 일치 위치 `N` 개를 받으려면 `nmatch`가 최소 `N+1`이어야 한다.) 안 쓰인 구조체 항목에는 -1 값이 담기게 된다.

`pmatch`의 타입인 `regmatch_t` 구조체는 `<regex.h>`에 정의돼 있다.

```c
typedef struct {
    regoff_t rm_so;
    regoff_t rm_eo;
} regmatch_t;
```

-1 아닌 `rm_so` 항목은 문자열 내에서 가장 큰 다음 일치 부분열의 시작 위치를 나타낸다. `rm_eo` 항목은 일치 부분 끝을 상대적으로 나타내는데, 일치 텍스트 다음 첫 문자의 위치이다.

### POSIX 오류 보고

`regerror()`를 이용해 `regcomp()` 및 `regexec()`가 반환하는 오류 코드를 오류 메시지 문자열로 바꾼다.

오류 코드 `errcode`, 패턴 버퍼 `preg`, 문자열 버퍼에 대한 포인터 `errbuf`, 그 문자열 버퍼의 크기 `errbuf_size`를 `regerror()`에 준다. 그러면 널 종료 오류 메시지 문자열을 담는 데 필요한 `errbuf`의 크기를 반환한다. `errbuf`와 `errbuf_size` 모두 0이 아니면 `errbuf`에 오류 메시지의 처음 `errbuf_size - 1` 개 문자와 종료 널 바이트('\0')를 채운다.

### POSIX 패턴 버퍼 해제

컴파일 된 패턴 버퍼 `preg`를 `regfree()`에 주면 `regcomp()`에서 컴파일 하며 패턴 버퍼에 할당한 메모리를 해제한다.

## RETURN VALUE

`regcomp()`는 컴파일 성공 시 0을 반환하며 실패 시 오류 코드를 반환한다.

`regexec()`는 일치 성공 시 0을 반환하며 실패 시 `REG_NOMATCH`를 반환한다.

## ERRORS

`regcomp()`가 다음 오류를 반환할 수 있다.

<dl>
<dt><code>REG_BADBR</code></dt>
<dd>역참조 연산자 사용 오류.</dd>
<dt><code>REG_BADPAT</code></dt>
<dd>그룹이나 목록 같은 패턴 연산자 사용 오류.</dd>
<dt><code>REG_BADRPT</code></dt>
<dd>'*'를 첫 문자로 쓰는 것 같은 반복 연산자 사용 오류.</dd>
<dt><code>REG_EBRACE</code></dt>
<dd>중괄호 구간 연산자 짝 안 맞음.</dd>
<dt><code>REG_EBRACK</code></dt>
<dd>대괄호 목록 연산자 짝 안 맞음.</dd>
<dt><code>REG_ECOLLATE</code></dt>
<dd>조합 요소 오류.</dd>
<dt><code>REG_ECTYPE</code></dt>
<dd>문자 클래스 이름 잘못됨.</dd>
<dt><code>REG_EEND</code></dt>
<dd>불특정 오류. POSIX.2에는 규정돼 있지 않음.</dd>
<dt><code>REG_EESCAPE</code></dt>
<dd>끝에 백슬래시.</dd>
<dt><code>REG_EPAREN</code></dt>
<dd>괄호 그룹 연산자 짝 안 맞음.</dd>
<dt><code>REG_ERANGE</code></dt>
<dd>범위 연산자 사용 오류. 예를 들어 범위 끝점이 시작점 앞에 나옴.</dd>
<dt><code>REG_ESIZE</code></dt>
<dd>컴파일 된 정규 표현식에 64 kB 넘는 패턴 버퍼가 필요함. POSIX.2에는 규정돼 있지 않음.</dd>
<dt><code>REG_ESPACE</code></dt>
<dd>regex 루틴 실행 중 메모리 부족.</dd>
<dt><code>REG_ESUBREG</code></dt>
<dd>잘못된 하위식 역참조.</dd>
</dl>

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `regcomp()`, `regexec()` | 스레드 안전성 | MT-Safe locale |
| `regerror()` | 스레드 안전성 | MT-Safe env |
| `regfree()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## SEE ALSO

`grep(1)`, <tt>[[regex(7)]]</tt>

glibc 매뉴얼 <em>Regular Expressions</em> 절

----

2019-03-06
