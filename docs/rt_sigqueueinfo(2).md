## NAME

rt_sigqueueinfo, rt_tgsigqueueinfo - 시그널과 데이터를 큐에 넣기

## SYNOPSIS

```c
int rt_sigqueueinfo(pid_t tgid, int sig, siginfo_t *uinfo);

int rt_tgsigqueueinfo(pid_t tgid, pid_t tid, int sig,
                      siginfo_t *uinfo);
```

*주의*: 이 시스템 호출들에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`rt_sigqueueinfo()` 및 `rt_tgsigqueueinfo()` 시스템 호출은 프로세스나 스레드에게 시그널에 데이터를 더해 보내는 데 사용하는 저수준 인터페이스이다. 수신자에서 <tt>[[sigaction(2)]]</tt> `SA_SIGINFO` 플래그로 시그널 핸들러를 설정하여 동반 데이터를 얻을 수 있다.

이 시스템 호출들은 응용에서 직접 사용하기 위한 것이 아니라 <tt>[[sigqueue(3)]]</tt>와 <tt>[[pthread_sigqueue(3)]]</tt>를 구현할 수 있도록 제공하는 것이다.

`rt_sigqueueinfo()` 시스템 호출은 ID가 `tgid`인 스레드 그룹에게 시그널 `sig`를 보낸다. ("스레드 그룹"이라는 용어는 "프로세스"와 동의어이며 `tid`는 전통적인 유닉스 프로세스 ID에 해당한다.) 그 스레드 그룹의 임의 구성원에게 (즉 현재 그 시그널을 막고 있지 않은 스레드들 중 하나에게) 시그널이 전달된다.

`uinfo` 인자는 시그널에 동반되는 데이터를 나타낸다. 이 인자는 <tt>[[sigaction(2)]]</tt>에 기술된 (그리고 `<sigaction.h>`를 포함시켜서 정의하는) `siginfo_t` 타입의 구조체에 대한 포인터이다. 호출자가 이 구조체의 다음 필드들을 설정해야 한다.

<dl>
<dt><code>si_code</code></dt>
<dd>
리눅스 커널 소스 파일 <code>include/asm-generic/siginfo.h</code>에 있는 <code>SI_*</code> 코드들 중 하나여야 하되 코드가 음수여야 하며 (즉 커널에서 <tt>[[kill(2)]]</tt>로 보낸 시그널을 나타내는 데 쓰는 <code>SI_USER</code>일 수 없으며) (리눅스 2.6.39부터) (커널에서 <tt>[[tgkill(2)]]</tt>로 보낸 시그널을 나타내는 데 쓰는) <code>SI_TKILL</code>일 수 없다는 제약이 있다.
</dd>

<dt><code>si_pid</code></dt>
<dd>
프로세스 ID로 설정해야 하는데, 보통은 송신자의 프로세스 ID이다.
</dd>

<dt><code>si_uid</code></dt>
<dd>
사용자 ID로 설정해야 하는데, 보통은 송신자의 실제 사용자 ID이다.
</dd>

<dt><code>si_value</code></dt>
<dd>
이 필드는 시그널에 동반시킬 사용자 데이터를 담는다. 더 자세한 정보는 <tt>[[sigqueue(3)]]</tt>의 마지막(<code>union sigval</code>) 인자 설명을 보라.
</dd>
</dl>

커널에서 내부적으로 `si_signo` 필드를 `sig`에 지정된 값으로 설정해서 시그널 수신자가 그 필드를 통해서도 시그널 번호를 얻을 수 있도록 한다.

`rt_tgsigqueueinfo()` 시스템 호출은 `rt_sigqueueinfo()`와 비슷하되 스레드 그룹 ID `tgid`와 스레드 그룹 내 스레드 `tid`의 조합으로 지정한 한 스레드에게 시그널과 데이터를 보낸다.

## RETURN VALUE

성공 시 이 시스템 호출들은 0을 반환한다. 오류 시 -1을 반환하며 그 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EAGAIN</code></dt>
<dd>큐에 넣을 수 있는 시그널 개수 한계에 도달했다. (자세한 내용은 <tt>[[signal(7)]]</tt>을 보라.)</dd>
<dt><code>EINVAL</code></dt>
<dd><code>sig</code>나 <code>tgid</code>, <code>tid</code>가 유효하지 않다.</dd>
<dt><code>EPERM</code></dt>
<dd>호출자가 대상에게 시그널을 보낼 권한을 가지고 있지 않다. 필요한 권한에 대해선 <tt>[[kill(2)]]</tt>을 보라. 또는 <code>uinfo->si_code</code>가 유효하지 않다.</dd>
<dt><code>ESRCH</code></dt>
<dd>

<code>rt_sigqueueinfo()</code>: <code>tgid</code>에 일치하는 스레드 그룹을 찾을 수 없다.

<code>rt_tgsigqueueinfo()</code>: <code>tgid</code>와 <code>tid</code>에 일치하는 스레드를 찾을 수 없다.
</dd>
</dl>

## VERSIONS

리눅스 버전 2.2에서 `rt_sigqueueinfo()` 시스템 호출이 추가되었다. 리눅스 버전 2.6.31에서 `rt_tgsigqueueinfo()` 시스템 호출이 추가되었다.

## CONFORMING TO

이 시스템 호출들은 리눅스 전용이다.

## NOTES

이 시스템 호출들은 응용에서 쓰기 위한 것이 아니므로 glibc 래퍼 함수가 없다. 그럴 일은 없겠지만 혹시라도 직접 호출하고 싶은 경우에는 <tt>[[syscall(2)]]</tt>을 이용하면 된다.

<tt>[[kill(2)]]</tt>에서처럼 널 시그널(0)을 사용해서 지정한 프로세스 내지 스레드가 존재하는지 확인할 수 있다.

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[tgkill(2)]]</tt>, <tt>[[pthread_sigqueue(3)]]</tt>, <tt>[[sigqueue(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2017-09-15
