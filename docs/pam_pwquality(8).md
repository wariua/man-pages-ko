## NAME

pam_pwquality - 패스워드 품질 검사를 수행하는 PAM 모듈

## SYNOPSIS

<pre>
<strong>pam_pwquality.so</strong> [<em>...</em>]
</pre>

## DESCRIPTION

이 모듈을 어떤 서비스의 **password** 스택에 끼워 넣어서 패스워드에 대한 강도 검사를 제공할 수 있다. pam_cracklib 모듈 코드를 기반으로 하며 그 모듈의 옵션들이 그대로 호환된다.

이 모듈이 하는 동작은 사용자에게 패스워드를 입력받아서 시스템 사전과 여러 규칙들로 강도를 검사해서 부족한 값인지 확인하는 것이다.

첫 번째 동작은 패스워드를 한 번 입력받는 것이고, 강도를 검사해서 강력한 패스워드인 것 같으면 (올바로 입력했던 건지 검증하기 위해) 두 번째로 패스워드를 입력받는다. 문제가 없으면 그 패스워드가 후속 모듈들로 전달돼서 새 인증 토큰으로 설치된다.

강도 검사 방법은 이렇다.

회문
:   새 패스워드가 회문(팰린드롬)인가?

대소문자 변경
:   새 패스워드가 이전 패스워드에서 대소문자만 바꾼 것인가?

유사
:   새 패스워드가 이전 패스워드와 너무 비슷한가? 기본적으로 `difok`라는 인자 하나로 제어하는데, 이전 패스워드와 새 패스워드 간에 글자 변경(삽입, 제거, 대체)이 적어도 이만큼 있어야 새 패스워드를 받아들인다.

단순
:   새 패스워드가 너무 단순한가? 6개 인자 `minlen`, `maxclassrepeat`, `dcredit`, `ucredit`, `lcredit`, `ocredit`로 제어한다. 동작 방식과 기본값에 대해선 인자에 대한 절 참고.

교대
:   새 패스워드가 이전 패스워드의 앞뒤를 바꾼 것인가?

같은 문자 연속
:   연속으로 같은 문자를 선택적으로 검사.

너무 긴 단조 증가/감소열
:   너무 길게 단조 증가/감소하는 문자열을 선택적으로 검사.

사용자 이름 포함
:   패스워드에 사용자의 이름이 어떤 형태로든 포함돼 있는지를 선택적으로 검사.

사전 검사
:   *Cracklib* 루틴을 호출해서 패스워드가 사전에 있는 단어인지 확인한다.

모듈 인자를 쓰거나 설정 파일 `/etc/security/pwquality.conf`를 변경해서 이 검사들의 동작을 설정할 수 있다. 모듈 인자가 설정 파일의 설정보다 우선한다.

## OPTIONS

`debug`
:   이 옵션은 모듈 동작을 나타내는 정보를 `syslog(3)`로 기록하게 한다. (패스워드 정보는 로그 파일로 기록하지 않는다.)

`authtok_type=XXX`
:   모듈의 기본 동작은 패스워드 요청 시 "New UNIX password: " 및 "Retype UNIX password: " 프롬프트를 띄우는 것이다. 이 옵션을 쓰면 예시의 단어 *UNIX*를 바꿀 수 있으며, 기본은 빈 값이다.

`retry=N`
:   사용자에게 최대 `N` 번 물어본 다음 오류를 반환한다. 기본은 `1`번이다.

`difok=N`
:   이 인자는 새 패스워드가 이전 패스워드와 달라야 하는 문자 수를 기본값 `1`에서 다른 값으로 바꾼다.

    특수값 `0`은 새 패스워드와 이전 패스워드의 유사성 검사들을 모두 끈다. 단, 새 패스워드가 이전 패스워드와 똑같은지는 검사한다.

`minlen=N`
:   새 패스워드의 허용 가능한 최소 크기. (점수 동작이 꺼져 있지 않으면 1씩 더한 값. 기본적으로 꺼져 있다.) 새 패스워드의 문자 수에 각 문자 종류(*other*, *upper*, *lower*, *digit*)별로 점수가 (길이에 +1이) 더해진다. 이 매개변수의 기본값은 8이다. 참고로 *Cracklib*에도 두 가지 길이 제한이 있어서 사전 검사 때 적용된다. "way too short"이라고 나오는 `4`글자 제한이 하드코딩돼 있고 빌드 때 정의되는 제한(`6`)도 있어서 `minlen`과 무관하게 검사를 하게 된다.

`dcredit=N`
:   (N >= 0) 새 패스워드에서 숫자로 얻을 수 있는 최대 점수. 숫자가 `N` 개 이하로 있으면 각 숫자를 현재 `minlen` 값 검사에 +1씩 계산한다. `dcredit` 기본값은 `0`으로, 패스워드에 있는 숫자가 추가 점수를 주지 않는다는 뜻이다.

    (N < 0) 새 패스워드에 있어야 하는 숫자 최소 개수.

`ucredit=N`
:   (N >= 0) 새 패스워드에서 대문자로 얻을 수 있는 최대 점수. 대문자가 `N` 개 이하로 있으면 각 대문자를 현재 `minlen` 값 검사에 +1씩 계산한다. `ucredit` 기본값은 `0`으로, 패스워드에 있는 대문자가 추가 점수를 주지 않는다는 뜻이다.

    (N < 0) 새 패스워드에 있어야 하는 대문자 최소 개수.

`lcredit=N`
:   (N >= 0) 새 패스워드에서 소문자로 얻을 수 있는 최대 점수. 소문자가 `N` 개 이하로 있으면 각 소문자를 현재 `minlen` 값 검사에 +1씩 계산한다. `lcredit` 기본값은 `0`으로, 패스워드에 있는 소문자가 추가 점수를 주지 않는다는 뜻이다.

    (N < 0) 새 패스워드에 있어야 하는 소문자 최소 개수.

`ocredit=N`
:   (N >= 0) 새 패스워드에서 기타 문자로 얻을 수 있는 최대 점수. 기타 문자가 `N` 개 이하로 있으면 각 기타 문자를 현재 `minlen` 값 검사에 +1씩 계산한다. `ocredit` 기본값은 `0`으로, 패스워드에 있는 기타 문자가 추가 점수를 주지 않는다는 뜻이다.

    (N < 0) 새 패스워드에 있어야 하는 기타 문자 최소 개수.

`minclass=N`
:   새 패스워드에 있어야 하는 문자 종류 최소 가짓수. 네 가지 문자 종류는 숫자, 대문자, 소문자, 기타 문자다. `credit` 검사와의 차이는 특정 문자 종류를 요구하는 게 아니라는 점이다. 네 가지 종류 중에 `N` 가지가 있으면 된다. 기본적으로 검사가 꺼져 있다.

`maxrepeat=N`
:   같은 문자가 `N` 개 넘게 연이어 나오는 패스워드를 거부. 기본은 0으로, 이 검사를 하지 않는다는 뜻이다.

`maxsequence=N`
:   단조 변화하는 `N` 개보다 긴 문자열이 있는 패스워드를 거부. 기본은 0으로, 이 검사를 하지 않는다는 뜻이다. '12345'나 'fedcb'가 그런 열의 예다. 참고로 그런 열이 패스워드의 일부에 불과하지 않는 한 그런 패스워드 대부분은 단순성 검사도 통과하지 못한다.

`maxclassrepeat=N`
:   같은 종류 문자가 `N` 개 넘게 연이어 나오는 패스워드를 거부. 기본은 0으로, 이 검사를 하지 않는다는 뜻이다.

`gecoscheck=N`
:   0이 아니면 사용자의 <tt>[[passwd(5)]]</tt> GECOS 필드에 있는 3글자 넘는 단어가 새 패스워드에 들어가 있는지 검사. 기본은 0으로, 이 검사를 하지 않는다는 뜻이다.

`dictcheck=N`
:   0이 아니면 (다소 변경한) 패스워드가 사전의 단어와 일치하는지 검사. 현재 *cracklib* 라이브러리를 이용해 사전 검사를 수행한다. 기본은 1로, 이 검사를 한다는 뜻이다.

`usercheck=N`
:   0이 아니면 (다소 변경한) 패스워드에 사용자 이름이 어떤 형태로 포함돼 있는지 검사. 기본은 1로, 이 검사를 한다는 뜻이다. 사용자 이름이 3글자보다 짧으면 수행하지 않는다.

`usersubstr=N`
:   (usercheck의 최소 길이로 인한) 3보다 크면 패스워드에 최소 `N` 글자인 사용자 이름 하위문자열이 어떤 형태로 포함돼 있는지 검사. 기본값은 0으로, 이 검사를 하지 않는다는 뜻이다.

`enforcing=N`
:   0이 아니면 검사에 실패한 경우 패스워드를 거부한다. 아니면 경고만 찍는다. 기본값은 1로, (root 아닌 사용자에 대해) 약한 패스워드를 거부한다는 뜻이다.

`badwords=<단어 목록>`
:   이 공백 구분 목록에 있는 3글자보다 긴 단어 각각을 새 패스워드에서 탐색해서 금지한다. 기본적으로 목록이 비어 있으며, 이 검사를 하지 않는다는 뜻이다.

`dictpath=/path/to/dict`
:   이 옵션으로 기본과 다른 cracklib 사전 경로를 지정할 수 있다.

`enforce_for_root`
:   패스워드를 바꾸려고 하는 사용자가 root인 경우에도 검사 실패 시 오류를 반환한다. 기본적으로 이 옵션이 꺼져 있는데, 실패한 검사에 대한 메시지는 찍지만 어쨌든 root는 패스워드를 바꿀 수 있다는 뜻이다. 참고로 root에게는 이전 패스워드를 묻지 않으므로 이전 패스워드와 새 패스워드를 비교하는 검사를 수행하지 않는다.

`local_users_only`
:   `/etc/passwd` 파일에 없는 사용자에 대해선 패스워드 품질을 검사하지 않는다. 그래도 스택 후속 모듈에서 `use_authtok` 옵션을 쓸 수 있도록 패스워드를 묻기는 한다. 기본적으로 이 옵션이 꺼져 있다.

`use_authtok`
:   이 모듈에서 사용자에게 새 패스워드를 묻지 않고 스택 앞쪽 **password** 모듈에서 제공한 패스워드를 이용하도록 *강제한다*.

## 제공하는 모듈 종류

**password** 모듈 타입만 제공한다.

## RETURN VALUES

`PAM_SUCCESS`
:   새 패스워드가 모든 검사를 통과했다.

`PAM_AUTHTOK_ERR`
:   새 패스워드가 입력되지 않았거나, 사용자 이름을 알 수 없거나, 새 패스워드가 강도 검사를 통과하지 못했다.

`PAM_AUTHTOK_RECOVERY_ERR`
:   스택 앞쪽 모듈에서 이전 패스워드를 제공하지 않았거나, 사용자에게 요청했지만 얻지 못했다. `use_authtok` 지정 시 앞쪽 오류가 발생할 수 있다.

`PAM_SERVICE_ERR`
:   내부 오류가 발생했다.

## EXAMPLES

이 모듈 사용 예시로 `pam_unix(8)`의 password 부문과 함께 사용하는 방법을 살펴보자.

```text
#
# 두 가지 password 타입 모듈을 쌓는다. 이 예에선 강력한 패스워드를
# 입력할 기회가 사용자에게 3번 주어진다. "use_authtok" 인자는
# pam_unix 모듈에서 패스워드를 묻지 말고 pam_pwquality에서 제공한
# 걸 쓰도록 한다.
#
password required pam_pwquality.so retry=3
password required pam_unix.so use_authtok
```

또 다른 예시는 sha256 패스워드 암호화를 쓰려는 경우를 위한 것이다.

```text
#
# 요즘 시스템에서 최소 14바이트의 패스워드를 지원할 수 있게 하고,
# 숫자에 2점과 기타 문자에 2점씩 추가 점수를 준다. 새 패스워드에는
# 이전 패스워드에 없는 바이트가 적어도 3개 있어야 한다.
#
password required pam_pwquality.so \
              difok=3 minlen=15 dcredit=2 ocredit=2
password required pam_unix.so use_authtok nullok sha256
```

또 다른 예시는 점수 방식을 쓰지 않으려는 경우다.

```text
#
# 사용자가 최소 길이 8, 숫자 최소 1개, 대문자 최소 1개,
# 기타 문자 최소 1개인 패스워드를 고르도록 한다.
#
password required pam_pwquality.so \
              dcredit=-1 ucredit=-1 ocredit=-1 lcredit=0 minlen=8
password required pam_unix.so use_authtok nullok sha256
```

## SEE ALSO

`pwscore(1)`, <tt>[[pwquality.conf(5)]]</tt>, `pam_pwquality(8)`, <tt>[[pam.conf(5)]]</tt>, `PAM(8)`

## AUTHORS

Tomas Mraz <tmraz@redhat.com>

**pam_cracklib** 모듈의 원작성자 Cristian Gafton <gafton@redhat.com>

----

2020-08-03
