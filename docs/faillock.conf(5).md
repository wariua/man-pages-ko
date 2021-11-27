## NAME

faillock.conf - pam_faillock 설정 파일

## DESCRIPTION

`faillock.conf`를 이용하면 여러 번의 인증 시도 실패 후 사용자 잠그기 기본 동작 방식을 설정할 수 있다. *pam_faillock* 모듈에서 이 파일을 읽으며 *pam_faillock*에 직접 설정을 주는 것보다 권장하는 방식이다.

아주 단순한 *이름 = 값* 형식이며 `#` 문자로 시작하는 주석이 있을 수 있다. 행 처음과 끝, `=` 부호 앞뒤의 공백은 무시된다.

## OPTIONS

`dir=/path/to/tally-directory`
:   실패 기록을 담은 사용자별 파일들을 유지할 디렉터리. 기본은 /var/run/faillock.

`audit`
:   알 수 없는 사용자인 경우 시스템 로그로 사용자 이름을 기록한다.

`silent`
:   사용자에게 정보성 메시지를 찍지 않는다. 이 옵션을 쓰지 않는 경우 시스템에 존재하는 사용자와 부재하는 사용자에 대해 인증 동작에 차이가 생긴다는 점에 유의하자.

`no_log_info`
:  `syslog(3)`를 통해 정보성 메시지를 기록하지 않는다.

`local_users_only`
:   /etc/passwd 파일에 있는 지역 사용자에 대해서만 사용자 인증 시도 실패를 추적하고 중앙 관리 (AD, IdM, LDAP 등) 사용자들은 무시한다. `faillock(8)` 명령 역시 사용자 실패 인증 시도를 추적하지 않게 된다. 이 옵션을 켜면 사용자가 지역적으로도 잠기고 중앙 관리 메커니즘에서도 잠기는 이중 잠금 상황을 방지하게 된다.

`deny=n`
:   최근 기간 동안 이 사용자에 대해 연속 인증 실패 횟수가 `n`을 넘으면 접근을 거부한다. 기본은 3이다.

`fail_interval=n`
:   `n` 초 기간 동안 연속 인증 실패가 발생해야 사용자 계정이 잠긴다. 기본은 900(15분)이다.

`unlock_time=n`
:   잠금 후 `n` 초가 지나면 접근을 다시 허용한다. 0 값은 `never` 값과 같은 뜻이다. 즉, `faillock(8)` 명령으로 faillock 항목을 초기화하지 않으면 접근이 다시 허용되지 않는다. 기본은 600(10분)이다.

    참고로 *pam_faillock*에서 쓰는 기본 디렉터리가 일반적으로 시스템 부팅 시 비워지기 때문에 시스템 재부팅 후 접근이 다시 가능해진다. 그게 바람직하지 않다면 `dir` 옵션으로 다른 집계 디렉터리를 설정해야 한다.

    또한 사용자를 영구적으로 잠그는 건 일반적으로 바람직하지 않다. 사용자 이름이 난수적이고 잠재적 공격자에게 감춰져 있는 경우가 아니라면 쉽게 서비스 거부 공격의 표적이 될 수 있기 때문이다.

`even_deny_root`
:   root 계정도 일반 계정처럼 잠길 수 있다.

`root_unlock_time=n`
:   이 옵션은 `even_deny_root` 옵션을 함의한다. 계정이 잠긴 후 `n` 초가 지나면 root 계정에 접근을 허용한다. 이 옵션을 지정하지 않으면 `unlock_time` 옵션과 같은 값이다.

`admin_group=name`
:   이 옵션으로 그룹 이름을 지정하면 그 그룹 구성원들을 root 계정과 동일하게 다룬다. 즉, `even_deny_root` 및 `root_unlock_time` 옵션이 그 구성원들에게 적용된다. 기본적으로 이 옵션이 설정돼 있지 않다.

## EXAMPLES

`/etc/security/faillock.conf` 파일 예시:

```text
deny=4
unlock_time=1200
silent
```

## FILES

`/etc/security/faillock.conf`
:   동작 옵션 설정 파일

## SEE ALSO

`faillock(8)`, <tt>[[pam_faillock(8)]]</tt>, <tt>[[pam.conf(5)]]</tt>, <tt>[[pam.d(5)]]</tt>, `pam(8)`

## AUTHOR

Tomas Mraz가 pam_faillock을 작성했다. Brian Ward가 faillock.conf 지원을 작성했다.

----

2020-06-08
