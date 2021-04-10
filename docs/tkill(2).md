## NAME

tkill, tgkill - 스레드로 시그널 보내기

## SYNOPSIS

```c
int tkill(int tid, int sig);

int tgkill(int tgid, int tid, int sig);
```

*주의*: `tkill()`에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`tgkill()`은 스레드 그룹 `tgid` 안에 있는 스레드 ID가 `tid`인 스레드에게 시그널 `sig`를 보낸다. (반면 프로세스(즉 스레드 그룹) 전체에게 시그널을 보내려면 <tt>[[kill(2)]]</tt>을 쓸 수 있으며, 그러면 그 프로세스 내의 임의 스레드에게 시그널이 전달된다.)

`tkill()`은 `tgkill()`의 구식 선조이다. 대상 스레드 ID만 지정할 수 있으며, 그래서 스레드가 종료되어 그 스레드 ID가 재활용되는 경우 잘못된 스레드에게 시그널을 보낼 수도 있다. 이 시스템 호출 사용을 피해야 한다.

가공 안 된 시스템 호출 인터페이스이며 내부 스레드 라이브러리에서의 사용을 위한 것이다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`EINVAL`
:   지정한 스레드 ID나 스레드 그룹 ID, 시그널이 유효하지 않다.

`EPERM`
:   권한 거부. 필요한 권한에 대해선 <tt>[[kill(2)]]</tt>을 보라.

`ESRCH`
:   지정한 스레드 ID를 (스레드 그룹 ID를) 가진 프로세스가 존재하지 않는다.

`EAGAIN`
:   `RLIMIT_SIGPENDING` 자원 한계에 도달했고 `sig`가 실시간 시그널이다.

`EAGAIN`
:   사용 가능한 커널 메모리가 불충분했고 `sig`가 실시간 시그널이다.

## VERSIONS

리눅스 2.4.19 / 2.5.4부터 `tkill()`을 지원했다. 리눅스 2.5.75에서 `tgkill()`이 추가되었다.

glibc 버전 2.30에서 `tgkill()`에 대한 라이브러리 지원이 추가되었다.

## CONFORMING TO

`tkill()`과 `tgkill()`은 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

스레드 그룹에 대한 설명은 <tt>[[clone(2)]]</tt>의 `CLONE_THREAD` 설명을 보라.

glibc에서 `tkill()`의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다. glibc 2.30 전에선 `tgkill()`에도 래퍼 함수가 없었다.

## SEE ALSO

<tt>[[clone(2)]]</tt>, <tt>[[gettid(2)]]</tt>, <tt>[[kill(2)]]</tt>, <tt>[[rt_sigqueueinfo(2)]]</tt>

----

2019-08-02
