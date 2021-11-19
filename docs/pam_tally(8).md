## NAME

pam_tally - 로그인 카운터 (집계) 모듈

## SYNOPSIS

<pre>
<strong>pam_tally.so</strong> [file=<em>/path/to/counter</em>] [onerr=[<em>fail</em>|<em>succeed</em>]] [magic_root]
             [even_deny_root_account] [deny=<em>n</em>] [lock_time=<em>n</em>] [unlock_time=<em>n</em>]
             [per_user] [no_lock_time] [no_reset] [audit] [silent]
             [no_log_info]

<strong>pam_tally</strong> [--file <em>/path/to/counter</em>] [--user <em>username</em>] [--reset[=<em>n</em>]] [--quiet]
</pre>

## DESCRIPTION

이 모듈은 접근 시도 횟수를 유지한다. 성공 시 카운트를 초기화할 수 있으며 시도가 너무 많이 실패한 경우 접근을 거부할 수 있다.

pam_tally의 여러 한계점이 pam_tally2에서 해결되었다. 따라서 pam_tally 사용을 권장하지 않으며 향후 릴리스에서 제거될 예정이다.

pam_tally는 `pam_tally.so`와 `pam_tally` 두 요소로 돼 있다. 앞쪽은 PAM 모듈이고 뒤쪽은 단독 프로그램이다. `pam_tally`는 카운터 파일을 조회하고 조작하는 데 쓸 수 있는 (선택적) 응용이다. 사용자 카운트를 표시하거나, 개별 카운트를 설정하거나, 모든 카운트를 초기화할 수 있다. 카운트를 인위적으로 높게 설정하는 것으로 패스워드를 바꾸지 않고 사용자를 차단할 수도 있다. 그리고 예를 들어 cron 작업으로 자정마다 모든 카운트를 초기화하는 게 유용할 수도 있다. 카운터 파일을 관리하는 데 pam_tally 대신 `faillog(8)` 명령을 이용할 수 있다.

서비스 거부를 방지하기 위해 기본적으로 *root*는 접근 시도가 실패해도 계정을 차단하지 **않는다**. 사용자들에게 셸 계정을 주지 않으며 `su`나 (telnet/rsh 등이 아닌) 장비 콘솔을 통해서만 root가 로그인할 수 있다면 이렇게 해도 안전하다.

## OPTIONS

### 전역 옵션

*auth* 및 *account* 모듈 타입에 쓸 수 있다.

`onerr=[fail|succeed]`
:   이상한 (파일을 열 수 없는 등의) 상황이 발생한 경우에 `onerr=succeed`이면 `PAM_SUCCESS`를 반환하고, 아니면 대응하는 PAM 오류 코드를 반환한다.

`file=/path/to/counter`
:   카운트를 유지할 파일. 기본은 /var/log/faillog.

`audit`
:   알 수 없는 사용자인 경우 시스템 로그로 사용자 이름을 기록한다.

`silent`
:   정보성 메시지를 찍지 않는다.

`no_log_info`
:   `syslog(3)`를 통해 정보성 메시지를 기록하지 않는다.

### AUTH 옵션

인증 단계에서는 먼저 사용자가 접근이 거부돼야 하는지 확인하고, 아니라면 로그인 시도 횟수를 올린다. 그리고 `pam_setcred(3)` 호출 시에 시도 횟수를 초기화한다.

`deny=n`
:   사용자의 집계 횟수가 `n` 번을 넘으면 접근을 거부한다.

`lock_time=n`
:   시도 실패 후 `n` 초 동안은 항상 거부한다.

`unlock_time=n`
:   시도 실패 후 `n` 초가 지나면 접근을 허용한다. 이 옵션을 쓰면 최대 시도 횟수를 초과했을 때 지정된 시간 동안 사용자 계정이 잠기게 된다. 이 옵션을 안 쓰면 시스템 관리자가 직접 잠금을 풀어 주기 전까지 계정이 잠겨 있게 된다.

`magic_root`
:   uid=0인 사용자로 모듈이 호출됐을 때는 카운터를 올리지 않는다. `su`처럼 사용자가 띄우는 서비스에는 이 인자를 쓰는 게 좋고 다른 서비스에는 빼는 게 좋다.

`no_lock_time`
:   /var/log/faillog의 .fail_locktime 필드를 쓰지 않는다.

`no_reset`
:   진입 성공 시 횟수를 초기화하지는 않고 내리기만 한다.

`even_deny_root_account`
:   root 계정도 사용 불가능하게 될 수 있다.

`per_user`
:   /var/log/faillog에서 사용자의 .fail_max/.fail_locktime 필드가 0이 아니면 `deny=n`/ `lock_time=n` 매개변수 대신 그 값을 쓴다.

`no_lock_time`
:   /var/log/faillog의 .fail_locktime 필드를 쓰지 않는다.

### ACCOUNT 옵션

account 단계에서는 사용자가 magic root가 아니면 시도 횟수를 초기화한다. `pam_setcred(3)`를 제대로 호출하지 않는 서비스에, 또는 다른 모듈의 account 단계 실패와 상관없이 초기화를 해야 하는 경우에 선택적으로 이 단계를 사용할 수 있다.

`magic_root`
:   uid=0인 사용자로 모듈이 호출됐을 때는 카운터를 올리지 않는다. `su`처럼 사용자가 띄우는 서비스에는 이 인자를 쓰는 게 좋고 다른 서비스에는 빼는 게 좋다.

`no_reset`
:   진입 성공 시 횟수를 초기화하지는 않고 내리기만 한다.

## 제공하는 모듈 종류

**auth** 및 **account** 모듈 타입을 제공한다.

## RETURN VALUE

`PAM_AUTH_ERR`
:   유효하지 않은 옵션을 줬거나, 모듈에서 사용자 이름을 얻을 수 없거나, 유효한 카운터 파일을 찾을 수 없거나, 너무 많은 로그인 실패.

`PAM_SUCCESS`
:   성공적으로 동작했음.

`PAM_USER_UNKNOWN`
:   알 수 없는 사용자.

## EXAMPLES

/etc/pam.d/login에 다음 행을 추가해서 로그인에 너무 많이 실패했을 때 계정을 잠글 수 있다. 허용 실패 횟수를 /var/log/faillog로 지정하며 pam_tally나 `faillog(8)`로 미리 설정해 둬야 한다.

```text
auth     required       pam_securetty.so
auth     required       pam_tally.so per_user
auth     required       pam_env.so
auth     required       pam_unix.so
auth     required       pam_nologin.so
account  required       pam_unix.so
password required       pam_unix.so
session  required       pam_limits.so
session  required       pam_unix.so
session  required       pam_lastlog.so nowtmp
session  optional       pam_mail.so standard
```

## FILES

`/var/log/faillog`
:   실패 기록 파일

## SEE ALSO

`faillog(8)`, <tt>[[pam.conf(5)]]</tt>, <tt>[[pam.d(5)]]</tt>, <tt>[[pam(7)]]</tt>

## AUTHOR

Tim Baverstock과 Tomas Mraz가 pam_tally를 작성했다.

----

2017-05-18
