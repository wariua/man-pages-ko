## NAME

pam.conf, pam.d - PAM 설정 파일

## DESCRIPTION

*PAM*을 이용하는 권한 부여 응용이 동작을 시작하면서 PAM API와 연결된 코드를 작동시킨다. 거기서 여러 작업을 수행하는데, 가장 중요한 게 설정 파일 /etc/pam.conf를 읽는 것이다. 또는 /etc/pam.d/ 디렉터리의 내용물을 읽을 수도 있다. 그 디렉터리가 존재하면 Linux-PAM에서 /etc/pam.conf를 무시하게 된다.

그 파일들에는 서비스에 필요한 인증 작업을 해 줄 *PAM*들과 개별 *PAM*이 실패한 경우의 적절한 PAM API 동작 방식이 나열돼 있다.

/etc/pam.conf 설정 파일의 문법은 이렇다. 파일은 규칙들의 목록으로 이뤄져 있다. 각 규칙은 보통 한 행으로 돼 있지만 이스케이프 개행 `\<LF>`으로 연장할 수도 있다. 주석은 `#` 기호로 시작해서 행 끝까지 이어진다.

각 행은 공백으로 구분된 토큰들의 집합 형식으로 돼 있으며, 그 중 처음 세 개는 대소문자를 구별하지 않는다.

**service type control module-path module-arguments**

/etc/pam.d/ 디렉터리에 담긴 파일의 문법은 *service* 필드가 빠져 있다는 점을 빼면 동일하다. 이 경우 /etc/pam.d/ 디렉터리 내의 파일 이름이 *service*가 된다. 이 파일 이름은 소문자여야 한다.

*PAM*의 중요한 특징은 여러 규칙들을 *스택처럼 쌓아서* 해당 인증 작업에 여러 PAM 서비스들을 조합할 수 있다는 점이다.

*service*는 보통 해당 응용의 익숙한 이름이다. *login*과 *su*가 좋은 예다. *service* 이름 *other*는 *기본* 규칙들을 위해 예약돼 있다. 어떤 서비스 응용이 주어지면 그 서비스가 명시된 행들만 (그런 행이 없는 경우엔 *other* 항목들을) 이용한다.

*type*은 그 규칙이 해당하는 관리 그룹이다. 이를 이용해 이어지는 모듈이 어느 관리 그룹에 연계돼야 하는지 지정한다. 유효한 항목은 다음과 같다.

account
:   이 모듈 타입은 인증을 기반으로 하지 않는 계정 관리를 수행한다. 보통 현재 시간이나 현재 가용 시스템 자원 (최대 사용자 수), 신청 사용자의 위치에 따라 ('root' 루그인을 콘솔에서만) 서비스 접근 제한/허용하는 데 쓴다.

auth
:   이 모듈 타입은 사용자 인증의 두 가지 측면을 제공한다. 첫째, 응용에서 사용자에게 암호나 기타 식별 수단을 입력받도록 해서 사용자가 스스로 주장하는 그 사람인지 확인한다. 둘째, 크리덴셜 승인 속성을 통해 그룹 소속이나 기타 권한을 승인할 수 있다.

password
:   이 모듈 타입은 사용자와 연계된 인증 토큰을 갱신하는 데 필요하다. 보통 '시험/응답' 기반 인증(auth) 타입마다 모듈이 하나씩 있다.

session
:   이 모듈 타입은 사용자가 서비스를 받기 전이나 받은 후에 해야 할 일과 연관돼 있다. 사용자와의 어떤 데이터 교환 개시/종료 관련 정보를 로그로 기록하기, 디렉터리 마운트 등이 포함된다.

위 목록에 있는 *type* 값 앞에 `-` 문자가 붙어 있으면 그 모듈이 시스템 내에 없어서 적재할 수 없는 경우에도 PAM 라이브러리가 시스템 로그에 기록을 남기지 않는다. 시스템에 늘 설치돼 있는 건 아니어서 로그인 세션을 올바로 인증하고 인가하는 데 꼭 필요하진 않은 모듈에 특히 유용할 수 있다.

세 번째 필드인 *control*은 모듈에서 인증 작업을 성공하지 못했을 때 PAM API의 동작을 나타낸다. 이 control 필드에는 두 가지 문법이 있는데, 단순 문법은 간단한 키워드를 쓰는 것이고, 복잡한 문법은 *value=action* 짝들을 꺽쇠괄호로 감싸는 방식이다.

(전통적인) 단순 문법에서 요효한 *control* 값은 다음과 같다.

required
:   이 PAM 모듈의 실패가 궁극적으로 PAM API의 실패 반환을 일으키되, *쌓여 있는* (이 *service* 및 *type*에 대한) 나머지 모듈들을 호출한 후에 그렇게 된다.

requisite
:   *required*와 비슷하되, 모듈이 실패를 반환하는 경우 바로 응용이나 상위 PAM 스택으로 제어가 반환된다. 실패한 첫 번째 required 내지 requisite 모듈의 반환 값이 반환된다. 참고로 사용자가 안전하지 않은 매체로 패스워드를 입력하게 될 가능성을 막기 위해 이 플래그를 사용할 수 있다. 그런데 이런 동작이 공격자에게 시스템의 유효 계정에 대해 알려 줄 수 있다는 것도 가능한 일이다. 적대적 환경에서 민감한 패스워드 노출에 대한 염려가 무시할 만한 수준이 아닌 경우에 두 가능성의 경중을 따져 볼 필요가 있다.

sufficient
:   이 모듈이 성공하고 앞서 실패한 *required* 모듈이 없으면 PAM 프레임워크가 스택의 이후 모듈을 호출하지 않고 즉시 응용이나 상위 PAM 스택으로 성공을 반환한다. *sufficient* 모듈의 실패는 무시되며 PAM 모듈 스택 처리가 영향 없이 계속 진행된다.

optional
:   스택에서 이 *service*+*type*에 연계된 유일한 모듈인 경우에만 이 모듈의 성공 내지 실패가 영향을 준다.

include
:   인자로 지정한 설정 파일에서 해당 타입의 행을 모두 가져다 포함시킨다.

substack
:   인자로 지정한 설정 파일에서 해당 타입의 행을 모두 가져다 포함시킨다. *include*와의 차이는 하위 스택에서의 *done* 및 *die* 동작이 모듈 스택 전체가 아니라 그 하위 스택의 나머지 부분만 건너뛰게 한다는 점이다. 하위 스택에서의 점프 역시 그 밖으로 평가 위치를 점프하지 못하며, 부모 스택에서 점프가 이뤄질 때는 하위 스택 전체를 모듈 한 개로 센다. *reset* 동작은 모듈 스택의 상태를 하위 스택 평가를 시작할 때의 상태로 재설정한다.

복잡한 문법에서 유효한 *control* 값은 다음 형식이다.

```text
[value1=action1 value2=action2 ...]
```

여기서 *valueN*은 그 행에 지정된 모듈의 호출 함수가 반환한 코드에 해당한다. 다음 중 하나를 선택한다: *success*, *open_err*, *symbol_err*, *service_err*, *system_err*, *buf_err*, *perm_denied*, *auth_err*, *cred_insufficient*, *authinfo_unavail*, *user_unknown*, *maxtries*, *new_authtok_reqd*, *acct_expired*, *session_err*, *cred_unavail*, *cred_expired*, *cred_err*, *no_module_data*, *conv_err*, *authtok_err*, *authtok_recover_err*, *authtok_lock_busy*, *authtok_disable_aging*, *try_again*, *ignore*, *abort*, *authtok_expired*, *module_unknown*, *bad_item*, *conv_again*, *incomplete*, *default*.

이 중 마지막의 *default*는 명시적으로 언급되지 않은 '모든 *valueN*'을 뜻한다. 참고로 전체 PAM 오류 목록을 /usr/include/security/\_pam\_types.h에서 볼 수 있다. *actionN*은 다음 중 한 형태일 수 있다.

ignore
:   모듈 스택에서 사용 시 모듈의 반환 상태가 응용이 얻을 반환 코드에 영향을 주지 않게 된다.

bad
:   이 동작은 반환 코드를 모듈 실패를 나타내는 것으로 봐야 한다는 뜻이다. 이 모듈이 스택에서 실패한 첫 모듈이면 그 상태 값을 전체 스택의 값으로 쓰게 된다.

die
:   bad와 같되, 모듈 스택 처리를 종료하고 즉시 PAM에서 응용으로 반환한다.

ok
:   관리자가 보기에 이 반환 코드가 그대로 전체 모듈 스택의 반환 코드가 돼야 한다는 뜻이다. 달리 말해 스택의 이전 상태가 *PAM_SUCCESS* 반환으로 이어질 상태였던 경우에 모듈 반환 코드가 그 값을 오버라이드하게 된다. 참고로 스택의 이전 상태가 모듈 실패를 나타내는 어떤 값이었던 경우에는 이 'ok' 값으로 그 값을 오버라이드하지 않는다.

done
:   ok와 같되, 앞서 무시되지 않은 모듈 실패가 없었다면 모듈 스택 처리를 종료하고 즉시 PAM에서 응용으로 반환한다.

N (부호 없는 정수)
:   ok와 같되, 스택의 다음 N개 모듈을 건너뛴다. N을 0으로 하는 건 허용되지 않으며, 그 경우 ok와 동일하게 된다.

reset
:   메모리에서 모듈 스택의 상태를 모두 비우고 스택의 다음 모듈부터 새로 시작한다.

네 가지 키워드(required, requisite, sufficient, optional)를 [...] 문법으로 동등하게 표현할 수 있다. 다음과 같다.

required
:   `[success=ok new_authtok_reqd=ok ignore=ignore default=bad]`

requisite
:   `[success=ok new_authtok_reqd=ok ignore=ignore default=die]`

sufficient
:   `[success=done new_authtok_reqd=done default=ignore]`

optional
:   `[success=ok new_authtok_reqd=ok default=ignore]`

*module-path*는 응용에서 이용할 PAM의 전체 파일 이름('/'로 시작)이거나 기본 모듈 위치 기준의 상대 경로명이다. 기본 모듈 위치는 아키텍처에 따라 /lib/security/ 또는 /lib64/security/다.

*module-arguments*는 해당 PAM의 구체적 동작 방식을 바꾸는 데 쓸 수 있는 공백 구분 토큰 목록이다. 개별 모듈별로 그런 인자들이 문서화돼 있다. 참고로 인자에 공백을 포함시키고 싶다면 그 인자를 꺽쇠괄호로 감싸야 한다.

```text
squid auth required pam_mysql.so user=passwd_query passwd=mada \
      db=eminence [query=select user_name from internet_service \
      where user_name='%u' and password=PASSWORD('%p') and \
    service='web_proxy']
```

이 방식 사용 시에 문자열 안에 `[` 문자를 포함시킬 수 있다. 문자열 안에 `]` 문자를 포함시키면서 인자의 일부로 파싱되게 하고 싶으면 `\]`를 써야 한다. 즉 다음처럼 파싱된다.

```text
[..[..\]..]    -->   ..[..]..
```

설정 파일(들 중 하나)의 어느 행이라도 올바른 형식이 아니면 일반적으로 (과잉으로 조심하는 쪽을 택해서) 인증 과정이 실패하게 된다. `syslog(3)` 호출을 이용해 해당 오류를 시스템 로그 파일로 기록한다.

설정 파일 하나를 쓰는 것보다 유연하게 libpam을 구성하는 방식으로 /etc/pam.d/ 디렉터리 내용물을 통해 설정하는 방식이 있다. 이 경우 그 디렉터리를 채운 파일들의 이름이 (소문자) 서비스 이름과 같다. 즉, 그 이름의 서비스에 대한 개별 설정 파일이 된다.

/etc/pam.d/ 내 파일의 문법은 /etc/pam.conf 파일과 비슷하며 다음 형식의 행들로 이뤄진다.

```text
type  control  module-path  module-arguments
```

유일한 차이는 서비스 이름이 없다는 점이다. 물론 해당 설정 파일의 이름이 서비스 이름이다. 예를 들어 /etc/pam.d/login이 **login** 서비스에 대한 설정을 담고 있다.

## SEE ALSO

`pam(3)`, `PAM(8)`, `pam_start(3)`

----

2017-05-18
