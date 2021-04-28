## NAME

addr2line - 주소를 파일 이름과 행 번호로 변환하기

## SYNOPSIS

<pre>
addr2line [<strong>-a</strong>|<strong>--addresses</strong>]
          [<strong>-b</strong> <em>bfdname</em>|<strong>--target=</strong><em>bfdname</em>]
          [<strong>-C</strong>|<strong>--demangle</strong>[=<em>style</em>]]
          [<strong>-r</strong>|<strong>--no-recurse-limit</strong>]
          [<strong>-R</strong>|<strong>--recurse-limit</strong>]
          [<strong>-e</strong> <em>filename</em>|<strong>--exe=</strong><em>filename</em>]
          [<strong>-f</strong>|<strong>--functions</strong>] [<strong>-s</strong>|<strong>--basename</strong>]
          [<strong>-i</strong>|<strong>--inlines</strong>]
          [<strong>-p</strong>|<strong>--pretty-print</strong>]
          [<strong>-j</strong>|<strong>--section=</strong><em>name</em>]
          [<strong>-H</strong>|<strong>--help</strong>] [<strong>-V</strong>|<strong>--version</strong>]
          [addr addr ...]
</pre>

## DESCRIPTION

`addr2line`은 주소를 파일 이름과 행 번호로 바꿔 준다. 실행 파일 내 주소나 재배치 가능 오브젝트 섹션 내 오프셋을 받아서 디버깅 정보를 이용해 연계된 파일 이름과 행 번호를 알아낸다.

사용할 실행 파일 내지 재배치 가능 오브젝트를 `-e` 옵션으로 지정한다. 기본은 `a.out` 파일이다. 사용할 재배치 가능 오브젝트 내 섹션은 `-j` 옵션으로 지정한다.

`addr2line`에는 두 가지 동작 방식이 있다.

첫 번째 방식에서는 명령행에서 16진수 주소를 지정하면 `addr2line`이 각 주소에 대한 파일 이름과 행 번호를 표시한다.

두 번째 방식에서는 `addr2line`이 표준 입력으로부터 16진수 주소를 읽어서 각 주소에 대한 파일 이름과 행 번호를 표준 출력으로 찍는다. 이 방식을 쓰면 파이프 내에 `addr2line`을 써서 동적으로 선택한 주소를 변환할 수 있다.

출력 형식은 `FILENAME:LINENO`이다. 기본적으로 각 입력 주소가 한 행씩 출력을 만들어 낸다.

각 `FILENAME:LINENO` 행 앞에 행을 추가할 수 있는 옵션이 두 가지 있다.

`-a` 옵션을 쓰면 입력 주소 행을 표시한다.

`-f` 옵션을 쓰면 `FUNCTIONNAME` 행을 표시한다. 그 주소를 담은 함수의 이름이다.

`FILENAME:LINENO` 행 뒤에 행을 추가할 수 있는 옵션이 한 가지 있다.

`-i` 옵션을 쓰면 해당 주소가 컴파일러 인라인 처리 때문에 거기 있는 것이면 뒤에 행들을 추가로 표시한다. 인라인 처리된 함수마다 한 행 또는 (`-f` 옵션을 쓴 경우) 두 행씩을 추가로 표시한다.

또는 `-p` 옵션을 쓰면 각 입력 주소마다 주소, 함수 이름, 파일 이름, 행 번호를 담은 한 줄짜리 긴 행이 출력된다. `-i`까지 썼다면 인라인 처리된 함수들도 같은 방식으로 찍히되, 별도 행에서 앞에 `(inlined by)`가 붙어서 표시된다.

파일 이름이나 함수 이름을 알아낼 수 없으면 `addr2line`에서는 그 자리에 물음표 두 개를 찍는다. 행 번호를 알아낼 수 없으면 `addr2line`에서는 0을 찍는다.

## OPTIONS

여기 같이 나와 있는 긴 옵션과 짧은 옵션은 동등하다.

`-a`<br>`--addresses`
:   함수 이름과 파일 및 행 번호 정보 앞에 주소를 표시한다. 쉽게 식별할 수 있도록 주소 앞에 `0x`를 붙여서 찍는다.

`-b bfdname`<br>`--target=bfdname`
:   오브젝트 파일의 오브젝트 코드 형식이 `bfdname`이라고 지정한다.

`-C`<br>`--demangle[=style]`
:   저수준 심볼 이름을 사용자 수준 이름으로 해독(*디맹글(demangle)*)한다. 시스템에서 앞에 붙인 밑줄을 없애는 것에 더해서 C++ 함수 이름을 읽을 수 있게 만들어 준다. 컴파일러마다 맹글링(mangling) 방식이 다르다. 선택적인 해독 방식 인자를 사용해서 자기 컴파일러에 맞는 해독 방식을 선택할 수 있다.

`-e filename`<br>`--exe=filename`
:   주소들을 변환할 실행 파일의 이름을 지정한다. 기본 파일은 `a.out`이다.

`-f`<br>`--functions`
:   파일 및 행 번호 정보와 함께 함수 이름도 표시한다.

`-s`<br>`--basenames`
:   각 파일 이름의 기본 이름만 표시한다.

`-i`<br>`--inlines`
:   주소가 인라인 처리된 함수에 속하면 첫 번째 비인라인 함수까지의 모든 감싸는 스코프에 대한 소스 정보를 함께 찍는다. 예를 들어 "main"에서 "callee1"을 인라인 하고 거기서 다시 "callee2"를 인라인 하는데 주소가 "callee2"에 있으면 "callee1" 및 "main"에 대한 소스 정보도 함께 찍힌다.

`-j`<br>`--section`
:   절대 주소가 아니라 지정한 섹션 기준 오프셋을 읽는다.

`-p`<br>`--pretty-print`
:   출력을 더 인간 친화적으로 만든다. 즉 각 위치를 한 행에 찍는다. `-i` 옵션을 지정하면 모든 감싸는 스코프 행 앞에 `(inlined by)`가 붙는다.

`-r`<br>`-R`<br>`--recurse-limit`<br>`--no-recurse-limit`<br>`--recursion-limit`<br>`--no-recursion-limit`
:   문자열 해독 중 수행하는 재귀 횟수에 대한 제한을 켜거나 끈다. 이름 해독 형식에서 무한 단계의 재귀가 가능하기 때문에 해독으로 인해 호스트 머신의 가용 스택 공간이 고갈돼서 메모리 폴트가 일어나도록 문자열을 만드는 게 가능하다. 제한 동작은 재귀 중첩을 2048단계로 제한해서 그런 일이 일어나지 않게 한다.

    기본은 제한을 켜는 것이지만 정말로 복잡한 이름을 해독하기 위해선 끌 필요가 있을 수도 있다. 하지만 재귀 제한이 꺼져 있으면 스택 고갈이 가능하고, 그런 경우에 대한 버그 보고는 거부된다는 점에 유의하라.

    `-r` 옵션은 `--no-recurse-limit` 옵션과 의미가 같다. `-R` 옵션은 `--recurse-limit` 옵션과 의미가 같다.

    `-C` 내지 `--demangle` 옵션을 켰을 때만 이 옵션이 효과가 있다는 점에 유의하라.

`@file`
:   `file`에서 명령행 옵션들을 읽는다. 읽어 들인 옵션들이 원래 `@file` 옵션 자리에 들어간다. `file`이 존재하지 않거나 읽을 수 없는 경우에는 이 옵션을 제거하지 않고 문자 그대로 다루게 된다.

    `file` 내의 옵션들은 공백으로 구분한다. 옵션에 공백 문자를 포함시키려면 옵션 전체를 작은따옴표나 큰따옴표로 감싸면 된다. 문자 앞에 백슬래시를 붙이면 (백슬래시를 포함한) 어떤 문자든 집어넣을 수 있다. `file` 자체에 다시 `@file` 옵션이 있을 수 있다. 그러면 재귀적으로 처리가 이뤄진다.

## SEE ALSO

info *binutils* 항목.

## COPYRIGHT

Copyright (c) 1991-2020 Free Software Foundation, Inc.

Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.3 or any later version published by the Free Software Foundation; with no Invariant Sections, with no Front-Cover Texts, and with no Back-Cover Texts.  A copy of the license is included in the section entitled "GNU Free Documentation License".

----

2020-09-21

binutils-2.35.1
