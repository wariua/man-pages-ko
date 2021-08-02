## NAME

lslocks - 로컬 시스템의 락 나열

## SYNOPSIS

<pre>
<strong>lslocks</strong> [options]
</pre>

## DESCRIPTION

`lslocks`는 리눅스 시스템에서 현재 잡혀 있는 모든 파일 락에 대한 정보를 나열한다.

lslocks에서 OFD(열린 파일 기술 항목) 락도 나열하는데, 그 락은 프로세스에 연계돼 있지 않고 (PID가 -1) 열린 파일 기술 항목에 연계돼 있다. 리눅스 3.15부터 OFD 락을 사용할 수 있는데, 자세한 내용은 <tt>[[fcntl(2)]]</tt>을 보라.

## OPTIONS

`-b`, `--bytes`
:   SIZE 열을 사람이 읽기 좋은 형식 대신 바이트 단위로 찍기.

`-i`, `--noinaccessible`
:   현재 사용자에게 접근 불가능한 락 파일 무시.

`-J`, `--json`
:   JSON 출력 형식 사용.

`-n`, `--noheadings`
:   헤더 행 찍지 않기.

`-o`, `--output list`
:   어떤 출력 열을 찍을지 지정한다. 지원하는 열들의 전체 목록을 `--help`로 볼 수 있다.

    `list`를 `+list` 형식으로 지정하면 (예: `lslocks -o +BLOCKER`) 기본 열 목록에서 확장할 수 있다.

`--output-all`
:   가능한 모든 열 출력하기.

`-p`, `--pid pid`
:   이 `pid`의 프로세스가 잡은 락만 표시하기.

`-r`, `--raw`
:   무가공 출력 형식 사용.

`-u`, `--notruncate`
:   열의 텍스트 잘라내지 않기.

`-V`, `-version`
:   버전 정보 찍고 끝내기.

`-h`, `--help`
:   도움말 찍고 끝내기.

## OUTPUT

COMMAND
:   락을 잡고 있는 프로세스의 명령 이름.

PID
:   락을 잡고 있는 프로세스의 프로세스 ID. OFDLCK에선 -1.

TYPE
:   락 종류. FLOCK (<tt>[[flock(2)]]</tt>으로 생성)이나 POSIX (<tt>[[fcntl(2)]]</tt>이나 <tt>[[lockf(3)]]</tt>로 생성), 또는 OFDLCK (<tt>[[fcntl(2)]]</tt>로 생성).

SIZE
:   잠긴 파일의 크기.

MODE
:   락 접근 권한 (읽기, 쓰기). 프로세스가 블록돼서 그 락을 기다리고 있으면 모드 뒤에 '\*'(별표)가 붙는다.

M
:   락이 강제형인지 여부. 아니면 (락이 권고형이면) 0, 맞으면 1. (<tt>[[fcntl(2)]]</tt> 참고.)

START
:   락의 상대적 바이트 오프셋.

END
:   락의 끝 오프셋.

PATH
:   락의 전체 경로. 경로가 없거나 경로를 읽을 권한이 없으면 장치 마운트 지점 경로에 "..."를 덧붙여서 찍는다. 경로가 잘려 있을 수도 있다. `--notruncate`로 전체 경로를 볼 수 있다.

BLOCKER
:   락을 막고 있는 프로세스의 PID.

## NOTES

`lslocks` 명령은 `lslk(8)` 명령을 대체하기 위한 것이다. Victor A. Abell <abe@purdue.edu>이 처음 작성했으며 2001년부터 유지 보수가 이뤄지지 않고 있다.

## AUTHORS

Davidlohr Bueso <dave@gnu.org>

## SEE ALSO

`flock(1)`, <tt>[[fcntl(2)]]</tt>, <tt>[[lockf(3)]]</tt>

## AVAILABILITY

lslocks 명령은 util-linux 패키지의 일부이며 <https://www.kernel.org/pub/linux/utils/util-linux/>에서 구할 수 있다.

----

2014년 12월
