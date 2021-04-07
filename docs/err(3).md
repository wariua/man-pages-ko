## NAME

err, verr, errx, verrx, warn, vwarn, warnx, vwarnx - 형식 있는 오류 메시지

## SYNOPSIS

```c
#include <err.h>

void err(int eval, const char *fmt, ...);

void errx(int eval, const char *fmt, ...);

void warn(const char *fmt, ...);

void warnx(const char *fmt, ...);

#include <stdarg.h>

void verr(int eval, const char *fmt, va_list args);

void verrx(int eval, const char *fmt, va_list args);

void vwarn(const char *fmt, va_list args);

void vwarnx(const char *fmt, va_list args);
```

## DESCRIPTION

`err()` 및 `warn()` 계열 함수들은 표준 오류 출력으로 서식 준 오류 메시지를 표시한다. 모든 경우에 프로그램 이름의 마지막 부분, 콜론 문자, 공백을 출력한다. `fmt` 인자가 NULL이 아니면 <tt>[[printf(3)]]</tt> 방식으로 서식 준 오류 메시지를 출력한다. 출력 내용 끝에는 개행 문자가 붙는다.

`err()`, `verr()`, `warn()`, `vwarn()` 함수는 전역 변수 `errno`로 <tt>[[strerror(3)]]</tt>에서 얻은 오류 메시지를 덧붙이는데, `fmt` 인자가 NULL이 아니면 사이에 콜론과 공백을 붙인다.

`errx()` 및 `warnx()` 함수는 오류 메시지를 덧붙이지 않는다.

`err()`, `verr()`, `errx()`, `verrx()` 함수는 반환하지 않고 `eval` 인자 값으로 프로그램을 끝낸다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `err()`, `errx()`,<br>`warn()`, `warnx()`,<br>`verr()`, `verrx()`,<br>`vwarn()`, `vwarnx()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

이 함수들은 비표준 BSD 확장이다.

## EXAMPLE

현재 `errno` 정보 문자열을 표시하고 끝내기:

```c
p = malloc(size);
if (p == NULL)
    err(1, NULL);
fd = open(file_name, O_RDONLY, 0);
if (fd == -1)
    err(1, "%s", file_name);
```

오류 메시지 표시하고 끝내기:

```c
if (tm.tm_hour < START_TIME)
    errx(1, "too early, wait until %s", start_time_string);
```

문제 경고하기:

```c
fd = open(raw_device, O_RDONLY, 0);
if (fd == -1)
    warnx("%s: %s: trying the block device",
            raw_device, strerror(errno));
fd = open(block_device, O_RDONLY, 0);
if (fd == -1)
    err(1, "%s", block_device);
```

## SEE ALSO

<tt>[[error(3)]]</tt>, <tt>[[exit(3)]]</tt>, <tt>[[perror(3)]]</tt>, <tt>[[printf(3)]]</tt>, <tt>[[strerror(3)]]</tt>

----

2017-09-15
