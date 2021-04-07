## NAME

environ - 사용자 환경

## SYNOPSIS

```c
extern char **environ;
```

## DESCRIPTION

변수 `environ`은 문자열 포인터들의 배열을 가리키는데 이를 "환경"이라고 한다. 그 배열의 마지막 포인터는 NULL 값이다. (사용자 프로그램 내에서 이 변수를 선언해야 한다. 단 기능 확인 매크로 `_GNU_SOURCE`가 정의돼 있으면 헤더 파일 `<unistd.h>`에 선언돼 있다.) <tt>[[exec(3)]]</tt> 호출에서 프로세스를 시작할 때 프로세스에서 사용 가능하도록 이 문자열 배열을 준비해 준다. <tt>[[fork(2)]]</tt>를 통해 자식 프로세스가 만들어질 때 부모 환경의 *사본을* 물려받는다.

관행상 `environ`의 문자열들은 "`name=value`" 형태다. 자주 쓰는 것들로 다음이 있다.

<dl>
<dt><code>USER</code></dt>
<dd>로그인 한 사용자의 이름. (일부 BSD 유래 프로그램들에서 사용.)</dd>

<dt><code>LOGNAME</code></dt>
<dd>로그인 한 사용자의 이름. (일부 시스템 V 유래 프로그램들에서 사용.)</dd>

<dt><code>HOME</code></dt>
<dd>사용자의 로그인 디렉터리. 암호 파일 <code>passwd(5)</code>에서 가져와서 <code>login(1)</code>에서 설정.</dd>

<dt><code>LANG</code></dt>
<dd>로캘 카테고리들에 사용할 로캘 이름. <code>LC_ALL</code>이, 또는 더 상세한 환경 변수 <code>LC_COLLATE</code>, <code>LC_CTYPE</code>, <code>LC_MESSAGES</code>, <code>LC_MONETARY</code>, <code>LC_NUMERIC</code>, <code>LC_TIME</code> 등이 있으면 그게 우선시됨. (<code>LC_*</code> 환경 변수들에 대한 자세한 내용은 <tt>[[locale(7)]]</tt> 참고.)</dd>

<dt><code>PATH</code></dt>
<dd><code>sh(1)</code> 및 기타 여러 프로그램에서 경로명을 불완전하게 알고 있는 파일을 탐색할 때 적용하는 디렉터리 선두부들의 목록. 선두부들이 ':'로 구분돼 있다. (이와 비슷하게 일부 셸에는 디렉터리 변경 명령 대상을 찾는 데 쓰는 <code>CDPATH</code>가 있고, <code>man(1)</code>에서 매뉴얼 페이지를 찾는 데 쓰는 <code>MANPATH</code>가 있기도 하다.)</dd>

<dt><code>PWD</code></dt>
<dd>현재 작업 디렉터리. 일부 셸에서 설정해 준다.</dd>

<dt><code>SHELL</code></dt>
<dd>사용자 로그인 셸의 경로명.</dd>

<dt><code>TERM</code></dt>
<dd>출력 방식을 맞출 터미널 종류.</dd>

<dt><code>PAGER</code></dt>
<dd>텍스트 파일 표시를 위한 사용자 선호 유틸리티.</dd>

<dt><code>EDITOR</code>/<code>VISUAL</code></dt>
<dd>텍스트 파일 편집을 위한 사용자 선호 유틸리티.</dd>
</dl>

`sh(1)`에서는 `export` 명령으로, `csh(1)`을 쓴다면 `setenv` 명령으로 셸의 환경에 이름을 넣을 수 있다.

다양한 방식으로 셸의 최초 환경이 채워진다. 예를 들어 (`pam(8)`을 쓰는 시스템에서는) 모든 사용자의 로그인 시점에 `pam_env(8)`에서 `/etc/environment`의 정의들을 가져와서 처리한다. 그리고 시스템 전역 `/etc/profile` 스크립트나 사용자별 초기화 스크립트 같은 여러 셸 초기화 스크립트에 셸의 환경에 변수를 추가하는 명령이 있을 수 있다. 자세한 내용은 사용하는 셸의 매뉴얼 페이지를 보라.

본(Bourne) 스타일 셸에서는 다음 문법을 지원한다.

```
NAME=value command
```

이를 이용해 `command`를 실행하는 프로세스로 범위로 한정되는 환경 변수 정의를 만들 수 있다. `command` 앞에 공백으로 구분해서 정의가 여러 개 올 수 있다.

<tt>[[exec(3)]]</tt> 시점에 환경에 인자들이 들어갈 수도 있다. C 프로그램에서 <tt>[[getenv(3)]]</tt>, <tt>[[putenv(3)]]</tt>, <tt>[[setenv(3)]]</tt>, <tt>[[unsetenv(3)]]</tt> 함수를 사용해 자기 환경을 조작할 수 있다.

참고로 특정 환경 변수의 값이나 존재 여부가 여러 프로그램 및 라이브러리 루틴의 동작에 영향을 준다. 예로 다음이 있다.

* 변수 `LANG`, `LANGUAGE`, `NLSPATH`, `LOCPATH`, `LC_ALL`, `LC_MESSAGES` 등이 로캘 처리에 영향을 준다. <tt>[[catopen(3)]]</tt>, <tt>[[gettext(3)]]</tt>, <tt>[[locale(7)]]</tt> 참고.

* `TMPDIR`이 <tt>[[tempnam(3)]]</tt> 등의 루틴에서 만드는 이름의 경로 선두부와 `sort(1)` 등의 프로그램에서 쓰는 임시 디렉터리에 영향을 준다.

* `LD_LIBRARY_PATH`, `LD_PRELOAD`, 여타 `LD_*` 변수들이 동적 로더/링커의 동작에 영향을 준다.

* `POSIXLY_CORRECT`가 특정 프로그램 및 라이브러리 루틴들이 POSIX 규정을 따르게 만든다.

* `MALLOC_*` 변수들이 <tt>[[malloc(3)]]</tt>의 동작에 영향을 준다.

* `HOSTALIASES` 변수로 지정한 파일에 담긴 별명들을 <tt>[[gethostbyname(3)]]</tt>에서 사용한다.

* `TZ` 및 `TZDIR`의 시간대 정보를 <tt>[[tzset(3)]]</tt>에서, 그리고 그 함수를 이용하는 <tt>[[ctime(3)]]</tt>, <tt>[[localtime(3)]]</tt>, <tt>[[mktime(3)]]</tt>, <tt>[[strftime(3)]]</tt> 같은 함수들에서 사용한다. `tzselect(8)`도 참고.

* `TERMCAP`은 해당 터미널을 어떻게 다룰지에 대한 정보를 (또는 그런 정보를 담은 파일 이름을) 준다.

* `COLUMNS`와 `LINES`는 창 크기를 응용에게 알려 준다. 실제 크기과 다르게 지정할 수도 있다.

* `PRINTER`나 `LPDEST`로 사용하려는 프린터를 나타낼 수 있다. `lpr(1)` 참고.

## NOTES

<tt>[[prctl(2)]]</tt>의 `PR_SET_MM_ENV_START` 및 `PR_SET_MM_ENV_END` 동작을 이용해 프로세스 환경의 위치를 제어할 수 있다.

## BUGS

명백히 여기에는 보안상의 위험이 있다. `IFS`나 `LD_LIBRARY_PATH`에 이상한 값을 지정한 사용자에게 속아서 여러 시스템 명령들이 오동작 하곤 했다.

이름 공간 오염 위험도 있다. `make`나 `autoconf` 같은 프로그램에선 환경에 대문자로 된 비슷한 이름의 변수를 써서 기본 유틸리티의 이름을 바꿀 수 있다. 가령 `CC`를 이용해 원하는 C 컴파일러를 선택한다. (비슷하게 `MAKE`, `AR`, `AS`, `FD`, `LD`, `LEX`, `RM`, `YACC` 등이 있다.) 하지만 일부 전통적 용법에서는 그런 환경 변수로 경로명이 아니라 프로그램 옵션을 지정한다. 가령 `MORE`, `LESS`, `GZIP` 등이 있다. 이 용법은 잘못된 것이고 새로운 프로그램에서는 피해야 한다. `gzip`의 저자들은 옵션 이름을 `GZIP_OPT`로 바꾸는 걸 생각해 봐야 한다.

## SEE ALSO

`bash(1)`, `csh(1)`, `env(1)`, `login(1)`, `printenv(1)`, `sh(1)`, `tcsh(1)`, <tt>[[execve(2)]]</tt>, <tt>[[clearenv(3)]]</tt>, <tt>[[exec(3)]]</tt>, <tt>[[getenv(3)]]</tt>, <tt>[[putenv(3)]]</tt>, <tt>[[setenv(3)]]</tt>, <tt>[[unsetenv(3)]]</tt>, <tt>[[locale(7)]]</tt>, <tt>[[ld.so(8)]]</tt>, `pam_env(8)`

----

2017-09-15
