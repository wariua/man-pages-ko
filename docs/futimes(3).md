## NAME

futimes, lutimes - 파일 타임스탬프 바꾸기

## SYNOPSIS

```c
#include <sys/time.h>

int futimes(int fd, const struct timeval tv[2]);

int lutimes(const char *filename, const struct timeval tv[2]);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>futimes()</code>, <code>lutimes()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.19부터:</dt>
 <dd><code>_DEFAULT_SOURCE</code></dd>
 <dt>glibc 2.19 및 이전:</dt>
 <dd><code>_BSD_SOURCE</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`futimes()`는 <tt>[[utimes(2)]]</tt>와 같은 방식으로 파일의 접근 시간과 수정 시간을 바꾼다. 차이는 경로명이 아니라 파일 디스크립터를 통해 타임스탬프를 바꿀 파일을 지정한다는 점이다.

`lutimes()`는 <tt>[[utimes(2)]]</tt>와 같은 방식으로 파일의 접근 시간과 수정 시간을 바꾼다. 차이는 `filename`이 심볼릭 링크를 가리키는 경우 그 링크를 역참조하지 않는다는 점이다. 대신 그 심볼릭 링크의 타임스탬프를 바꾼다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

오류는 <tt>[[utimes(2)]]</tt>와 마찬가지이되, `futimes()`에 다음이 추가된다.

<dl>
<dt><code>EBADF</code></dt>
<dd><code>fd</code>가 유효한 파일 디스크립터가 아니다.</dd>
<dt><code>ENOSYS</code></dt>
<dd><code>/proc</code> 파일 시스템에 접근할 수 없다.</dd>
</dl>

`lutimes()`에서는 다음 오류가 추가로 발생할 수 있다.

<dl>
<dt><code>ENOSYS</code><dt>
<dd>커널이 이 호출을 지원하지 않는다. 리눅스 2.6.22나 이후 버전이 필요하다.</dd>
</dl>

## VERSIONS

glibc 2.3부터 `futimes()`가 사용 가능하다. glibc 2.6부터 `lutimes()`가 사용 가능한데, 커널 2.6.22부터 지원하는 <tt>[[utimensat(2)]]</tt> 시스템 호출을 이용해 구현되어 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `futimes()`, `lutimes()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 어떤 표준에도 명세되어 있지 않다. 리눅스를 제외하면 BSD 계열에서만 사용 가능하다.

## SEE ALSO

<tt>[[utime(2)]]</tt>, <tt>[[utimensat(2)]]</tt>, <tt>[[symlink(7)]]</tt>

----

2017-09-15
