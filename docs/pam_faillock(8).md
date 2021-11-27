## NAME

pam_faillock - 지정한 기간 동안 인증 실패 횟수를 세는 모듈

## SYNOPSIS

<pre>
<strong>auth ... pam_faillock.so</strong> {preauth|authfail|authsucc}
                         [conf=<em>/path/to/config-file</em>]
                         [dir=<em>/path/to/tally-directory</em>] [even_deny_root]
                         [deny=<em>n</em>] [fail_interval=<em>n</em>] [unlock_time=<em>n</em>]
                         [root_unlock_time=<em>n</em>] [admin_group=<em>name</em>] [audit]
                         [silent] [no_log_info]

<strong>account ... pam_faillock.so</strong> [dir=<em>/path/to/tally-directory</em>] [no_log_info]
</pre>

## DESCRIPTION

이 모듈은 실패한 인증 시도 목록을 지정한 시간 동안 사용자별로 유지해서 연속으로 *deny* 번 넘게 인증이 실패한 경우 계정을 잠근다.

서비스 거부를 방지하기 위해 기본적으로 *root*는 인증 시도가 실패해도 계정을 차단하지 **않는다**. 사용자들에게 셸 계정을 주지 않으며 `su`나 (telnet/rsh 등이 아닌) 장비 콘솔을 통해서만 root가 로그인할 수 있다면 이렇게 해도 안전하다.

## OPTIONS

`{preauth|authfail|authsucc}`
:   PAM 스택에서 모듈 인스턴스의 위치에 따라 이 인자를 설정해야 한다.

    패스워드 같은 사용자 크리덴셜을 묻는 모듈 앞에서 이 모듈을 호출할 때 `preauth` 인자를 써야 한다. 그러면 최근에 비정상적으로 많이 연속으로 인증이 실패해서 사용자가 서비스에 접근하지 못하게 차단해야 하는지 여부만 검사한다. `authsucc`를 쓴다면 이 호출을 하지 않아도 된다.

    인증 결과가 실패라고 판단하는 모듈 다음에서 이 모듈을 호출할 때 `authfail` 인자를 써야 한다. 앞선 인증 실패로 인해 사용자가 이미 차단돼 있는 경우가 아니면 적절한 사용자별 기록 파일에 실패를 기록한다.

    인증 결과가 성공이라고 판단하는 모듈 다음에서 이 모듈을 호출할 때 `authsucc` 인자를 써야 한다. 앞선 인증 실패로 인해 사용자가 이미 차단돼 있는 경우가 아니면 사용자별 기록 파일에서 실패 기록을 모두 지운다. 이미 차단돼 있는 경우라면 인증 오류를 반환한다. 이 호출을 하지 않으면 pam_faillock에서 연속 인증 실패와 불연속 실패를 구별할 수 없게 된다. 그 경우에는 `preauth` 호출을 해 줘야 한다. PAM 스택 구성의 어떤 이유 때문에 *pam_faillock*을 account 모듈로 호출하는 것도 가능하다. 그 구성에선 `preauth` 단계에서도 모듈을 호출해야 한다.

`conf=/path/to/config-file`
:   기본 설정 파일 /etc/security/faillock.conf 대신 다른 파일을 쓴다.

모듈 동작을 설정하는 옵션들을 <tt>[[faillock.conf(5)]]</tt> 맨 페이지에서 설명한다. 설정 파일의 값보다 모듈 명령행에 지정한 옵션이 우선한다.

## 제공하는 모듈 종류

**auth** 및 **account** 모듈 타입을 제공한다.

## RETURN VALUES

`PAM_AUTH_ERR`
:   유효하지 않은 옵션을 줬거나, 모듈에서 사용자 이름을 얻을 수 없거나, 유효한 카운터 파일을 찾을 수 없거나, 너무 많은 로그인 실패.

`PAM_BUF_ERR`
:   메모리 버퍼 오류.

`PAM_CONV_ERR`
:   응용에서 제공한 대화 함수에서 사용자명을 얻는 데 실패했다.

`PAM_INCOMPLETE`
:   응용에서 제공한 대화 함수에서 PAM_CONV_AGAIN을 반환했다.

`PAM_SUCCESS`
:   성공적으로 동작했음.

`PAM_IGNORE`
:   passwd 데이터베이스에 사용자가 없음.

## NOTES

모듈 명령행에 옵션을 지정하는 걸 권하지 않는다. 대신 /etc/security/faillock.conf를 이용하는 게 좋다.

PAM 스택에서 *pam_faillock*의 동작 구조가 *pam_tally* 모듈과 다르다.

사용자별로 실패 기록을 담은 파일이 그 사용자를 소유자로 해서 따로 만들어진다. 그래서 스크린세이버에서 호출할 때도 `pam_faillock.so` 모듈이 잘 동작하게 된다.

모르는 사용자에 대해선 실패를 기록하지 않기 때문에 /etc/security/faillock.conf에 `silent` 옵션을 지정하지 않고서, 또는 control 필드를 `requisite`로 해서 `preauth`로 이 모듈을 사용하면 시스템에 어떤 사용자 계정이 존재하는지 여부에 대한 정보가 누출된다. 존재하지 않는 사용자 계정에 대해선 사용자 계정이 잠긴다는 메시지가 절대 표시되지 않으므로 특정 계정이 시스템에 존재하지 않는다는 걸 공격자가 추론할 수 있게 된다.

## EXAMPLES

다음은 /etc/pam.d/login에 가능한 두 가지 예시 설정이다. 기본 시간인 15분 동안 4번 연속으로 로그인이 실패한 다음에 *pam_faillock*에서 계정을 잠그게 한다. root 계정은 잠기지 않는다. 20분 후에 자동으로 계정 잠금이 풀린다.

첫 번째 예에선 *auth* 단계에서만 모듈을 호출하며 *pam_faillock*에 의해 잠기는 계정에 대한 정보를 찍지 않는다. 계정이 잠겼을 때 사용자에게 로그인이 차단돼 있다는 걸 알려 주고 인증을 중단해서 패스워드를 묻지도 않게 하고 싶다면 `preauth` 호출을 추가하면 된다.

`/etc/security/faillock.conf` 파일 예시:

```config
deny=4
unlock_time=1200
silent
```

`/etc/pam.d/login` 파일 예시:

```text
auth     required       pam_securetty.so
auth     required       pam_env.so
auth     required       pam_nologin.so
# 계정 잠금에 대한 메시지를 표시하려면:
# auth requisite pam_faillock.so preauth
auth     [success=1 default=bad] pam_unix.so
auth     [default=die]  pam_faillock.so authfail
auth     sufficient     pam_faillock.so authsucc
auth     required       pam_deny.so
account  required       pam_unix.so
password required       pam_unix.so shadow
session  required       pam_selinux.so close
session  required       pam_loginuid.so
session  required       pam_unix.so
session  required       pam_selinux.so open
```

두 번째 예에선 *auth* 단계와 *account* 단계 모두에서 모듈을 호출한다. faillock.conf에 `silent` 옵션이 지정돼 있지 않으면 계정이 잠겨 있다는 걸 인증 시도 사용자에게 알려 준다.
```text
auth     required       pam_securetty.so
auth     required       pam_env.so
auth     required       pam_nologin.so
auth     required       pam_faillock.so preauth
# 잠긴 계정에 패스워드를 묻지 않고 싶다면 위 행을 requisite로
auth     sufficient     pam_unix.so
auth     [default=die]  pam_faillock.so authfail
auth     required       pam_deny.so
account  required       pam_faillock.so
# 위의 pam_faillock.so 호출이 빠지면 연속이 아닌 인증 실패에
# 대해서도 잠금이 이뤄진다.
account  required       pam_unix.so
password required       pam_unix.so shadow
session  required       pam_selinux.so close
session  required       pam_loginuid.so
session  required       pam_unix.so
session  required       pam_selinux.so open
```

## FILES

`/var/run/faillock/*`
:   사용자 인증 실패를 기록하는 파일들

`/etc/security/faillock.conf`
:   pam_faillock 옵션 설정 파일

## SEE ALSO

`faillock(8)`, <tt>[[faillock.conf(5)]]</tt>, <tt>[[pam.conf(5)]]</tt>, <tt>[[pam.d(5)]]</tt>, `pam(8)`

## AUTHOR

Tomas Mraz가 pam_faillock을 작성했다.

----

2020-06-08
