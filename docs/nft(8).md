## NAME

nft - 패킷 필터링 및 분류를 위한 nftables 프레임워크의 관리 도구

## SYNOPSIS

<pre>
<strong>nft</strong> [ <strong>-nNscaeSupyjt</strong> ] [ <strong>-I</strong> <em>directory</em> ] [ <strong>-f</strong> <em>filename</em> | <strong>-i</strong> | <em>cmd</em> ...]
<strong>nft -h</strong>
<strong>nft -v</strong>
</pre>

## DESCRIPTION

nft는 리눅스 커널 nftables 프레임워크의 패킷 필터링 및 분류 규칙을 설정하고 관리하고 조사하는 데 쓰는 명령행 도구다. 그 리눅스 커널 서브시스템을 nf_tables라고 하는데, 여기서 'nf'는 Netfilter를 나타낸다.

## OPTIONS

옵션 요약 전체를 보려면 `nft --help`를 실행하면 된다.

`-h`, `--help`
:   도움말 메시지와 전체 옵션을 보여 준다.

`-v`, `--version`
:   버전을 보여 준다.

`-n`, `--numeric`
:   출력을 완전히 숫자로만 찍는다.

`-s`, `--stateless`
:   규칙과 상태 객체의 상태 정보를 생략한다.

`-N`, `--reversedns`
:   DNS 역질의를 통해 IP 주소를 이름으로 변환한다. 네트워크 트래픽을 발생시키므로 목록 표시가 느려질 수 있다.

`-S`, `--service`
:   `/etc/services`에 정의된 대로 포트 번호를 서비스 이름으로 변환한다.

`-u`, `--guid`
:   `/etc/passwd` 및 `/etc/group`에 정의된 대로 숫자로 된 UID/GID를 이름으로 변환한다.

`-p`, `--numeric-protocol`
:   제4계층 프로토콜을 숫자로 표시한다.

`-y`, `--numeric-priority`
:   기본 체인 우선순위를 숫자로 표시한다.

`-c`, `--check`
:   변경 사항을 실제 적용하지 않고 명령 유효성만 확인한다.

`-a`, `--handle`
:   출력 내용에서 객체 핸들을 보여 준다.

`-e`, `--echo`
:   `add`나 `insert`, `replace` 명령으로 룰셋에 항목을 집어넣을 때 `nft monitor`처럼 알림을 찍는다.

`-j`, `--json`
:   JSON 형식으로 출력한다. 스키마 설명은 `libnftables-json(5)`을 보라.

`-I`, `--includepath directory`
:   포함 파일을 찾을 디렉터리 목록에 디렉터리 `directory`를 추가한다. 이 옵션은 여러 번 지정할 수 있다.

`-f`, `--file filename`
:   `filename`에서 입력을 읽어 들인다. `filename`이 `-`이면 stdin에서 읽는다.

`-i`, `--interactive`
:   대화형 readline CLI에서 입력을 읽어 들인다. `quit`으로 빠져나갈 수 있다. EOF 표시를 쓸 수도 있는데, 보통 CTRL-D이다.

`-T`, `--numeric-time`
:   시각, 요일, 시간 값을 숫자로 보인다.

`-t`, `--terse`
:   출력에서 집합 내용물을 생략한다.

## 입력 파일 형식

### 구문 규약

행 단위로 입력을 파싱 한다. 개행 문자 바로 앞의 행 마지막 문자가 따옴표로 감싸지 않은 백슬래시(\\)일 때는 다음 행을 계속 이어진 것처럼 처리한다. 한 행에서 여러 명령을 세미콜론(;)으로 구분할 수 있다.

해시 기호(#)로 주석이 시작된다. 그 행의 나머지 문자들을 모두 무시한다.

식별자는 알파벳 문자(a-z,A-Z)로 시작해서 0개 이상의 알파벳 문자(a-z,A-Z), 숫자(0-9), 슬래시(/), 백슬래시(\\), 밑줄(\_), 마침표(.) 문자가 온다. 다른 문자를 쓰거나 키워드와 충돌하는 식별자는 큰따옴표(")로 감싸 줘야 한다.

### 파일 포함하기

<pre>
<strong>include</strong> <em>filename</em>
</pre>

`include` 문을 써서 다른 파일을 포함할 수 있다. 포함 파일을 찾을 디렉터리들을 `-I`/`--includepath` 옵션으로 지정할 수 있다. 또한 경로 앞에 './'를 붙여서 현재 작업 디렉터리에 위치한 파일을 (즉 상대 경로로) 포함하도록 강제하거나 '/'를 써서 절대 경로로 파일 위치를 나타낼 수도 있다.

`-I`/`--includepath`를 지정하지 않으면 nft는 컴파일 시점에 지정된 기본 디렉터리를 이용한다. `-h`/`--help` 옵션을 통해 그 기본 디렉터리를 알아낼 수 있다.

include 문은 일반적인 셸 와일드카드 기호(\*,?,[])를 지원한다. include 문에 와일드카드 기호를 쓴 경우에는 include 문에 일치하는 파일이 없어도 오류가 아니다. 그래서 `include "/etc/firewall/rules/*"` 같은 문으로 포함한 디렉터리가 비어 있을 수도 있다. 와일드카드에 걸린 항목들은 알파벳 순서로 올라온다. 마침표(.)로 시작하는 파일은 include 문에 걸리지 않는다.

### 심볼 변수

<pre>
<strong>define</strong> <em>variable</em> = <em>expr</em>
<strong>$variable</strong>
</pre>

`define` 문을 써서 심볼 변수를 정의할 수 있다. 변수 참조는 식이며 이를 이용해 다른 변수를 초기화 할 수 있다. 정의 유효 범위는 현재 블록과 그 안에 포함된 모든 블록이다.

#### 심볼 변수 사용하기

```text
define int_if1 = eth0
define int_if2 = eth1
define int_ifs = { $int_if1, $int_if2 }

filter input iif $int_ifs accept
```

## 주소 패밀리

주소 패밀리에 따라 어떤 종류의 패킷이 처리되는지 정해진다. 각 주소 패밀리별로 커널 패킷 처리 경로의 특정 지점들에 소위 훅이 있어서 그 훅에 대한 규칙이 존재하면 nftables를 호출한다.

`ip`
:   IPv4 주소 패밀리

`ip6`
:   IPv6 주소 패밀리

`inet`
:   인터넷(IPv4/IPv6) 주소 패밀리

`arp`
:   ARP 주소 패밀리, IPv4 ARP 패킷 처리

`bridge`
:   브리지 주소 패밀리, 브리지 장치를 통과하는 패킷 처리

`netdev`
:   netdev 주소 패밀리, 진입점에서 패킷 처리

모든 nftables 객체는 주소 패밀리별 네임스페이스 안에 존재하며, 그래서 모든 식별자에는 주소 패밀리가 포함돼 있다. 주소 패밀리 없이 식별자를 지정하면 기본적으로 `ip` 패밀리를 쓴다.

### IPv4/IPv6/Inet 주소 패밀리

IPv4/IPv6/Inet 주소 패밀리는 IPv4 패킷, IPv6 패킷, 그리고 두 종류 모두를 다룬다. 네트워크 스택의 패킷 처리 단계 다섯 곳에 훅이 있다.

표 1: IPv4/IPv6/Inet 주소 패밀리 훅

| 훅          | 설명 |
| ----------- | ---- |
| prerouting  | 시스템에 들어오는 모든 패킷이 prerouting 훅에서 처리된다. 라우팅 처리 전에 호출되며 이르게 필터링을 하거나 라우팅에 영향을 주는 패킷 속성을 바꾸는 데 쓰인다. |
| input       | 로컬 시스템으로 전달되는 패킷이 input 훅에서 처리된다. |
| forward     | 다른 호스트로 전달되는 패킷이 forward 훅에서 처리된다. |
| output      | 로컬 프로세스가 보내는 패킷이 output 훅에서 처리된다. |
| postrouting | 시스템을 떠나는 모든 패킷이 postrouting 훅에서 처리된다. |

### ARP 주소 패밀리

ARP 주소 패밀리는 시스템이 받고 보내는 ARP 패킷들을 다룬다. 클러스터링을 위해 ARP 패킷을 조작하는 데 흔히 쓴다.

표 2: ARP 주소 패밀리 훅

| 훅     | 설명 |
| ------ | ---- |
| input  | 로컬 시스템으로 전달되는 패킷이 input 훅에서 처리된다. |
| output | 로컬 시스템에서 보내는 패킷이 output 훅에서 처리된다. |

### 브리지 주소 패밀리

브리지 주소 패밀리는 브리지 장치를 통과하는 이더넷 패킷을 다룬다.

지원하는 훅 목록은 위의 IPv4/IPv6/Inet 주소 패밀리와 동일하다.

### Netdev 주소 패밀리

Netdev 주소 패밀리는 진입점(ingress)에서 패킷을 처리한다.

표 3: Netdev 주소 패밀리 훅

| 훅      | 설명 |
| ------- | ---- |
| ingress | 시스템에 들어오는 모든 패킷이 이 훅에서 처리된다. 제3계층 프로토콜 핸들러 전에 호출되며 이른 필터링이나 폴리싱에 쓸 수 있다. |

## 룰셋

<pre>
{<strong>list</strong> | <strong>flush</strong>} <strong>ruleset</strong> [<em>family</em>]
</pre>

현재 커널 내에 위치한 테이블, 체인 등의 세트 전체를 나타내는 데 `ruleset` 키워드를 쓴다. 다음 `ruleset` 명령이 있다.

`list`
:   사람이 읽기 좋은 형식으로 룰셋을 출력한다.

`flush`
:   룰셋 전체를 비운다. iptables와 달리 모든 테이블과 그 안에 담긴 모든 걸 제거한다는 점에 유의해야 한다. 실질적으로 빈 룰셋이 되며, 어떤 패킷 필터링도 일어나지 않게 되므로 커널에서는 수신한 모든 유효 패킷을 받아들인다.

`list`와 `flush`를 특정 주소 패밀리로 한정할 수도 있다. 유효한 패밀리 이름의 목록은 위의 "주소 패밀리" 절을 보라.

설계상 `list ruleset` 명령의 출력을 `nft -f` 입력으로 쓸 수 있다. 실질적으로 `iptables-save`와 `iptables-restore`에 대응한다.

## 테이블

<pre>
{<strong>add</strong> | <strong>create</strong>} <strong>table</strong> [<em>family</em>] <em>table</em> [<strong>{ flags</strong> <em>flags</em> <strong>; }</strong>]
{<strong>delete</strong> | <strong>list</strong> | <strong>flush</strong>} <strong>table</strong> [<em>family</em>] <em>table</em>
<strong>list tables</strong> [<em>tables</em>]
<strong>delete table</strong> [<em>family</em>] <strong>handle</strong> <em>handle</em>
</pre>

테이블은 체인, 집합, 상태 객체를 담는 컨테이너다. 주소 패밀리와 이름으로 식별된다. 주소 패밀리는 `ip`, `ip6`, `inet`, `arp`, `bridge`, `netdev` 중 하나여야 한다. `inet` 주소 패밀리는 하이브리드 IPv4/IPv6 테이블을 만드는 데 쓰는 가상 패밀리다. `meta expression nfproto` 키워드를 쓰면 패킷이 (IPv4와 IPv6 중) 어느 패밀리 맥락에 있는지 검사할 수 있다. 주소 패밀리를 지정하지 않으면 기본으로 `ip`를 쓴다. add와 create의 유일한 차이는 지정한 테이블이 이미 존재하는 경우에 전자는 오류를 반환하지 않는 반면 `create`은 오류를 반환한다는 점이다.

표 4: 테이블 플래그

| 플래그    | 설명 |
| --------- | ---- |
| `dormant` | 테이블을 더 이상 평가하지 않는다. (기본 체인들을 등록 해제한다.) |

#### 테이블 추가, 변경, 삭제

```text
# 대화형으로 nft 시작
nft --interactive

# 새 테이블 생성
create table inet mytable

# 새 기본 체인 추가: 입력 패킷 받기
add chain inet mytable myin { type filter hook input priority 0; }

# 체인에 카운터 하나 추가
add rule inet mytable myin counter

# 테이블을 잠시 비활성화 -- 더는 규칙들이 평가되지 않음
add table inet mytable { flags dormant; }

# 테이블을 다시 활성화
add table inet mytable
```

`add`
:   지정한 이름으로 지정한 패밀리에 새 테이블 추가.

`delete`
:   지정한 테이블 삭제.

`list`
:   지정한 테이블의 모든 체인 및 규칙 나열.

`flush`
:   지정한 테이블의 모든 체인 및 규칙 비우기.

## 체인

<pre>
{<strong>add</strong> | <strong>create</strong>} <strong>chain</strong> [<em>family</em>] <em>table</em> <em>chain</em> [<strong>{ type</strong> <em>type</em> <strong>hook</strong> <em>hook</em> [<strong>device</strong> <em>device</em>] <strong>priority</strong> <em>priority</em> <strong>;</strong> [<strong>policy</strong> <em>policy</em> <strong>;</strong>] <strong>}</strong>]
{<strong>delete</strong> | <strong>list</strong> | <strong>flush</strong>} <strong>chain</strong> [<em>family</em>] <em>table chain</em>
<strong>list chains</strong> [<em>family</em>]
<strong>delete chain</strong> [<em>family</em>] <em>table</em> <strong>handle</strong> <em>handle</em>
<strong>rename chain</strong> [<em>family</em>] <em>table chain newname</em>
</pre>

체인은 규칙들을 담는 컨테이너다. 두 가지 종류가 있는데, 기본 체인과 일반 체인이다. 기본 체인은 네트워킹 스택에서 패킷이 진입하는 지점이다. 일반 체인은 점프 대상으로 쓸 수 있으며 규칙들로 구조를 만드는 데 쓴다.

`add`
:   지정한 테이블에 새 체인 추가. 훅과 우선순위 값을 지정하면 기본 체인으로 만들어서 네트워킹 스택에 연결한다.

`create`
:   `add` 명령과 비슷하되 체인이 이미 존재하면 오류를 반환한다.

`delete`
:   지정한 체인 삭제. 체인에 어떤 규칙도 없어야 하고 점프 대상으로 쓰이고 있지 않아야 한다.

`rename`
:   지정한 체인의 이름 변경.

`list`
:   지정한 체인의 모든 규칙 나열.

`flush`
:   지정한 체인의 모든 규칙 비우기.

기본 체인에선 `type`, `hook`, `priority` 매개변수가 필수다.

표 5: 지원하는 체인 타입

| 타입   | 패밀리        | 훅     | 설명 |
| ------ | ------------- | ------ | ---- |
| filter | 모두          | 모두   | 긴가민가할 때 쓰면 되는 표준 체인 타입. |
| nat    | ip, ip6, inet | prerouting, input, output, postrouting | 이 체인 타입에서는 conntrack 항목에 따라 네트워크 주소 변환을 수행한다. 연결의 첫 번째 패킷만 실제로 이 체인을 거친다. 체인 규칙에서는 일반적으로 생성되는 conntrack 항목의 세부 사항(예를 들어 NAT 문)을 규정한다. |
| route  | ip, ip6       | output | 패킷이 이 체인 타입을 거치고서 허용되는 경우에 IP 헤더의 관련 부분이 변경됐으면 라우트 검색을 새로 수행한다. 이를 이용해 가령 nftables에서 정책 라우팅을 할 수 있다. |

위에 설명한 특별한 경우들(가령 `nat`에서 `forward` 훅을 지원하지 않거나 `route`에서 `output` 훅만 지원하는 것) 외에도 신경 써야 할 특이 사항이 두 가지 더 있다.

* netdev 패밀리는 한 가지 조합, 즉 `filter` 타입에 `ingress` 훅만 지원한다. 또 이 패밀리의 기본 체인에는 `device` 매개변수가 꼭 있어야 하는데, 입력 인터페이스별로 체인이 존재하기 때문이다.

* arp 패밀리는 `input` 훅과 `output` 훅만 지원하며 둘 모두 `filter` 타입에서다.

`priority` 매개변수는 같은 `hook` 값의 체인들을 거치는 순서를 나타내는 부호 있는 정수 값 또는 표준 우선순위 이름을 받는다. 순서는 오름차순이다. 즉 낮은 우선순위 값이 높은 값보다 우선도가 높다.

표준 우선순위 값들 대신 쉽게 기억할 수 있는 이름을 쓸 수 있다. 모든 이름이 각 패밀리의 모든 훅에서 통하는 건 아니지만 (아래 호환성 표 참고) 그래도 그 숫자 값은 체인 우선순위 지정에 사용할 수 있다.

그 이름과 값들은 기본 체인 등록 때 xtables에서 쓰는 우선순위에 따라 정의되고 사용 가능해진다.

대부분의 패밀리에서 같은 값을 쓰지만 브리지는 다른 값을 쓴다. 값과 호환성을 기술하는 다음 두 표를 참고하라.

표 6: 표준 우선순위 이름, 패밀리, 훅 호환성 표

| 이름     | 값   | 패밀리        | 훅          |
| -------- | ---- | ------------- | ----------- |
| raw      | -300 | ip, ip6, inet | 모두        |
| mangle   | -150 | ip, ip6, inet | 모두        |
| dstnat   | -100 | ip, ip6, inet | prerouting  |
| filter   | 0    | ip, ip6, inet, arp, netdev | 모두 |
| security | 50   | ip, ip6, inet | 모두        |
| srcnat   | 100  | ip, ip6, inet | postrouting |

표 7: 브리지 패밀리의 표준 우선순위 이름과 훅 호환성

| 이름   | 값   | 훅          |
| ------ | ---- | ----------- |
| dstnat | -300 | prerouting  |
| filter | -200 | 모두        |
| out    | 100  | output      |
| srcnat | 300  | postrouting |

이 표준 이름에 간단한 산술 연산(더하기와 빼기)을 해서 상대 우선순위를 쉽게 지정할 수도 있다. 가령 `mangle - 5`는 `-155`를 나타낸다. 값을 찍을 때도 표준 값에서 10 넘게 차이가 나지 않으면 그런 식으로 찍는다.

기본 체인에는 체인 `policy`를 설정할 수도 있다. 즉 안에 담긴 규칙들에서 명시적으로 허용하지도 않고 거부하지도 않은 패킷이 어떻게 되는가이다. 지원하는 정책 값은 `accept`(기본)와 `drop`이다.

## 규칙

<pre>
{<strong>add</strong> | <strong>insert</strong>} <strong>rule</strong> [<em>family</em>] <em>table chain</em> [<strong>handle</strong> <em>handle</em> | <strong>index</strong> <em>index</em>] <em>statement</em> ... [<strong>comment</strong> <em>comment</em>]
<strong>replace rule</strong> [<em>family</em>] <em>table chain</em> <strong>handle</strong> <em>handle statement</em> ... [<strong>comment</strong> <em>comment</em>]
<strong>delete rule</strong> [<em>family</em>] <em>table chain</em> <strong>handle</strong> <em>handle</em>
</pre>

지정한 테이블의 체인에 규칙이 추가된다. 패밀리를 지정하지 않으면 ip 패밀리를 쓴다. 일군의 문법 규칙에 따라 식(expression)과 문(statement)이라는 두 가지 요소로 규칙이 구성된다.

add와 insert 명령에서는 선택적으로 위치 지정이 가능한데, 기존 규칙의 `handle`이나 (0에서 시작하는) `index`로 지정한다. 내부적으로는 항상 `handle`로 규칙 위치를 나타내며 `index`에서 변환하는 건 사용자 공간에서 이뤄진다. 때문에 변환이 이뤄진 후 동시에 룰셋 변경이 일어나는 경우 영향이 있을 수도 있다. 즉 참조 대상 규칙 앞에서 규칙이 삽입 내지 삭제되면 실제 규칙 인덱스가 바뀔 수 있다. 그리고 참조 대상 규칙이 삭제되면 유효하지 않은 `handle`을 준 경우처럼 명령을 커널에서 거부한다.

`comment`는 한 단어거나 큰 따옴표(")로 감싼 여러 단어 문자열이며 실제 규칙과 관련된 메모를 하는 데 쓸 수 있다. **주의**: 규칙 추가 시 bash를 쓴다면 따옴표에 이스케이프를 해 줘야 한다. 예: \\"enable ssh for servers\\".

`add`
:   문 목록으로 나타낸 새 규칙을 추가. 위치를 지정하지 않으면 지정한 체인에 규칙을 덧붙이고, 지정한 경우에는 지정한 규칙 뒤에 규칙을 삽입한다.

`insert`
:   `add`와 같되 체인의 처음이나 지정한 규칙 앞에 규칙을 삽입.

`replace`
:   `add`와 비슷하되 지정한 규칙을 교체.

`delete`
:   지정한 규칙 삭제.

#### ip 테이블 input 체인에 규칙 추가

```text
nft add rule filter output ip daddr 192.168.0.0/24 accept # 'ip filter' 상정
# 같은 명령을 좀 더 길게 쓰기
nft add rule ip filter output ip daddr 192.168.0.0/24 accept
```

#### inet 테이블에서 규칙 삭제

```text
# nft -a list ruleset
table inet filter {
        chain input {
                type filter hook input priority 0; policy accept;
                ct state established,related accept # handle 4
                ip saddr 10.1.1.1 tcp dport ssh accept # handle 5
          ...
# 핸들이 5인 규칙 삭제하기
# nft delete rule inet filter input handle 5
```

## 집합

nftables에는 두 가지 집합 개념이 있다. 익명 집합은 따로 이름이 없는 집합이다. 집합을 쓰는 규칙을 만들 때 집합 멤버들을 중괄호로 감싸고 쉼표로 원소들을 구분한다. 그 규칙이 제거되면 집합도 제거된다. 이 집합은 갱신이 불가능하다. 즉 익명 집합은 일단 선언하고 나면 그 익명 집합을 쓰는 규칙을 제거/변경하지 않고는 변경할 수 없다.

#### 익명 집합 이용해 특정 서브넷 및 포트 허용하기

```text
nft add rule filter input ip saddr { 10.0.0.0/8, 192.168.0.0/16 } tcp dport { 22, 443 } accept
```

기명 집합은 규칙에서 참조하기 전에 먼저 정의해야 한다. 익명 집합과 달리 언제든 기명 집합의 원소를 추가하거나 제거할 수 있다. 규칙에서 집합 이름 앞에 @를 붙여서 집합을 참조한다.

#### 기명 집합 이용해 주소 및 포트 허용하기

```text
nft add rule filter input ip saddr @allowed_hosts tcp dport @allowed_ports accept
```

집합 allowed_hosts와 allowed_ports가 먼저 만들어져 있어야 한다. 다음 절에서 nft set 문법을 더 자세히 설명한다.

<pre>
<strong>add set</strong> [<em>family</em>] <em>table set</em> <strong>{ type</strong> <em>type</em> <strong>;</strong> [<strong>flags</strong> <em>flags</em> <strong>;</strong>] [<strong>timeout</strong> <em>timeout</em> <strong>;</strong>] [<strong>gc-interval</strong> <em>gc-interval</em> <strong>;</strong>] [<strong>elements = {</strong> <em>element</em>[<strong>,</strong> ...] <strong>} ;</strong>] [<strong>size</strong> <em>size</em> <strong>;</strong>] [<strong>policy</strong> <em>policy</em> <strong>;</strong>] [<strong>auto-merge ;</strong>] <strong>}</strong>
{<strong>delete</strong> | <strong>list</strong> | <strong>flush</strong>} <strong>set</strong> [<em>family</em>] <em>table set</em>
<strong>list sets</strong> [<em>family</em>]
<strong>delete set</strong> [<em>family</em>] <em>table</em> <strong>handle</strong> <em>handle</em>
{<strong>add</strong> | <strong>delete</strong>} <strong>element</strong> [<em>family</em>] <em>table set</em> <strong>{</strong> <em>element</em>[<strong>,</strong> ...] <strong>}</strong>
</pre>

집합은 사용자 정의 데이터 타입인 원소 컨테이너다. 사용자 지정 이름으로 유일하게 식별되며 테이블에 붙는다. 집합 생성 시점에 지정할 수 있는 플래그들로 동작을 조정할 수 있다.

`add`
:   지정한 테이블에 새 집합 추가. 집합 속성을 지정하는 방법에 대해선 아래의 집합 지정 표 참고.

`delete`
:   지정한 집합 삭제.

`list`
:   지정한 집합의 원소 표시.

`flush`
:   지정한 집합의 모든 원소 제거.

`add element`
:   지정한 집합에 쉽표 구분 목록의 원소들을 추가.

`delete element`
:   지정한 집합에서 쉼표 구분 목록의 원소들을 삭제.

표 8: 집합 지정

| 키워드             | 설명 | 타입 |
| ------------------ | ---- | ---- |
| type               | 집합 원소의 데이터 타입 | 문자열: ipv4_addr, ipv6_addr, ether_addr, inet_proto, inet_service, mark |
| flags              | 집합 플래그 | 문자열: constant, dynamic, interval, timeout |
| timeout            | 집합에서 원소가 유지되는 시간. 집합이 패킷 경로(룰셋)로부터 추가되는 경우 필수. | 문자열, 십진수에 단위 붙음. 단위: d, h, m, s |
| gc&#x2011;interval | 가비지 컬렉션 간격. timeout이나 timeout 플래그가 활성일 때만 사용 가능. | 문자열, 십진수에 단위 붙음. 단위: d, h, m, s |
| elements           | 집합에 담기는 원소들 | 집합 데이터 타입 |
| size               | 집합의 최대 원소 수. 집합이 패킷 경로(룰셋)로부터 추가되는 경우 필수. | 부호 없는 정수 (64비트) |
| policy             | 집합 정책 | 문자열: performance [기본], memory |
| auto&#x2011;merge  | 인접/중첩 집합 원소 자동 병합 (interval 집합에만) | |

## 맵

<pre>
<strong>add map</strong> [<em>family</em>] <em>table map</em> <strong>{ type</strong> <em>type</em> [<strong>flags</strong> <em>flags</em> <strong>;</strong>] [<strong>elements = {</strong> <em>element</em>[<strong>,</strong> ...] <strong>} ;</strong>] [<strong>size</strong> <em>size</em> <strong>;</strong>] [<strong>policy</strong> <em>policy</em> <strong>;</strong>] <strong>}</strong>
{<strong>delete</strong> | <strong>list</strong> | <strong>flush</strong>} <strong>map</strong> [<em>family</em>] <em>table map</em>
<strong>list maps</strong> [<em>family</em>]
{<strong>add</strong> | <strong>delete</strong>} <strong>element</strong> [<em>family</em>] <em>table map</em> <strong>{ elements = {</strong> <em>element</em>[<strong>,</strong> ...] <strong>} ; }</strong>
</pre>

맵은 입력으로 하는 어떤 특정 키에 따라 데이터를 저장한다. 사용자 지정 이름으로 유일하게 식별되며 테이블에 붙는다.

`add`
:   지정한 테이블에 새 맵 추가.

`delete`
:   지정한 맵 삭제.

`list`
:   지정한 맵의 원소 표시.

`flush`
:   지정한 맵의 모든 원소 제거.

`add element`
:   지정한 맵에 쉼표 구분 목록의 원소들을 추가.

`delete element`
:   지정한 맵에서 쉼표 구분 목록의 원소들을 삭제.

표 9: 맵 지정

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| type     | 맵 원소의 데이터 타입 | 문자열 ':' 문자열: ipv4_addr, ipv6_addr, ether_addr, inet_proto, inet_service, mark, counter, quota. counter과 quota는 키로 쓸 수 없음 |
| flags    | 맵 플래그 | 문자열: constant, interval |
| elements | 맵에 담기는 원소들 | 맵 데이터 타입 |
| size     | 맵의 최대 원소 수 | 부호 없는 정수 (64비트) |
| policy   | 맵 정책 | 문자열: performance [기본], memory |

## 플로테이블

<pre>
{<strong>add</strong> | <strong>create</strong>} <strong>flowtable</strong> [<em>family</em>] <em>table flowtable</em> <strong>{ hook</strong> <em>hook</em> <strong>priority</strong> <em>priority</em> <strong>; devices = {</strong> <em>device</em>[<strong>,</strong> ...] <strong>} ; }</strong>
<strong>list flowtables</strong> [<em>family</em>]
{<strong>delete</strong> | <strong>list</strong>} <strong>flowtable</strong> {<em>family</em>] <em>table flowtable</em>
<strong>delete flowtable</strong> [<em>family</em>] <em>table</em> <strong>handle</strong> <em>handle</em>
</pre>

플로테이블을 통해 소프트웨어에서 패킷 포워딩 속도를 높일 수 있다. 입력 인터페이스, 출발 및 목적 주소, 출발 및 목적 포트, 제3/4계층 프로토콜로 이뤄진 튜플을 통해 플로테이블 항목을 나타낸다. 각 항목에는 또한 패킷을 포워딩 하기 위한 목적 인터페이스와 (링크 계층 목적 주소를 갱신하기 위한) 게이트웨이 주소를 캐싱 한다. ttl 및 hoplimit 필드도 줄어든다. 그래서 플로테이블은 패킷이 전통적 포워딩 경로를 우회할 수 있는 또 다른 경로를 제공한다. 플로테이블은 prerouting 훅 전에 있는 ingress 훅에 위치한다. forward 체인에서 flow 식을 통해 오프로드 하고 싶은 흐름을 선택할 수 있다. 플로테이블은 주소 패밀리와 이름으로 식별된다. 주소 패밀리는 `ip`, `ip6`, `inet` 중 하나여야 한다. `inet` 주소 패밀리는 하이브리드 IPv4/IPv6 테이블을 만드는 데 쓰는 가상의 패밀리다. 패밀리를 지정하지 않으면 기본으로 `ip`를 쓴다.

`priority`는 부호 있는 정수나 (0을 나타내는) `filter`일 수 있다. 더하기와 빼기를 써서 상대 우선순위를 지정할 수 있다. 가령 `filter + 5`는 `5`를 나타낸다.

`add`
:   지정한 패밀리에 지정한 이름으로 새 플로테이블 추가.

`delete`
:   지정한 플로테이블 삭제.

`list`
:   모든 플로테이블 나열.

## 상태 객체

<pre>
{<strong>add</strong> | <strong>delete</strong> | <strong>list</strong> | <strong>reset</strong>} <em>type</em> [<em>family</em>] <em>table object</em>
<strong>delete</strong> <em>type</em> [<em>family</em>] <em>table</em> <strong>handle</strong> <em>handle</em>
<strong>list counters</strong> [<em>family</em>]
<strong>list quotas</strong> [<em>family</em>]
</pre>

상태 객체는 테이블에 붙으며 유일한 이름으로 식별된다. 규칙들에서 상태 정보를 모은 것이며 규칙에서 참조하려면 "타입 이름" 키워드를 쓴다. 가령 "counter 이름"으로 쓴다.

`add`
:   지정한 테이블에 새 상태 객체 추가.

`delete`
:   지정한 객체 삭제.

`list`
:   객체가 담은 상태 정보 표시.

`reset`
:   상태 객체 나열 및 재설정.

### ct helper

<pre>
<strong>ct helper</strong> <em>helper</em> <strong>{ type</strong> <em>type</em> <strong>protocol</strong> <em>protocol</em> <strong>;</strong> [<strong>l3proto</strong> <em>family</em> <strong>;</strong>] <strong>}</strong>
</pre>

ct helper로 연결 추적 헬퍼를 정의하며, 그걸 `ct helper set` 문으로 사용할 수 있다. `type`과 `protocol`은 필수고 l3proto는 지정하지 않으면 테이블 패밀리로 정한다. 즉 inet 테이블에서는 커널에서 지원하면 ipv4 및 ipv6 헬퍼 백엔드를 모두 적재하려고 시도하게 된다.

표 10: conntrack 헬퍼 지정

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| type     | 헬퍼 타입 이름 | 따옴표 친 문자열 (예: "ftp") |
| protocol | 헬퍼의 제4계층 프로토콜 | 문자열 (예: tcp) |
| l3proto  | 헬퍼의 제3계층 프로토콜 | 주소 패밀리 (예: ip) |

#### ftp 헬퍼 정의하고 할당하기

iptables와 달리 conntrack 검색이 완료된 후에, 예를 들어 기본 훅 우선순위 0으로 헬퍼 할당을 수행해야 한다.

```text
table inet myhelpers {
    ct helper ftp-standard {
        type "ftp" protocol tcp
    }
    chain prerouting {
        type filter hook prerouting priority 0;
        tcp dport 21 ct helper set "ftp-standard"
    }
}
```

### ct timeout

<pre>
<strong>ct timeout</strong> <em>name</em> <strong>{ protocol</strong> <em>protocol</em> <strong>; policy = {</strong> <em>state</em><strong>:</strong> <em>value</em> [<strong>,</strong> ...] <strong>} ;</strong> [<strong>l3proto</strong> <em>family</em> <strong>;</strong>] <strong>}</strong>
</pre>

ct timeout으로 연결 추적 타임아웃 값을 변경한다. `ct timeout set` 문으로 타임아웃 정책을 할당한다. `protocol`과 `policy`는 필수고 l3proto는 지정하지 않으면 테이블 패밀리로 정한다.

표 11: conntrack 타임아웃 지정

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| protocol | 타임아웃 객체의 제4계층 프로토콜 | 문자열 (예: tcp) |
| state    | 연결 상태 이름 | 문자열 (예: "established") |
| value    | 연결 상태의 타임아웃 값 | 부호 없는 정수 |
| l3proto  | 타임아웃 객체의 제3계층 프로토콜 | 주소 패밀리 (예: ip) |

#### ct 타임아웃 정책 정의하고 할당하기

```text
table ip filter {
        ct timeout customtimeout {
                protocol tcp;
                l3proto ip
                policy = { established: 120, close: 20 }
        }

        chain output {
                type filter hook output priority filter; policy accept;
                ct timeout set "customtimeout"
        }
}
```

#### 갱신된 타임아웃 정책 확인하기

```text
% conntrack -E
```

다음처럼 나와야 한다.

```text
[UPDATE] tcp      6 120 ESTABLISHED src=172.16.19.128 dst=172.16.19.1
sport=22 dport=41360 [UNREPLIED] src=172.16.19.1 dst=172.16.19.128
sport=41360 dport=22
```

### ct expectation

<pre>
<strong>ct expectation</strong> <em>name</em> <strong>{ protocol</strong> <em>protocol</em> <strong>; dport</strong> <em>dport</em> <strong>; timeout</strong> <em>timeout</em> <strong>; size</strong> <em>size</em> <strong>;</strong> [<strong>l3proto</strong> <em>family</em> <strong>;</strong>] <strong>}</strong>
</pre>

ct expectation으로 연결 예상을 만든다. `ct expectation set` 문으로 예상을 할당한다. `protocol`, `dport`, `timeout`, `size`는 필수고 l3proto는 지정하지 않으면 테이블 패밀리로 정한다.

표 12: 연결 예상 지정

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| protocol | 예상 객체의 제4계층 프로토콜 | 문자열 (예: tcp) |
| dport    | 예상 연결의 목적 포트 | 부호 없는 정수 |
| timeout  | 예상의 타임아웃 값 | 부호 없는 정수 |
| size     | 예상의 크기 값 | 부호 없는 정수 |
| l3proto  | 예상 객체의 제3계층 프로토콜 | 주소 패밀리 (예: ip) |

#### ct 예상 정책 정의하고 할당하기

```text
table ip filter {
        ct expectation expect {
                protocol udp
                dport 9876
                timeout 2m
                size 8
                l3proto ip
        }

        chain input {
                type filter hook input priority filter; policy accept;
                ct expectation set "expect"
        }
}
```

### counter

<pre>
<strong>counter</strong> [<em>packets bytes</em>]
</pre>

표 13: 카운터 지정

| 키워드  | 설명 | 타입 |
| ------- | ---- | ---- |
| packets | 시작 패킷 카운트 | 부호 없는 정수 (64비트) |
| bytes   | 시작 바이트 카운트 | 부호 없는 정수 (64비트) |

### quota

<pre>
<strong>quota</strong> [<strong>over</strong> | <strong>until</strong>] [<em>used</em>]
</pre>

표 14: 쿼터 지정

| 키워드 | 설명 | 타입 |
| ------ | ---- | ---- |
| quota  | 쿼터 제한, 쿼터 이름으로 사용 | 인자 둘. 부호 없는 정수(64비트)와 문자열: bytes, kbytes, mbytes. 그 인자 앞에 "over"와 "until"이 옴. |
| used   | 쿼터의 시작 값 | 인자 둘. 부호 없는 정수(64비트)와 문자열: bytes, kbytes, mbytes |

## 식

식은 값을 나타내는데, 네트워크 주소나 포트 번호 같은 상수일 수도 있고 룰셋을 평가하면서 패킷에서 수집한 데이터일 수도 있다. 이진식, 논리식, 관계식 등으로 식들을 결합해서 (검사를 위한) 복합식 내지 관계식을 만들 수 있다. NAT나 패킷 마킹 같은 특정 동작에 인자로 쓰기도 한다.

각 식에는 데이터 타입이 있고, 그에 따라 크기, 심볼 값의 파싱 및 표현 방법, 다른 식과의 타입 호환성이 결정된다.

### describe 명령

<pre>
<strong>describe</strong> <em>expression</em> | <em>data type</em>
</pre>

`describe` 명령은 식의 종류와 그 데이터 타입에 대한 정보를 보여 준다. 데이터 타입을 줄 수도 있으며, 그 경우 nft는 그 타입에 대한 추가 정보를 표시한다.

#### describe 명령

```text
$ nft describe tcp flags
payload expression, datatype tcp_flag (TCP flag) (basetype bitmask, integer), 8 bits

predefined symbolic constants:
fin                           0x01
syn                           0x02
rst                           0x04
psh                           0x08
ack                           0x10
urg                           0x20
ecn                           0x40
cwr                           0x80
```

## 데이터 타입

데이터 타입에 따라 크기, 심볼 값의 파싱 및 표현 방법, 식의 타입 호환성이 결정된다. 여러 가지 전역 데이터 타입이 있으며, 추가로 어떤 식들에서 그 식 종류에 한정된 데이터 타입을 추가로 정의한다. 대부분의 데이터 타입은 크기가 고정돼 있지만 일부는 크기가 동적일 수 있는데, 가령 문자열 타입이 그렇다.

어떤 타입에는 미리 정의된 심볼 상수들이 있다. nft `describe` 명령으로 그 상수들을 나열할 수 있다.

```text
$ nft describe ct_state
datatype ct_state (conntrack state) (basetype bitmask, integer), 32 bits

pre-defined symbolic constants (in hexadecimal):
invalid                         0x00000001
new ...
```

하위 타입에서 다른 타입이 파생될 수도 있다. 예를 들어 IPv4 주소 타입은 정수 타입에서 파생된 것인데, IPv4 주소를 정수 값으로도 나타낼 수 있다는 뜻이다.

특정 맥락(집합 및 맵 정의)에서는 데이터 타입을 명시적으로 지정해야 한다. 타입마다 있는 이름을 거기 쓴다.

### 정수 타입

| 이름 | 키워드  | 크기 | 기반 타입 |
| ---- | ------- | ---- | --------- |
| 정수 | integer | 가변 | -         |

정수 타입은 수 값에 쓴다. 10진수, 16진수, 8진수로 나타낼 수 있다. 정수 타입에는 정해진 크기가 없으며 쓰이는 식에 따라 그 크기가 결정된다.

### 비트마스크 타입

| 이름       | 키워드  | 크기 | 기반 타입 |
| ---------- | ------- | ---- | --------- |
| 비트마스크 | bitmask | 가변 | integer   |

비트마스크 타입(`bitmask`)은 비트마스크에 쓴다.

### 문자열 타입

| 이름   | 키워드 | 크기 | 기반 타입 |
| ------ | ------ | ---- | --------- |
| 문자열 | string | 가변 | -         |

문자열 타입은 문자열에 쓴다. 문자열은 알파벳 문자(a-zA-Z)로 시작하고 0개 이상의 알파벳이나 숫자, /, -, \_, . 문자가 온다. 추가로 큰괄호(")로 감싼 건 뭐든 문자열로 인식한다.

#### 문자열 표시

```text
# 인터페이스 이름
filter input iifname eth0

# 기이한 인터페이스 이름
filter input iifname "(eth0)"
```

### 링크 계층 주소 타입

| 이름           | 키워드 | 크기 | 기반 타입 |
| -------------- | ------ | ---- | --------- |
| 링크 계층 주소 | lladdr | 가변 | integer   |

링크 계층 주소 타입은 링크 계층 주소에 쓴다. 링크 계층 주소는 가변 개수의 16진수 숫자 두개 묶음을 콜론(:)으로 구분해서 나타낸다.

#### 링크 계층 주소 표시

```text
# 이더넷 목적 MAC 주소
filter input ether daddr 20:c9:d0:43:12:d9
```

### IPv4 주소 타입

| 이름      | 키워드    | 크기   | 기반 타입 |
| --------- | --------- | ------ | --------- |
| IPv4 주소 | ipv4_addr | 32비트 | integer   |

IPv4 주소 타입은 IPv4 주소에 쓴다. 점 찍은 10진수, 점 찍은 16진수, 점 찍은 8진수, 10진수, 16진수, 8진수 표기, 또는 호스트 이름으로 주소를 나타낸다. 호스트 이름은 표준 시스템 리졸버를 이용해 해석한다.

#### IPv4 주소 표시

```text
# 점 찍은 10진수 표기
filter output ip daddr 127.0.0.1

# 호스트 이름
filter output ip daddr localhost
```

### IPv6 주소 타입

| 이름      | 키워드    | 크기    | 기반 타입 |
| --------- | --------- | ------- | --------- |
| IPv6 주소 | ipv6_addr | 128비트 | integer   |

IPv6 주소 타입은 IPv6 주소에 쓴다. 호스트 이름이나 콜론으로 구분된 16진수 하프워드들로 나타낸다. 포트 번호와 구별하기 위해 주소를 대괄호("[]")로 감쌀 수도 있다.

#### IPv6 주소 표시

```text
# 축약된 루프백 주소
filter output ip6 daddr ::1
```

#### 대괄호 표기법을 쓴 IPv6 주소 표시

```text
# []가 없으면 포트 번호(22)가 ipv6 주소의 일부인 것으로
# 파싱 됨
ip6 nat prerouting tcp dport 2222 dnat to [1ce::d0]:22
```

### 불리언 타입

| 이름   | 키워드  | 크기  | 기반 타입 |
| ------ | ------- | ----- | --------- |
| 불리언 | boolean | 1비트 | integer   |

불리언 타입은 편의를 위한 사용자 공간의 문법적 타입이다. (보통 암묵적인) 관계 식의 오른쪽에 쓰여서 왼쪽 식을 불리언 (일반적으로 존재 여부) 검사로 바꾼다.

표 15: 다음 키워드들은 자동으로 해당 값의 불리언 타입으로 결정된다.

| 키워드  | 값 |
| ------- | -- |
| exists  | 1  |
| missing | 0  |

표 16: 불리언 비교를 지원하는 식들

| 식         | 동작 |
| ---------- | ---- |
| fib        | 라우트 존재 확인. |
| exthdr     | IPv6 확장 헤더 존재 확인. |
| tcp option | TCP 옵션 헤더 존재 확인. |

#### 불리언 지정

```text
# 라우트 존재하면 일치
filter input fib daddr . iif oif exists

# IPv6 트래픽 중 단편화 안 된 패킷에 일치
filter input exthdr frag missing

# TCP 타임스탬프 옵션이 있으면 일치
filter input tcp option timestamp exists
```

### ICMP 타입 타입

| 이름      | 키워드    | 크기  | 기반 타입 |
| --------- | --------- | ----- | --------- |
| ICMP 타입 | icmp_type | 8비트 | integer   |

ICMP 타입 타입은 ICMP 헤더의 type 필드를 간편하게 지정하는 데 쓴다.

표 17: ICMP 타입 지정 시 사용할 수 있는 키워드

| 키워드                  | 값 |
| ----------------------- | -- |
| echo-reply              | 0  |
| destination-unreachable | 3  |
| source-quench           | 4  |
| redirect                | 5  |
| echo-request            | 8  |
| router-advertisement    | 9  |
| router-solicitation     | 10 |
| time-exceeded           | 11 |
| parameter-problem       | 12 |
| timestamp-request       | 13 |
| timestamp-reply         | 14 |
| info-request            | 15 |
| info-reply              | 16 |
| address-mask-request    | 17 |
| address-mask-reply      | 18 |

#### ICMP 타입 지정

```text
# 핑 패킷 일치
filter output icmp type { echo-request, echo-reply }
```

### ICMP 코드 타입

| 이름      | 키워드    | 크기  | 기반 타입 |
| --------- | --------- | ----- | --------- |
| ICMP 코드 | icmp_code | 8비트 | integer   |

ICMP 코드 타입은 ICMP 헤더의 code 필드를 간편하게 지정하는 데 쓴다.

표 18: ICMP 코드 지정 시 사용할 수 있는 키워드

| 키워드           | 값 |
| ---------------- | -- |
| net-unreachable  | 0  |
| host-unreachable | 1  |
| prot-unreachable | 2  |
| port-unreachable | 3  |
| net-prohibited   | 9  |
| host-prohibited  | 10 |
| admin-prohibited | 13 |

### ICMPv6 타입 타입

| 이름        | 키워드      | 크기  | 기반 타입 |
| ----------- | ----------- | ----- | --------- |
| ICMPv6 타입 | icmpv6_type | 8비트 | integer   |

ICMPv6 타입 타입은 ICMPv6 헤더의 type 필드를 간편하게 지정하는 데 쓴다.

표 19: ICMPv6 타입 지정 시 사용할 수 있는 키워드

| 키워드                  | 값  |
| ----------------------- | --- |
| destination-unreachable | 1   |
| packet-too-big          | 2   |
| time-exceeded           | 3   |
| parameter-problem       | 4   |
| echo-request            | 128 |
| echo-reply              | 129 |
| mld-listener-query      | 130 |
| mld-listener-report     | 131 |
| mld-listener-done       | 132 |
| mld-listener-reduction  | 132 |
| nd-router-solicit       | 133 |
| nd-router-advert        | 134 |
| nd-neighbor-solicit     | 135 |
| nd-neighbor-advert      | 136 |
| nd-redirect             | 137 |
| router-renumbering      | 138 |
| ind-neighbor-solicit    | 141 |
| ind-neighbor-advert     | 142 |
| mld2-listener-report    | 143 |

#### ICMPv6 타입 지정

```text
# ICMPv6 핑 패킷 일치
filter output icmpv6 type { echo-request, echo-reply }
```

### ICMPv6 코드 타입

| 이름        | 키워드      | 크기  | 기반 타입 |
| ----------- | ----------- | ----- | --------- |
| ICMPv6 코드 | icmpv6_code | 8비트 | integer   |

ICMPv6 코드 타입은 ICMPv6 헤더의 code 필드를 간편하게 지정하는 데 쓴다.

표 20: ICMPv6 코드 지정 시 사용할 수 있는 키워드

| 키워드           | 값 |
| ---------------- | -- |
| no-route         | 0  |
| admin-prohibited | 1  |
| addr-unreachable | 3  |
| port-unreachable | 4  |
| policy-fail      | 5  |
| reject-route     | 6  |

### ICMPvX 코드 타입

| 이름        | 키워드      | 크기  | 기반 타입 |
| ----------- | ----------- | ----- | --------- |
| ICMPvX 코드 | icmpv6_type | 8비트 | integer   |

ICMPvX 코드 타입은 ICMP와 ICMPv6의 코드 타입에서 겹치는 값들을 추출한 것이며 inet 패밀리에서 쓰기 위한 것이다.

표 21: ICMPvX 코드 지정 시 사용할 수 있는 키워드

| 키워드           | 값 |
| ---------------- | -- |
| no-route         | 0  |
| port-unreachable | 1  |
| host-unreachable | 2  |
| admin-prohibited | 3  |

### conntrack 타입

표 22: ct 식과 문에 쓰는 타입들

| 이름                  | 키워드    | 크기    | 기반 타입 |
| --------------------- | --------- | ------- | --------- |
| conntrack 상태        | ct_state  | 4바이트 | bitmask   |
| conntrack 방향        | ct_dir    | 8비트   | integer   |
| conntrack 상황        | ct_status | 4바이트 | bitmask   |
| conntrack 이벤트 비트 | ct_event  | 4바이트 | bitmask   |
| conntrack 레이블      | ct_label  | 128비트 | bitmask   |

위의 타입들 각각에 대해 편의를 위한 키워드들이 있다.

표 23: conntrack 상태 (ct_state)

| 키워드      | 값 |
| ----------- | -- |
| invalid     | 1  |
| established | 2  |
| related     | 4  |
| new         | 8  |
| untracked   | 64 |

표 24: conntrack 방향 (ct_dir)

| 키워드   | 값 |
| -------- | -- |
| original | 0  |
| reply    | 1  |

표 25: conntrack 상황 (ct_status)

| 키워드     | 값  |
| ---------- | --- |
| expected   | 1   |
| seen-reply | 2   |
| assured    | 4   |
| confirmed  | 8   |
| snat       | 16  |
| dnat       | 32  |
| dying      | 512 |

표 26: conntrack 이벤트 비트 (ct_event)

| 키워드    | 값   |
| --------- | ---- |
| new       | 1    |
| related   | 2    |
| destroy   | 4    |
| reply     | 8    |
| assured   | 16   |
| protoinfo | 32   |
| helper    | 64   |
| mark      | 128  |
| seqadj    | 256  |
| secmark   | 512  |
| label     | 1024 |

conntrack 레이블 타입(ct_label)에 가능한 키워드들은 런타임에 `/etc/connlabel.conf`에서 읽어 들인다.

## 기본 식

가장 하위의 식이 기본 식이며 상수, 또는 패킷 페이로드나 메타 데이터, 상태 모듈에서 온 데이터 하나를 나타낸다.

### meta 식

<pre>
<strong>meta</strong> {<strong>length</strong> | <strong>nfproto</strong> | <strong>l4proto</strong> | <strong>protocol</strong> | <strong>priority</strong>}
[<strong>meta</strong>] {<strong>mark</strong> | <strong>iif</strong> | <strong>iifname</strong> | <strong>iiftype</strong> | <strong>oif</strong> | <strong>oifname</strong> | <strong>oiftype</strong> | <strong>skuid</strong> | <strong>skgid</strong> | <strong>nftrace</strong> | <strong>rtclassid</strong> | <strong>ibrname</strong> | <strong>obrname</strong> | <strong>pkttype</strong> | <strong>cpu</strong> | <strong>iifgroup</strong> | <strong>oifgroup</strong> | <strong>cgroup</strong> | <strong>random</strong> | <strong>ipsec</strong> | <strong>iifkind</strong> | <strong>oifkind</strong> | <strong>time</strong> | <strong>hour</strong> | <strong>day</strong>}
</pre>

meta 식은 패킷과 연관된 메타 데이터를 나타내는 식이다.

meta 식에는 지정 meta 식과 비지정 meta 식 두 종류가 있다. 지정 meta 식에선 메타 키 앞에 meta 키워드가 필요하고 비지정 meta 식은 메타 키를 바로 쓰거나 지정 meta 식으로 지정할 수 있다. meta l4proto는 IPv4나 IPv6 패킷에 포함된 특정 전송 프로토콜을 맞춰 보는 데 유용하다. IPv6 패킷에 IPv6 확장 헤더가 있으면 그 역시 건너뛰게 된다.

meta iif, oif, iifname, oifname는 패킷이 도착한 인터페이스와 나갈 인터페이스를 맞춰 보는 데 쓴다.

iif와 oif는 인터페이스 번호로 맞춰 보는 반면 iifname과 oifname은 인터페이스 이름으로 맞춰 본다. 이 둘은 같지 않다. 가령 다음 규칙을 생각해 보면,

```text
filter input meta iif "foo"
```

인터페이스 "foo"가 존재하는 경우에만 이 규칙을 추가할 수 있다. 또한 그 규칙은 인터페이스 "foo"의 이름이 "bar"로 바뀐 경우에도 계속 일치하게 된다.

그렇게 되는 이유는 내부적으로 인터페이스 번호를 쓰기 때문이다. tun/tap이나 다이얼업 인터페이스(예를 들어 ppp)처럼 동적으로 생성되는 인터페이스인 경우 iifname과 oifname을 쓰는 게 더 나을 수 있다.

그런 경우에 이름을 쓰면 규칙을 추가하기 위해 인터페이스가 꼭 존재할 필요가 없고, 인터페이스 이름이 바뀌면 일치하지 않게 됐다가 인터페이스가 삭제되고 같은 이름의 새 인터페이스가 생기면 다시 일치하게 된다.

표 27: meta 식 타입

| 키워드    | 설명 | 타입 |
| --------- | ---- | ---- |
| length    | 바이트 단위 패킷 길이 | integer (32비트) |
| nfproto   | 실제 훅 프로토콜 패밀리, inet 테이블에서만 유용 | integer (32비트) |
| l4proto   | 제4계층 프로토콜, ipv6 확장 헤더 건너뜀 | integer (8비트) |
| protocol  | EtherType 프로토콜 값 | ether_type |
| priority  | TC 패킷 우선순위 | tc_handle |
| mark      | 패킷 마크 | mark |
| iif       | 입력 인터페이스 번호 | iface_index |
| iifname   | 입력 인터페이스 이름 | ifname |
| iiftype   | 입력 인터페이스 타입 | iface_type |
| oif       | 출력 인터페이스 번호 | iface_index |
| oifname   | 출력 인터페이스 이름 | ifname |
| oiftype   | 출력 인터페이스 하드웨어 타입 | iface_type |
| skuid     | 발신 소켓에 연계된 UID | uid |
| skgid     | 발신 소켓에 연계된 GID | gid |
| rtclassid | 라우팅 realm | realm |
| ibrname   | 입력 브리지 인터페이스 이름 | ifname |
| obrname   | 출력 브리지 인터페이스 이름 | ifname |
| pkttype   | 패킷 타입 | pkt_type |
| cpu       | 패킷 처리 중인 cpu 번호 | integer (32비트) |
| iifgroup  | 입력 장치 그룹 | devgroup |
| oifgroup  | 출력 장치 그룹 | devgroup |
| cgroup    | 제어 그룹 ID | integer (32비트) |
| random    | 유사 난수 | integer (32비트) |
| ipsec     | 불리언 | boolean (1비트) |
| iifkind   | 입력 인터페이스 종류 | |
| oifkind   | 출력 인터페이스 종류 | |
| time      | 패킷을 수신한 절대 시간 | integer (32비트) 또는 string |
| day       | 주 중 요일 | integer (8비트) 또는 string |
| hour      | 하루 중 시간 | string |

표 28: meta 식 한정 타입

| 타입          | 설명 |
| ------------- | ---- |
| iface_index   | 인터페이스 번호 (32비트 수). 숫자로 또는 기존 인터페이스의 이름으로 지정 가능. |
| ifname        | 인터페이스 이름 (16바이트 문자열). 존재하지 않아도 됨. |
| iface_type    | 인터페이스 타입 (16비트 수). |
| uid           | 사용자 ID (32비트 수). 숫자로 또는 사용자 이름으로 지정 가능. |
| gid           | 그룹 ID (32비트 수). 숫자로 또는 그룹 이름으로 지정 가능. |
| realm         | 라우팅 realm (32비트 수). 숫자로 또는 /etc/iproute2/rt_realms에 정의된 심볼 이름으로 지정 가능. |
| devgroup_type | 장치 그룹 (32비트 수). 숫자로 또는 /etc/iproute2/group에 정의된 심볼 이름으로 지정 가능. |
| pkt_type      | 패킷 종류: `host` (로컬 호스트 향함), `broadcast` (모두에게), `multicast` (그룹에게), `other` (다른 호스트 향함). |
| ifkind        | 인터페이스 종류 (16바이트 문자열). 존재하지 않아도 됨. |
| time          | 정수 또는 ISO 형식 날짜. 예를 들어 "2019-06-06 17:00". 시간과 초는 선택적이며 원하는 생략 가능. 생략 시 자정을 상정함. 즉 "2019-06-06", "2019-06-06 00:00", "2019-06-06 00:00:00"은 동등함. 정수를 주는 경우 유닉스 타임스탬프라고 상정함. |
| day           | 주 중 요일("Monday", "Tuesday", 등) 또는 0에서 6 사이 정수. 문자열 일치 여부에 대소문자를 구별하지 않으며 완전히 일치할 필요 없음. (가령 "Mon"이라고 하면 "Monday"에 일치함.) 정수를 주는 경우 0이 일요일이고 6이 토요일임. |
| hour          | 24시간 형식으로 시간을 나타내는 문자열. 초를 선택적으로 지정할 수 있음. 예를 들어 17:00과 17:00:00이 동등함. |

#### meta 식 사용하기

```text
# 지정 meta 식
filter output meta oif eth0

# 비지정 meta 식
filter output oif eth0

# 패킷이 ipsec 처리 대상이었음
raw prerouting meta ipsec exists accept
```

### socket 식

<pre>
<strong>socket</strong> {<strong>transparent</strong> | <strong>mark</strong>}
</pre>

socket 식을 사용해 기존의 열린 TCP/UDP 소켓이나 패킷에 연계될 수 있는 소켓 속성을 탐색할 수 있다. 수립 상태이거나 0 아닌 주소에 (가능하면 로컬 아닌 주소에) 결속된 리스닝 소켓을 찾는다.

표 29: 사용 가능한 소켓 속성

| 이름        | 설명 | 타입 |
| ----------- | ---- | ---- |
| transparent | 찾은 소켓의 IP_TRANSPARENT 소켓 옵션 값. 0 또는 1일 수 있음. | boolean (1비트) |
| mark        | 소켓 마크(SOL_SOCKET, SO_MARK) 값. | mark |

#### 소켓 식 사용하기

```text
# 투명 소켓에 대응하는 패킷에 표시
table inet x {
    chain y {
        type filter hook prerouting priority -150; policy accept;
        socket transparent 1 mark set 0x00000001 accept
    }
}

# mark 값이 15인 소켓에 대응하는 패킷 추적
table inet x {
    chain y {
        type filter hook prerouting priority -150; policy accept;
        socket mark 0x0000000f nftrace set 1
    }
}

# 패킷 mark를 소켓 mark로 설정
table inet x {
    chain y {
        type filter hook prerouting priority -150; policy accept;
        tcp dport 8080 mark set socket mark
    }
}
```

### osf 식

<pre>
<strong>osf</strong> [<strong>ttl</strong> {<strong>loose</strong> | <strong>skip</strong>}] {<strong>name</strong> | <strong>version</strong>}
</pre>

osf 식은 수동적 운영 체제 감식을 한다. 이 식은 SYN 비트가 설정된 패킷에서 가져온 몇 가지 데이터(윈도 크기, MSS, 옵션 및 순서, DF 등)를 비교한다.

표 30: 사용 가능한 osf 속성

| 이름    | 설명 | 타입 |
| ------- | ---- | ---- |
| ttl     | 운영 체제를 판단하기 위해 패킷의 TTL 검사를 하기. | string |
| version | 패킷에서 OS 버전 검사 하기. | |
| name    | 맞춰 볼 OS 시그너처 이름. pf.os 파일에 전체 시그너처들이 있음. 식에서 탐지할 수 없었던 OS 시그너처엔 "unknown" 사용. | string |

#### 사용 가능한 ttl 값

TTL 속성을 주지 않으면 IP 헤더의 값과 핑거프린트 TTL 값이 같은지 비교한다. 일반적으로 LAN에서 잘 동작한다.

* loose: IP 헤더의 TTL이 핑거프린트 값보다 작은지 검사한다. 전역 라우팅 가능 주소에 잘 동작한다.
* skip: TTL을 아예 비교하지 않는다.

#### osf 식 사용하기

```text
# TTL 비교 없이 "Linux" OS 계열 시그너처에 일치하는 패킷 허용하기
table inet x {
    chain y {
        type filter hook input priority 0; policy accept;
        osf ttl skip name "Linux"
    }
}
```

### fib 식

<pre>
<strong>fib</strong> {<strong>saddr</strong> | <strong>daddr</strong> | <strong>mark</strong> | <strong>iif</strong> | <strong>oif</strong>} [<strong>.</strong> ...] {<strong>oif</strong> | <strong>oifname</strong> | <strong>type</strong>}
</pre>

fib 식은 fib(forwarding information base)를 조회해서 특정 주소가 사용하게 될 출력 인터페이스 번호 같은 정보를 얻는다. 입력은 fib 검색 함수 입력으로 쓸 요소들의 튜플이다.

표 31: fib 식 한정 타입

| 키워드  | 설명 | 타입 |
| ------- | ---- | ---- |
| oif     | 출력 인터페이스 번호 | integer (32비트) |
| oifname | 출력 인터페이스 이름 | string |
| type    | 주소 타입 | fib_addrtype |

모든 주소 타입들의 목록을 보려면 `nft describe fib_addrtype`.

#### fib 식 사용하기

```text
# 역경로 없는 패킷 버리기
filter prerouting fib saddr . iif oif missing drop
```

이 예에서 `saddr . iif`는 출발 주소와 입력 인터페이스를 가지고 라우팅 정보를 검색한다. oif는 그 라우팅 정보에서 출력 인터페이스 번호를 뽑아낸다. 그 출발 주소/입력 인터페이스 조합에 대한 라우트를 찾지 못했으면 출력 인터페이스 번호가 0이다. 입력 키 중 일부로 입력 인터페이스를 지정한 경우 출력 인터페이스 번호는 언제나 입력 인터페이스 번호와 같거나 0이다. `saddr oif`만 준 경우에는 oif가 아무 인터페이스 번호 또는 0일 수 있다.

```text
# 인터페이스에 설정 안 된 주소를 향한 패킷 버리기
filter prerouting fib daddr . iif type != { local, broadcast, multicast } drop

# 특정 '블랙홀' 테이블(0xdead, 적절한 ip rule 필요)에서 검색 수행하기
filter prerouting meta mark set 0xdead fib daddr . mark type vmap { blackhole : drop, prohibit : jump prohibited, unreachable : drop }
```

### 라우팅 식

<pre>
<strong>rt</strong> [<strong>ip</strong> | <strong>ip6</strong>] {<strong>classid</strong> | <strong>nexthop</strong> | <strong>mtu</strong> | <strong>ipsec</strong>}
</pre>

라우팅 식은 패킷에 연계된 라우팅 데이터를 가리킨다.

표 32: 라우팅 식 타입

| 키워드  | 설명 | 타입 |
| ------- | ---- | ---- |
| classid | 라우팅 realm | realm |
| nexthop | 라우팅 nexthop | ipv4_addr/ipv6_addr |
| mtu     | 라우트의 TCP 최대 세그먼트 크기 | integer (16비트) |
| ipsec   | ipsec 터널 또는 트랜스포트를 통한 라우트 | boolean |

표 33: 라우팅 식 한정 타입

| 타입  | 설명 |
| ----- | ---- |
| realm | 라우팅 realm (32비트 수). 숫자로 또는 /etc/iproute2/rt_realms에 정의된 심볼 이름으로 지정 가능. |

#### 라우팅 식 사용하기

```text
# IP 패밀리와 무관한 rt 식
filter output rt classid 10
filter output rt ipsec missing

# IP 패밀리에 의존적인 rt 식
ip filter output rt nexthop 192.168.0.1
ip6 filter output rt nexthop fd00::1
inet filter output rt ip nexthop 192.168.0.1
inet filter output rt ip6 nexthop fd00::1
```

### ipsec 식

<pre>
<strong>ipsec</strong> {<strong>in</strong> | <strong>out</strong>} [ <strong>spnum</strong> <em>NUM</em> ] {<strong>reqid</strong> | <strong>spi</strong>}
<strong>ipsec</strong> {<strong>in</strong> | <strong>out</strong>} [ <strong>spnum</strong> <em>NUM</em> ] {<strong>ip</strong> | <strong>ip6</strong>} {<strong>saddr</strong> | <strong>daddr</strong>}
</pre>

ipsec 식은 패킷에 연계된 ipsec 데이터를 가리킨다.

식에서 입력 또는 출력 방향 정책을 검사해야 하는 경우 `in` 또는 `out` 키워드를 써서 방향을 지정해야 한다. `in` 키워드는 prerouting, input, forward 훅에서 쓸 수 있다. `out` 키워드는 forward, output, postrouting 훅에 해당한다. 선택적인 spnum 키워드를 써서 체인 내의 특정 상태에 맞춰 볼 수 있으며 기본은 0이다.

표 34: ipsec 식 타입

| 키워드 | 설명 | 타입 |
| ------ | ---- | ---- |
| reqid  | 요청 ID | integer (32비트) |
| spi    | 보안 매개변수 색인 | integer (32비트) |
| saddr  | 터널의 출발 주소 | ipv4_addr/ipv6_addr |
| daddr  | 터널의 목적 주소 | ipv4_addr/ipv6_addr |

### numgen 식

<pre>
<strong>numgen</strong> {<strong>inc</strong> | <strong>random</strong>} <strong>mod</strong> <em>NUM</em> [ <strong>offset</strong> <em>NUM</em> ]
</pre>

수 생성기를 만든다. `inc` 및 `random` 키워드가 동작 방식을 결정한다. `inc` 방식에서는 마지막 반환 값을 증가시킬 뿐이다. `random` 방식에선 새 난수를 반환한다. `mod` 키워드 뒤의 값은 반환되는 수가 도달할 수 없는 상한을 (모듈로 연산) 지정한다. 선택적인 `offset`를 통해 반환 값을 고정된 간격만큼 증가시킬 수 있다.

`numgen`의 일반적인 용도는 부하 분산이다.

#### numgen 식 사용하기

```text
# 192.168.10.100과 192.168.20.200 중 하나로 라운드 로빈:
add rule nat prerouting dnat to numgen inc mod 2 map \
        { 0 : 192.168.10.100, 1 : 192.168.20.200 }

# 구간을 이용해 불균일하게 확률 기반 분산:
add rule nat prerouting dnat to numgen random mod 10 map \
        { 0-2 : 192.168.10.100, 3-9 : 192.168.20.200 }
```

## 페이로드 식

페이로드 식은 패킷 페이코드에서 온 데이터를 가리킨다.

### 이더넷 헤더 식

<pre>
<strong>ether</strong> {<strong>daddr</strong> | <strong>saddr</strong> | <strong>type</strong>}
</pre>

표 35: 이더넷 헤더 식 타입

| 키워드 | 설명 | 타입 |
| ------ | ---- | ---- |
| daddr  | 목적 MAC 주소 | ether_addr |
| saddr  | 출발 MAC 주소 | ether_addr |
| type   | EtherType     | ether_type |

### VLAN 헤더 식

<pre>
<strong>vlan</strong> {<strong>id</strong> | <strong>cfi</strong> | <strong>pcp</strong> | <strong>type</strong>}
</pre>

표 36: VLAN 헤더 식

| 키워드 | 설명 | 타입 |
| ------ | ---- | ---- |
| id     | VLAN ID (VID) | integer (12비트) |
| cfi    | Canonical Format Indicator | integer (1비트) |
| pcp    | Priority Code Point | integer (3비트) |
| type   | EtherType | ether_type |

### ARP 헤더 식

<pre>
<strong>arp</strong> {<strong>htype</strong> | <strong>ptype</strong> | <strong>hlen</strong> | <strong>plen</strong> | <strong>operation</strong> | <strong>saddr</strong> { <strong>ip</strong> | <strong>ether</strong> } | <strong>daddr</strong> { <strong>ip</strong> | <strong>ether</strong> }}
</pre>

표 37: ARP 헤더 식

| 키워드      | 설명 | 타입 |
| ----------- | ---- | ---- |
| htype       | ARP 하드웨어 타입 | integer (16비트) |
| ptype       | EtherType | ether_type |
| hlen        | 하드웨어 주소 길이 | integer (8비트) |
| plen        | 프로토콜 주소 길이 | integer (8비트) |
| operation   | 동작 | arp_op |
| saddr ether | 이더넷 송신자 주소 | ether_addr |
| daddr ether | 이더넷 대상 주소 | ether_addr |
| saddr ip    | IPv4 송신자 주소 | ipv4_addr |
| daddr ip    | IPv4 대상 주소 | ipv4_addr |

### IPv4 헤더 식

<pre>
<strong>ip</strong> {<strong>version</strong> | <strong>hdrlength</strong> | <strong>dscp</strong> | <strong>ecn</strong> | <strong>length</strong> | <strong>id</strong> | <strong>frag-off</strong> | <strong>ttl</strong> | <strong>protocol</strong> | <strong>checksum</strong> | <strong>saddr</strong> | <strong>daddr</strong> }
</pre>

표 38: IPv4 헤더 식

| 키워드    | 설명 | 타입 |
| --------- | ---- | ---- |
| version   | IP 헤더 버전 (4) | integer (4비트) |
| hdrlength | 옵션 포함 IP 헤더 길이 | integer (4비트) FIXME 단위 |
| dscp      | Differentiated Services Code Point | dscp |
| ecn       | Explicit Congestion Notification | ecn |
| length    | 패킷 총 길이 | integer (16비트) |
| id        | IP ID | integer (16비트) |
| frag-off  | 단편 오프셋 | integer (16비트) |
| ttl       | Time to live | integer (8비트) |
| protocol  | 상위 계층 프로토콜 | inet_proto |
| checksum  | IP 헤더 체크섬 | integer (16비트) |
| saddr     | 출발 주소 | ipv4_addr |
| daddr     | 목적 주소 | ipv4_addr |

### ICMP 헤더 식

<pre>
<strong>icmp</strong> {<strong>type</strong> | <strong>code</strong> | <strong>checksum</strong> | <strong>id</strong> | <strong>sequence</strong> | <strong>gateway</strong> | <strong>mtu</strong>}
</pre>

표 39: ICMP 헤더 식

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| type     | ICMP type 필드 | icmp_type |
| code     | ICMP code 필드 | integer (8비트) |
| checksum | ICMP checksum 필드 | integer (16비트) |
| id       | echo request/response의 ID | integer (16비트) |
| sequence | echo request/response의 일련 번호 | integer (16비트) |
| gateway  | redirect의 게이트웨이 | integer (32비트) |
| mtu      | 경로 MTU 탐색의 MTU | integer (16비트) |

### IGMP 헤더 식

<pre>
<strong>igmp</strong> {<strong>type</strong> | <strong>mrt</strong> | <strong>checksum</strong> | <strong>group</strong>}
</pre>

이 식은 IGMP 헤더 필드들을 가리킨다. `inet`, `bridge`, `netdev` 패밀리에서 쓸 때는 IPv4에 대한 암묵적 의존성이 생기게 된다. IPv6 상의 IGMP 같은 특이한 경우에 일치하게 하려면 규칙에 따로 `meta protocol ip6`를 추가해 줘야 한다.

표 40: ICMP 헤더 식

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| type     | IGMP type 필드 | igmp_type |
| mrt      | IGMP maximum response time 필드 | integer (8비트) |
| checksum | IGMP checksum 필드 | integer (16비트) |
| group    | 그룹 주소 | integer (32비트) |

### IPv6 헤더 식

<pre>
<strong>ip6</strong> {<strong>version</strong> | <strong>dscp</strong> | <strong>ecn</strong> | <strong>flowlabel</strong> | <strong>length</strong> | <strong>nexthdr</strong> | <strong>hoplimit</strong> | <strong>saddr</strong> | <strong>daddr</strong>}
</pre>

이 식은 IPv6 헤더 필드들을 가리킨다. `ip6 nexthdr` 사용 시 조심해야 한다. 그 값은 다음 헤더를 가리킬 뿐이다. 즉 `ip6 nexthdr tcp`는 IPv6 패킷에 확장 헤더가 하나도 없는 경우에만 걸린다. 단편화 돼 있거나 가령 라우팅 확장 헤더를 담고 있는 패킷은 걸리지 않게 된다. 실제 전송 헤더를 확인하고 싶고 확장 헤더는 무시하고 싶다면 `meta l4proto`를 써 달라.

표 41: IPv6 헤더 식

| 키워드    | 설명 | 타입 |
| --------- | ---- | ---- |
| version   | IP 헤더 버전 (6) | integer (4비트) |
| dscp      | Differentiated Services Code Point | dscp |
| ecn       | Explicit Congestion Notification | ecn |
| flowlabel | Flow label | integer (20비트) |
| length    | 페이로드 길이 | integer (16비트) |
| nexthdr   | nexthdr 프로토콜 | inet_proto |
| hoplimit  | Hop limit | integer (8비트) |
| saddr     | 출발 주소 | ipv6_addr |
| daddr     | 목적 주소 | ipv6_addr |

#### ip6 헤더 식 사용하기

```text
# 첫 번째 확장 헤더가 단편을 나타내면 일치
ip6 nexthdr ipv6-frag
```

### ICMPv6 헤더 식

<pre>
<strong>icmpv6</strong> {<strong>type</strong> | <strong>code</strong> | <strong>checksum</strong> | <strong>parameter-problem</strong> | <strong>packet-too-big</strong> | <strong>id</strong> | <strong>sequence</strong> | <strong>max-delay</strong>}
</pre>

이 식은 ICMPv6 헤더 필드들을 가리킨다. `inet`, `bridge`, `netdev` 패밀리에서 쓸 때는 IPv6에 대한 암묵적 의존성이 생기게 된다. IPv4 상의 ICMPv6 같은 특이한 경우에 일치하게 하려면 규칙에 따로 `meta protocol ip`를 추가해 줘야 한다.

표 42: ICMPv6 헤더 식

| 키워드            | 설명 | 타입 |
| ----------------- | ---- | ---- |
| type              | ICMPv6 type 필드 | icmpv6_type |
| code              | ICMPv6 code 필드 | integer (8비트) |
| checksum          | ICMPv6 checksum 필드 | integer (16비트) |
| parameter-problem | 문제 포인터 | integer (32비트) |
| packet-too-big    | 초과한 MTU | integer (32비트) |
| id                | echo request/response의 ID | integer (16비트) |
| sequence          | echo request/response의 일련 번호 | integer (16비트) |
| max-delay         | MLD 질의 응답 최대 지연 | integer (16비트) |

### TCP 헤더 식

<pre>
<strong>tcp</strong> {<strong>sport</strong> | <strong>dport</strong> | <strong>sequence</strong> | <strong>ackseq</strong> | <strong>doff</strong> | <strong>reserved</strong> | <strong>flags</strong> | <strong>window</strong> | <strong>checksum</strong> | <strong>urgptr</strong>}
</pre>

표 43: TCP 헤더 식

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| sport    | 출발 포트 | inet_service |
| dport    | 목적 포트 | inet_service |
| sequence | 일련 번호 | integer (32비트) |
| ackseq   | 확인 번호 | integer (32비트) |
| doff     | 데이터 오프셋 | integer (4비트) FIXME 단위 |
| reserved | 예비 영역 | integer (4비트) |
| flags    | TCP 플래그 | tcp_flag |
| window   | 윈도 | integer (16비트) |
| checksum | 체크섬 | integer (16비트) |
| urgptr   | 긴급 포인터 | integer (16비트) |

### UDP 헤더 식

<pre>
<strong>udp</strong> {<strong>sport</strong> | <strong>dport</strong> | <strong>length</strong> | <strong>checksum</strong>}
</pre>

표 44: UDP 헤더 식

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| sport    | 출발 포트 | inet_service |
| dport    | 목적 포트 | inet_service |
| length   | 패킷 총 길이 | integer (16비트) |
| checksum | 체크섬 | integer (16비트) |

### UDP-Lite 헤더 식

<pre>
<strong>udplite</strong> {<strong>sport</strong> | <strong>dport</strong> | <strong>checksum</strong>}
</pre>

표 45: UDP-Lite 헤더 식

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| sport    | 출발 포트 | inet_service |
| dport    | 목적 포트 | inet_service |
| checksum | 체크섬 | integer (16비트) |

### SCTP 헤더 식

<pre>
<strong>sctp</strong> {<strong>sport</strong> | <strong>dport</strong> | <strong>vtag</strong> | <strong>checksum</strong>}
</pre>

표 46: SCTP 헤더 식

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| sport    | 출발 포트 | inet_service |
| dport    | 목적 포트 | inet_service |
| vtag     | 검증 태그 | integer (32비트) |
| checksum | 체크섬 | integer (32비트) |

### DCCP 헤더 식

<pre>
<strong>dccp</strong> {<strong>sport</strong> | <strong>dport</strong>}
</pre>

표 47: DCCP 헤더 식

| 키워드 | 설명 | 타입 |
| ------ | ---- | ---- |
| sport  | 출발 포트 | inet_service |
| dport  | 목적 포트 | inet_service |

### 인증 헤더 식

<pre>
<strong>ah</strong> {<strong>nexthdr</strong> | <strong>hdrlength</strong> | <strong>reserved</strong> | <strong>spi</strong> | <strong>sequence</strong>}
</pre>

표 48: AH 헤더 식

| 키워드    | 설명 | 타입 |
| --------- | ---- | ---- |
| nexthdr   | 다음 헤더 프로토콜 | inet_proto |
| hdrlength | AH 헤더 길이 | integer (8비트) |
| reserved  | 예비 영역 | integer (16비트) |
| spi       | 보안 매개변수 색인 | integer (32비트) |
| sequence  | 일련 번호 | integer (32비트) |

### 보안 페이로드 캡슐화 헤더 식

<pre>
<strong>esp</strong> {<strong>spi</strong> | <strong>sequence</strong>}
</pre>

표 49: ESP 헤더 식

| 키워드   | 설명 | 타입 |
| -------- | ---- | ---- |
| spi      | 보안 매개변수 색인 | integer (32비트) |
| sequence | 일련 번호 | integer (32비트) |

### IPCOMP 헤더 식

<pre>
<strong>comp</strong> {<strong>nexthdr</strong> | <strong>flags</strong> | <strong>cpi</strong>}
</pre>

표 50: IPComp 헤더 식

| 키워드  | 설명 | 타입 |
| ------- | ---- | ---- |
| nexthdr | 다음 헤더 프로토콜 | inet_proto |
| flags   | 플래그 | bitmask |
| cpi     | 압축 매개변수 색인 | integer (16비트) |

### 비가공 페이로드 식

<pre>
<strong>@</strong><em>base</em><strong>,</strong><em>offset</em><strong>,</strong><em>length</em>
</pre>

비가공 페이로드 식은 *offset* 번째 비트부터 *length* 개 비트를 읽어 온다. 0번째 비트는 제일 첫 비트를 가리킨다. C 프로그래밍 언어로는 최상위 비트, 즉 옥텟이라면 0x80에 해당한다. 아직 사람이 읽을 수 있는 템플릿 식이 없는 헤더에 맞춰 보는 데 유용하다. 참고로 nft에서 비가공 페이로드 식에 자동으로 의존 조건을 추가해 주지 않는다. 가령 프로토콜 번호 5인 전송 헤더의 프로토콜 필드에 맞춰 보고 싶다면 그 비가공 식 앞에 `meta l4proto 5`처럼 써서 다른 전송 헤더의 패킷들을 직접 제외해 줘야 한다.

표 51: 지원하는 페이로드 프로토콜 base

| base | 설명 |
| ---- | ---- |
| ll   | 링크 계층. 예를 들어 이더넷 헤더 |
| nh   | 네트워크 헤더. 예를 들어 IPv4나 IPv6 |
| th   | 전송 헤더. 예를 들어 TCP |

#### UDP와 TCP 모두의 목적 포트 확인하기

```text
inet filter input meta l4proto {tcp, udp} @th,16,16 { 53, 80 }
```

위를 다음처럼 쓸 수도 있다.

```text
inet filter input meta l4proto {tcp,udp} th dport { 53, 80 }
```

더 편리하긴 하지만 비가공 식 표기와 마찬가지로 어떤 의존 조건도 만들거나 확인하지 않는다. 포트 개념이 있는 헤더 종류들로만 검사를 한정하는 건 사용자의 책임이다. 그렇게 해 주지 않으면 가령 ESP 패킷의 SPI 필드를 포트로 잘못 해석해서 식과 무관한 패킷이 잘못 걸리게 된다.

#### ARP 패킷 목적 프로토콜 주소가 지정 주소와 일치하면 대상 하드웨어 주소 다시 쓰기

```text
input meta iifname enp2s0 arp ptype 0x0800 arp htype 1 arp hlen 6 arp plen 4 @nh,192,32 0xc0a88f10 @nh,144,48 set 0x112233445566 accept
```

### 확장 헤더 식

확장 헤더 식은 IPv6 확장 헤더, TCP 옵션, IPv4 옵션 같은 가변 크기 프로토콜 헤더의 데이터를 가리킨다.

nftables에서는 현재 IPv6 확장 헤더, TCP 옵션, IPv4 옵션 검사(찾기)를 지원한다.

<pre>
<strong>hbh</strong> {<strong>nexthdr</strong> | <strong>hdrlength</strong>}
<strong>frag</strong> {<strong>nexthdr</strong> | <strong>frag-off</strong> | <strong>more-fragments</strong> | <strong>id</strong>}
<strong>rt</strong> {<strong>nexthdr</strong> | <strong>hdrlength</strong> | <strong>type</strong> | <strong>seg-left</strong>}
<strong>dst</strong> {<strong>nexthdr</strong> | <strong>hdrlength</strong>}
<strong>mh</strong> {<strong>nexthdr</strong> | <strong>hdrlength</strong> | <strong>checksum</strong> | <strong>type</strong>}
<strong>srh</strong> {<strong>flags</strong> | <strong>tag</strong> | <strong>sid</strong> | <strong>seg-left</strong>}
<strong>tcp option</strong> {<strong>eol</strong> | <strong>noop</strong> | <strong>maxseg</strong> | <strong>window</strong> | <strong>sack-permitted</strong> | <strong>sack</strong> | <strong>sack0</strong> | <strong>sack1</strong> | <strong>sack2</strong> | <strong>sack3</strong> | <strong>timestamp</strong>} <em>tcp_option_field</em>
<strong>ip option</strong> {<strong>lsrr</strong> | <strong>ra</strong> | <strong>rr</strong> | <strong>ssrr</strong>} <em>ip_option_field</em>
</pre>

다음 문법은 식 오른편이 헤더 존재 여부만 확인하는 불리언 타입인 관계 식에서만 유효하다.

<pre>
<strong>exthdr</strong> {<strong>hbh</strong> | <strong>frag</strong> | <strong>rt</strong> | <strong>dst</strong> | <strong>mh</strong>}
<strong>tcp option</strong> {<strong>eol</strong> | <strong>noop</strong> | <strong>maxseg</strong> | <strong>window</strong> | <strong>sack-permitted</strong> | <strong>sack</strong> | <strong>sack0</strong> | <strong>sack1</strong> | <strong>sack2</strong> | <strong>sack3</strong> | <strong>timestamp</strong>}
<strong>ip option</strong> {<strong>lsrr</strong> | <strong>ra</strong> | <strong>rr</strong> | <strong>ssrr</strong>}
</pre>

표 52: IPv6 확장 헤더

| 키워드 | 설명 |
| ------ | ---- |
| hbh    | Hop by Hop |
| rt     | Routing Header |
| frag   | Fragmentation Header |
| dst    | dst 옵션 |
| mh     | Mobility Header |
| srh    | Segment Routing Header |

표 53: TCP 옵션

| 키워드         | 설명 | TCP 옵션 필드 |
| -------------- | ---- | ------------- |
| eol            | 옵션 목록 끝 | kind |
| noop           | 1 바이트 TCP no-op 옵션 | kind |
| maxseg         | TCP 세그먼트 최대 크기 | kind, length, size |
| window         | TCP 윈도 스케일링 | kind, length, count |
| sack-permitted | TCP SACK 허용 | kind, length |
| sack           | TCP 선택적 확인 (0번 블록 별칭) | kind, length, left, right |
| sack0          | TCP 선택적 확인 (0번 블록) | kind, length, left, right |
| sack1          | TCP 선택적 확인 (1번 블록) | kind, length, left, right |
| sack2          | TCP 선택적 확인 (2번 블록) | kind, length, left, right |
| sack3          | TCP 선택적 확인 (3번 블록) | kind, length, left, right |
| timestamp      | TCP 타임스탬프 | kind, elngth, tsval, tsecr |

표 54: IP 옵션

| 키워드 | 설명 | IP 옵션 필드 |
| ------ | ---- | ------------ |
| lsrr   | Loose Source Route | type, length, ptr, addr |
| ra     | Router Alert | type, length, value |
| rr     | Record Route | type, length, ptr, addr |
| ssrr   | Strict Source Route | type, length, ptr, addr |

#### TCP 옵션 찾기

```text
filter input tcp option sack-permitted kind 1 counter
```

#### IPv6 exthdr 확인하기

```text
ip6 filter input frag more-fragments 1 counter
```

#### IP 옵션 찾기

```text
filter input ip option lsrr exists counter
```

### conntrack 식

conntrack 식은 패킷과 연계된 연결 추적 항목의 메타 데이터를 가리킨다.

세 가지 conntrack 식이 있다. 어떤 conntrack 식에선 conntrack 키 앞에 흐름 방향이 필요하지만 다른 식은 방향과 무관할 수 있기 때문에 바로 쓸 수도 있다. `packets`, `bytes`, `avgpkt` 키워드는 방향과 함께 쓸 수도 있고 없이 쓸 수도 있다. 방향을 생략하면 original 방향과 reply 방향의 합을 내놓는다. `zone`도 마찬가진데, 방향을 주면 그 존 ID가 해당 방향에 결속돼 있는 경우에만 존이 일치한다.

<pre>
<strong>ct</strong> {<strong>state</strong> | <strong>direction</strong> | <strong>status</strong> | <strong>mark</strong> | <strong>expiration</strong> | <strong>helper</strong> | <strong>label</strong>}
<strong>ct</strong> [<strong>original</strong> | <strong>reply</strong>] {<strong>l3proto</strong> | <strong>protocol</strong> | <strong>bytes</strong> | <strong>packets</strong> | <strong>avgpkt</strong> | <strong>zone</strong>}
<strong>ct</strong> {<strong>original</strong> | <strong>reply</strong>} {<strong>proto-src</strong> | <strong>proto-dst</strong>}
<strong>ct</strong> {<strong>original</strong> | <strong>reply</strong>} {<strong>ip</strong> | <strong>ip6</strong>} {<strong>saddr</strong> | <strong>daddr</strong>}
</pre>

표 55: conntrack 식

| 키워드     | 설명 | 타입 |
| ---------- | ---- | ---- |
| state      | 연결의 상태 | ct_state |
| direction  | 연결 기준 패킷 방향 | ct_dir |
| status     | 연결의 상황 | ct_status |
| mark       | 연결 마크 | mark |
| expiration | 연결 만료 시간 | time |
| helper     | 연결에 연계된 헬퍼 | string |
| label      | 연결 추적 레이블 비트 또는 nftables include 경로의 connlabel.conf에 정의된 심볼 이름 | ct_label |
| l3proto    | 연결의 제3계층 프로토콜 | nf_proto |
| saddr      | 해당 방향의 연결의 출발 주소 | ipv4_addr/ipv6_addr |
| daddr      | 해당 방향의 연결의 목적 주소 | ipv4_addr/ipv6_addr |
| protocol   | 해당 방향의 연결의 제4계층 프로토콜 | inet_proto |
| proto-src  | 해당 방향의 제4계층 프로토콜 출발 주소 | integer (16비트) |
| proto-dst  | 해당 방향의 제4계층 프로토콜 목적 주소 | integer (16비트) |
| packets    | 해당 방향 또는 original과 reply 모두에서 지나간 패킷 수 | integer (64비트) |
| bytes      | 지나간 바이트 수. `packets` 키워드 설명 참고 | integer (64비트) |
| avgpkt     | 패킷당 평균 바이트. `packets` 키워드 설명 참고 | integer (64비트) |
| zone       | conntrack 존 | integer (16비트) |

위에 나열된 conntrack 한정 타입들에 대한 설명을 위의 conntrack 타입 절에서 볼 수 있다.

#### 서버로 동시에 향하는 연결 수 제한하기

```text
filter input tcp dport 22 meter test { ip saddr ct count over 2 } reject
```

## 문

### 판정 문

### 페이로드 문

### 확장 헤더 문

### 로그 문

### 거절 문

### 카운터 문

### conntrack 문

### meta 문

### 제한 문

### NAT 문

### TPROXY 문

### SYNPROXY 문

### flow 문

### queue 문

### dup 문

### fwd 문

### set 문

### map 문

### vmap 문

## 추가 명령

### monitor

## 오류 보고

## 종료 상태

## SEE ALSO

`libnftables(3)`, `libnftables-json(5)`, `iptables(8)`, `ip6tables(8)`, `arptables(8)`, `ebtables(8)`, `ip(8)`, `tc(8)`

공식 위키: <https://wiki.nftables.org>

## AUTHORS

Patrick McHardy와 Pablo Neira Ayuso가 Netfilter 커뮤니티의 여러 다른 공헌자들과 함께 nftables를 작성했다.

## COPYRIGHT

Copyright © 2008-2014 Patrick McHardy <kaber@trash.net> Copyright © 2013-2018 Pablo Neira Ayuso <pablo@netfilter.org>

nftables is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License version 2 as published by the Free Software Foundation.

This documentation is licensed under the terms of the Creative Commons Attribution-ShareAlike 4.0 license, CC BY-SA 4.0 <http://creativecommons.org/licenses/by-sa/4.0/>.

----

12/06/2019 a8347553
