## NAME

c++filt - C++ 및 자바 심볼 디맹글하기

## SYNOPSIS

<pre>
c++filt [<strong>-_</strong>|<strong>--strip-underscore</strong>]
        [<strong>-n</strong>|<strong>--no-strip-underscore</strong>]
        [<strong>-p</strong>|<strong>--no-params</strong>]
        [<strong>-t</strong>|<strong>--types</strong>]
        [<strong>-i</strong>|<strong>--no-verbose</strong>]
        [<strong>-r</strong>|<strong>--no-recurse-limit</strong>]
        [<strong>-R</strong>|<strong>--recurse-limit</strong>]
        [<strong>-s</strong> <em>format</em>|<strong>--format=</strong><em>format</em>]
        [<strong>--help</strong>]  [<strong>--version</strong>]  [<em>symbol</em>...]
</pre>

## DESCRIPTION

C++ 및 자바 언어에서는 함수 오버로딩이 가능한데, 같은 이름으로 함수를 여러 가지로 작성해서 각 함수가 다른 타입의 매개변수들을 받게 할 수 있다는 뜻이다. 이런 같은 이름의 함수들을 구별할 수 있도록 하기 위해 C++과 자바에서는 함수 이름을 각 버전을 유일하게 구별해 주는 저수준 어셈블러 이름으로 변환한다. 이 동작을 *맹글(mangle)*이라고 한다. `c++file` [1] 프로그램은 반대 동작을 한다. 즉 저수준 이름을 사용자 수준 이름으로 해독(*디맹글(demangle)*)해서 사용자가 읽을 수 있게 만들어 준다.

입력에서 (문자, 숫자, 밑줄, 달러, 마침표로 이뤄진) 모든 단어가 맹글된 이름 후보들이다. 이름이 C++ 이름으로 해독되면 그 C++ 이름이 출력에서 저수준 이름 대신 나오고, 아니면 원래 단어가 출력된다. 그래서 맹글된 이름들을 담은 어셈블러 소스 파일 전체를 `c++filt`에 통과시키면 디맹글된 이름을 담은 동일한 소스 파일을 볼 수 있다.

명령행에서 심볼 이름을 `c++filt`에 줘서 개별 심볼을 해독할 수도 있다.

```text
c++filt <symbol>
```

`symbol` 인자가 주어지지 않으면 `c++filt`는 표준 입력에서 심볼 이름을 읽어 들인다. 그리고 모든 결과를 표준 출력으로 찍는다. 명령행에서 이름을 읽는 것과 표준 입력에서 이름을 읽는 것의 차이는 명령행 인자가 딱 맹글된 이름일 것이라고 기대하여 주변 텍스트와 구별하기 위한 검사를 수행하지 않는다는 점이다. 그래서 다음은 잘 동작해서 이름이 "f()"으로 디맹글되는 반면,

```text
c++filt -n -Z1fv
```

다음은 제대로 되지 않는다. (맹글된 이름 끝에 추가로 붙어 있는 쉼표 때문에 유효하지 않은 이름이 된다.)

```text
c++filt -n -Z1fv,
```

하지만 다음 명령은 잘 동작하게 된다.

```text
echo _Z1fv, | c++filt -n
```

"f(),"가 표시된다. 즉 디맹글된 이름 다음에 마지막의 쉼표가 따라온다. 이런 동작 방식이 있는 이유는 표준 입력에서 이름을 읽을 때는 그 내용이 어셈블러 소스 파일의 일부일 수도 있다고 예상할 수 있으며, 그 경우 맹글된 이름 뒤에 이름과 상관없는 문자가 추가로 있을 수도 있기 때문이다. 예:

```text
    .type   _Z1fv, @function
```

## OPTIONS

`-_`<br>`--strip-underscore`
:   어떤 시스템에서는 C 컴파일러와 C++ 컴파일러 모두 모든 이름 앞에 밑줄을 넣는다. 예를 들어 C 이름 "foo"가 저수준 이름 "\_foo"가 된다. 이 옵션은 그 앞쪽 밑줄을 없애게 한다. 지정하지 않았을 때 `c++filt`가 그 밑줄을 없애는지 여부는 대상에 따라 다르다.

`-n`<br>`--no-strip-underscore`
:   앞쪽 밑줄을 없애지 않는다.

`-p`<br>`--no-params`
:   함수 이름을 디맹글할 때 함수 매개변수 타입을 표시하지 않는다.

`-t`<br>`--types`
:   함수 이름뿐 아니라 타입까지 디맹글하려고 시도한다. 이 동작은 기본적으로 꺼져 있는데, 보통은 컴파일러 내부적으로만 맹글된 타입을 이용하는 데다가 맹글 안 한 이름과 혼동할 수 있기 때문이다. 예를 들어 "a"라는 함수를 맹글된 타입으로 해석하면 "signed char"로 디맹글된다.

`-i`<br>`--no-verbose`
:   디맹글된 출력에 구현 상세 정보를 포함시키지 않는다.

`-r`<br>`-R`<br>`--recurse-limit`<br>`--no-recurse-limit`<br>`--recursion-limit`<br>`--no-recursion-limit`
:   문자열 해독 중 수행하는 재귀 횟수에 대한 제한을 켜거나 끈다. 이름 해독 형식에서 무한 단계의 재귀가 가능하기 때문에 해독으로 인해 호스트 머신의 가용 스택 공간이 고갈돼서 메모리 폴트가 일어나도록 문자열을 만드는 게 가능하다. 제한 동작은 재귀 중첩을 2048단계로 제한해서 그런 일이 일어나지 않게 한다.

    기본은 제한을 켜는 것이지만 정말로 복잡한 이름을 해독하기 위해선 끌 필요가 있을 수도 있다. 하지만 재귀 제한이 꺼져 있으면 스택 고갈이 가능하고, 그런 경우에 대한 버그 보고는 거부된다는 점에 유의하라.

    `-r` 옵션은 `--no-recurse-limit` 옵션과 의미가 같다. `-R` 옵션은 `--recurse-limit` 옵션과 의미가 같다.

`-s format`<br>`--format=format`
:   `c++filt`는 여러 컴파일러들에서 쓰는 다양한 맹글 방식을 해독할 수 있다. 이 옵션 인자로 사용 방식을 선택한다.

    "auto"
    :   실행 파일에 따라서 자동 선택 (기본 방법)

    "gnu"
    :   GNU C++ 컴파일러(g++)에서 쓰는 방식

    "lucid"
    :   Lucid 컴파일러(lcc)에서 쓰는 방식

    "arm"
    :   C++ Annotated Reference Manual에서 명세한 방식

    "hp"
    :   HP 컴파일러(aCC)에서 쓰는 방식

    "edg"
    :   EDG 컴파일러에서 쓰는 방식

    "gnu-v3"
    :   GNU C++ 컴파일러(g++) V3 ABI에서 쓰는 방식

    "java"
    :   GNU 자바 컴파일러(gcj)에서 쓰는 방식

    "gnat"
    :   GNU Ada 컴파일러(GNAT)에서 쓰는 방식

`--help`
:   `c++filt` 옵션 요약 설명을 찍고 종료한다.

`--version`
:   `c++filt`의 버전 번호를 찍고 종료한다.

`@file`
:   `file`에서 명령행 옵션들을 읽는다. 읽어 들인 옵션들이 원래 `@file` 옵션 자리에 들어간다. `file`이 존재하지 않거나 읽을 수 없는 경우에는 이 옵션을 제거하지 않고 문자 그대로 다루게 된다.

    `file` 내의 옵션들은 공백으로 구분한다. 옵션에 공백 문자를 포함시키려면 옵션 전체를 작은따옴표나 큰따옴표로 감싸면 된다. 문자 앞에 백슬래시를 붙이면 (백슬래시를 포함한) 어떤 문자든 집어넣을 수 있다. `file` 자체에 다시 `@file` 옵션이 있을 수 있다. 그러면 재귀적으로 처리가 이뤄진다.

## FOOTNOTES

1. MS-DOS에서는 파일 이름에 "+" 문자를 허용하지 않으므로 MS-DOS에서 이 프로그램의 이름은 `CXXFILT`이다.

## SEE ALSO

info *binutils* 항목.

## COPYRIGHT

Copyright (c) 1991-2020 Free Software Foundation, Inc.

Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.3 or any later version published by the Free Software Foundation; with no Invariant Sections, with no Front-Cover Texts, and with no Back-Cover Texts.  A copy of the license is included in the section entitled "GNU Free Documentation License".

----

2020-09-21

binutils-2.35.1
