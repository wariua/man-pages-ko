## NAME

sched_getcpu - 호출 스레드가 돌고 있는 CPU 알아내기

## SYNOPSIS

```c
#include <sched.h>

int sched_getcpu(void);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>sched_getcpu()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.14부터:</dt>
 <dd><code>_GNU_SOURCE</code></dd>
 <dt>glibc 2.14 전:</dt>
 <dd>
 <code>_BSD_SOURCE || _SVID_SOURCE</code><br>
 <code>    /* _GNU_SOURCE로도 충분함 */</code>
 </dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`sched_getcpu()`는 호출 스레드가 현재 실행되고 있는 CPU의 번호를 반환한다.

## RETURN VALUE

성공 시 `sched_getcpu()`는 음수 아닌 CPU 번호를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>ENOSYS</code></dt>
<dd>이 커널에서 <tt>[[getcpu(2)]]</tt>를 구현하고 있지 않다.</dd>
</dl>

## VERSIONS

glibc 2.6부터 이 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `sched_getcpu()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`sched_getcpu()`는 glibc 전용이다.

## NOTES

다음 호출은

```c
cpu = sched_getcpu();
```

다음의 <tt>[[getcpu(2)]]</tt> 호출과 동등하다.

```c
int c, s;
s = getcpu(&c, NULL, NULL);
cpu = (s == -1) ? s : c;
```

## SEE ALSO

<tt>[[getcpu(2)]]</tt>, <tt>[[sched(7)]]</tt>

----

2017-09-15
