## NAME

gdb - GNU 디버거

## SYNOPSIS

<pre>
gdb [<strong>-help</strong>] [<strong>-nh</strong>] [<strong>-nx</strong>] [<strong>-q</strong>] [<strong>-batch</strong>] [<strong>-cd=</strong><em>dir</em>] [<strong>-f</strong>] [<strong>-b</strong> <em>bps</em>]
    [<strong>-tty=</strong><em>dev</em>] [<strong>-s</strong> <em>symfile</em>] [<strong>-e</strong> <em>prog</em>] [<strong>-se</strong> <em>prog</em>] [<strong>-c</strong> <em>core</em>] [<strong>-p</strong> <em>procID</em>]
    [<strong>-x</strong> <em>cmds</em>] [<strong>-d</strong> <em>dir</em>] [<em>prog</em>|<em>prog procID</em>|<em>prog core</em>]
</pre>

## DESCRIPTION

GDB 같은 디버거의 목적은 프로그램 실행 중에 그 "내부"에서 무슨 일이 일어나고 있는지를, 또는 프로그램이 죽는 순간에 뭘 하고 있었는지를 알 수 있게 해 주는 것이다.

GDB는 크게 다음 네 가지를 (그리고 이를 지원하며 다른 것들을) 해서 현장에서 버그 잡는 걸 돕는다.

* 프로그램을 시작한다. 동작에 영향을 끼칠 수 있을 무엇이든 지정할 수 있다.

* 지정한 조건에서 프로그램이 정지하게 한다.

* 무슨 일이 일어났는지를 프로그램 정지 상태에서 조사한다.

* 프로그램 내의 이것 저것을 바꾼다. 그래서 한번 어떤 버그의 효과를 바로잡고서 다른 버그를 살펴볼 수 있게 해 준다.

GDB를 사용해 C, C++, Fortran, Modula-2로 작성된 프로그램을 디버그 할 수 있다.

셸 명령 "gdb"로 GDB를 실행한다. 시작하고 나면 GDB 명령 "quit"으로 나가라고 할 때까지 터미널에서 명령들을 읽어 들인다. "help" 명령을 쓰면 GDB 내장 도움말을 볼 수 있다.

아무 인자나 옵션 없이도 "gdb"를 실행할 수 있다. 하지만 GDB를 실행할 때 가장 흔한 방식은 인자 한 개나 두 개로 실행 프로그램을 지정하는 것이다.

```
gdb program
```

실행 프로그램과 코어 파일을 모두 지정해서 시작할 수도 있다.

```
gdb program core
```

또는 실행 중인 프로세스를 디버그 하고 싶다면 두 번째 인자로 프로세스 ID를 지정할 수 있다.

```
gdb program 1234
gdb -p 1234
```

그러면 GDB가 프로세스 1234에 붙게 된다. (단 `1234`라는 파일이 없어야 한다. GDB는 코어 파일을 먼저 확인한다.) `-p` 옵션을 쓰면 파일명 `program`을 생략할 수 있다.

다음은 자주 쓰는 몇 가지 GDB 명령들이다.

`break [file:]function`
:   (`file` 내의) `function`에 중지점을 설정한다.

`run [arglist]`
:   (`arglist`로) 프로그램을 시작한다.

`bt`
:   백트레이스. 프로그램 스택을 표시한다.

`print expr`
:   식의 값을 표시한다.

`c`
:   (중지점 등에서 멈춘 상태에서) 프로그램 실행을 계속한다.

`next`
:   (멈춘 상태에서) 프로그램 다음 행을 실행한다. 행에 함수 호출이 있으면 실행하고 지나간다.

`edit [file:]function`
:   현재 멈춰 있는 프로그램 행을 편집기로 본다.

`list [file:]function`
:   현재 멈춰 있는 곳 부근의 프로그램 텍스트를 표시한다.

`step`
:   (멈춘 상태에서) 프로그램 다음 행을 실행한다. 행에 함수 호출이 있으면 안으로 들어간다.

`help [name]`
:   GDB 명령 `name`에 대한 정보나 GDB 사용에 대한 일반적 정보를 표시한다.

`quit`
:   GDB에서 나간다.

GDB에 대한 자세한 내용은 Richard M. Stallman과 Roland H. Pesch의 *Using GDB: A Guide to the GNU Source-Level Debugger*를 보라. "info" 프로그램에서 "gdb" 항목으로 같은 내용을 볼 수 있다.

## OPTIONS

옵션이 아닌 인자가 있으면 실행 파일과 코어 파일을 (또는 프로세스ID를) 지정하게 된다. 즉 연관된 옵션 플래그 없는 첫 번째 인자는 `-se` 옵션과 같으며 두 번째가 있고 파일 이름이면 `-c` 옵션과 같다. 여러 옵션들에는 긴 형태와 짧은 형태가 있다. 긴 형태는 다른 옵션과 구분이 가능하기만 하다면 일부만 입력하더라도 인식한다. (원한다면 옵션 인자에 - 대신 +를 붙일 수도 있지만 아래에선 일반적 관행을 따른다.)

모든 옵션과 명령행 인자들은 순차적으로 처리된다. `-x` 옵션 사용 시 순서에 따라 차이가 생긴다.

`-help`<br>`-h`
:   모든 옵션들을 간단한 설명과 함께 나열한다.

`-symbols=file`<br>`-s file`
:   파일 `file`에서 심볼 테이블을 읽어 들인다.

`-write`
:   실행 파일과 코어 파일로 쓰기를 할 수 있게 한다.

`-exec=file`<br>`-e file`
:   파일 `file`을 실행 파일로 사용해서 적절한 때에 실행하고 코어 덤프와 조합해서 초기 데이터를 알아내는 데 쓴다.

`-se=file`
:   파일 `file`에서 심볼 테이블을 읽어 들이고 실행 파일로 사용한다.

`-core=file`<br>`-c file`
:   파일 `file`을 코어 덤프로 사용해서 조사한다.

`-command=file`<br>`-x file`
:   파일 `file`에 있는 GDB 명령들을 실행한다.

`-ex command`
:   주어진 GDB 명령 `command`를 실행한다.

`-directory=directory`<br>`-d directory`
:   소스 파일 탐색 경로에 `directory`를 추가한다.

`-nh`
:   `~/.gdbinit`의 명령을 실행하지 않는다.

`-nx`<br>`-n`
:   초기화 파일 `.gdbinit`의 명령을 실행하지 않는다.

`-quiet`<br>`-q`
:   조용히 한다. 소개 및 저작권 메시지를 찍지 않는다. 배치 모드에서도 이 메시지들을 숨긴다.

`-batch`
:   배치 모드로 실행한다. `-x`로 지정한 명령 파일들 모두를 (그리고 금지하지 않았다면 `.gdbinit`을) 처리한 후에 상태 0으로 끝낸다. 명령 파일 내의 GDB 명령을 실행하다가 오류가 발생하면 0 아닌 상태 값으로 끝낸다.

    GDB를 필터로 실행할 때, 예를 들어 다른 컴퓨터의 프로그램을 내려받아서 돌릴 때 배치 모드가 유용할 수 있다. 더 편리하도록 배치 모드로 돌 때는 다음 메시지를 찍지 않는다. (보통 때는 GDB 제어 하에서 도는 프로그램이 종료할 때마다 찍는다.)

        Program exited normally.

`-cd=directory`
:   현재 디렉터리 대신 `directory`를 작업 디렉터리로 해서 GDB를 실행한다.

`-fullname`<br>`-f`
:   Emacs에서 GDB를 하위 프로세스로 실행할 때 이 옵션을 설정한다. GDB에서 스택 프레임을 표시할 때(프로그램이 멈출 때 포함)마다 전체 파일명과 행 번호를 표준적이고 인식 가능한 방식으로 출력하도록 한다. 그 인식 가능한 형식이란 `\032` 문자 두 개 다음에 파일 이름이 오고, 콜론으로 구분된 행 번호와 문자 위치가 오고, 개행이 오는 것이다. Emacs에서 GDB로의 인터페이스 프로그램에서는 `\032` 문자 두 개를 프레임 소스 코드를 표시하라는 신호로 쓴다.

`-b bps`
:   GDB에서 원격 디버깅에 쓰는 시리얼 인터페이스의 회선 속도(보드 속도, 즉 초당 비트)를 설정한다.

`-tty=device`
:   장치 `device`를 프로그램의 표준 입출력으로 사용해서 실행한다.

## SEE ALSO

완전한 GDB 문서는 Texinfo 설명서 형태로 유지한다. "info"와 "gdb" 프로그램, 그리고 GDB의 Texinfo 문서가 올바로 설치돼 있다면 다음 명령으로 완전한 설명서를 볼 수 있다.

```
info gdb
```

*Using GDB: A Guide to the GNU Source-Level Debugger*, Richard M. Stallman and Roland H. Pesch, 1991년 7월.

## COPYRIGHT

Copyright (c) 1988-2018 Free Software Foundation, Inc.

Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.3 or any later version published by the Free Software Foundation; with the Invariant Sections being "Free Software" and "Free Software Needs Free Documentation", with the Front-Cover Texts being "A GNU Manual," and with the Back-Cover Texts as in (a) below.

(a) The FSF's Back-Cover Text is: "You are free to copy and modify this GNU Manual.  Buying copies from GNU Press supports the FSF in developing GNU and promoting software freedom."

----

2018-04-09
