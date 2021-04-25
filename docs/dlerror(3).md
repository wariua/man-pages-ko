## NAME

dlerror - dlopen API의 함수들에 대한 오류 진단 얻기

## SYNOPSIS

```c
#include <dlfcn.h>

char *dlerror(void);
```

`-ldl`로 링크.

## DESCRIPTION

`dlerror()` 함수는 사람이 읽을 수 있는 널 종료 문자열을 반환하는데, 그 문자열에는 가장 최근 `dlerror()` 호출 이후로 dlopen API의 함수 호출에서 발생한 가장 최근의 오류가 기술돼 있다. 반환되는 문자열 끝에 개행이 포함돼 있지 *않다*.

초기화 이후로 또는 마지막 호출 이후로 어떤 오류도 발생하지 않았으면 `dlerror()`가 NULL을 반환한다.

## VERSIONS

glibc 2.0 및 이후에 `dlerror()`가 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `dlerror()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001.

## NOTES

`dlerror()`에서 반환하는 메시지가 정적 할당 버퍼에 있어서 다음 `dlerror()` 호출에 의해 덮어 써질 수도 있다.

### 역사

이 함수가 포함된 dlopen API는 SunOS에서 유래한 것이다.

## EXAMPLES

<tt>[[dlopen(3)]]</tt> 참고.

## SEE ALSO

<tt>[[dladdr(3)]]</tt>, <tt>[[dlinfo(3)]]</tt>, <tt>[[dlopen(3)]]</tt>, <tt>[[dlsym(3)]]</tt>

----

2021-03-22
