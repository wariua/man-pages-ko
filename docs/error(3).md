## NAME

error, error_at_line, error_message_count, error_one_per_line, error_print_progname - glibc 오류 보고 함수

## SYNOPSIS

```c
#include <error.h>

void error(int status, int errnum, const char *format, ...);
void error_at_line(int status, int errnum, const char *filename,
                   unsigned int linenum, const char *format, ...);

extern unsigned int error_message_count;
extern int error_one_per_line;

extern void (*error_print_progname)(void);
```

## DESCRIPTION

`error()`는 범용 오류 보고 함수이다. `stdout`을 플러시하고 나서 `stderr`로 프로그램 이름, 콜론과 공백, <tt>[[printf(3)]]</tt> 방식 서식 문자열 `format`으로 지정한 메시지를 출력하며, `errnum`이 0이 아니면 두 번째 콜론과 공백, 그리고 `strerror(errnum)`에서 얻은 문자열을 함께 출력한다. `format`에 필요한 인자가 있으면 인자 목록에서 `format` 뒤에 따라와야 한다. 출력 내용 끝에는 개행 문자가 붙는다.

`error()`에서 찍는 프로그램 이름은 전역 변수 <tt>[[program_invocation_name(3)]]</tt>의 값이다. `program_invocation_name`은 처음에는 `main()`의 `argv[0]`과 같은 값을 가지고 있다. 이 변수의 값을 바꾸면 `error()` 출력도 바뀐다.

`status`가 0 아닌 값이면 `error()`에서 그 값을 종료 상태로 해서 <tt>[[exit(3)]]</tt>를 호출해서 프로그램을 종료시킨다. 아니면 오류 메시지를 찍고서 반환한다.

`error_at_line()` 함수는 `error()`와 동일하되 추가로 `filename` 및 `linenum` 인자가 있다. 내놓는 출력이 `error()`와 마찬가지이되 프로그램 이름 다음에 콜론, `filename`의 값, 콜론, `linenum`의 값이 들어간다. `error_at_line()` 호출 시에 전처리기의 값 `__LINE__`과 `__FILE__`이 유용하긴 하지만 다른 값도 쓸 수 있다. 예를 들어 이 인자들로 입력 파일 내의 위치를 나타낼 수도 있을 것이다.

전역 변수 `error_one_per_line`이 0 아닌 값으로 설정돼 있으면 같은 `filename` 및 `linenum` 값으로 연달아 `error_at_line()`을 호출하면 (처음의) 메시지 한 개만 찍히게 된다.

전역 변수 `error_message_count`는 `error()` 및 `error_at_line()`으로 지금껏 출력한 메시지 수를 센다.

전역 변수 `error_print_progname`에 함수 주소가 할당돼 있으면 (즉 NULL이 아니면) 메시지 앞의 프로그램 이름과 콜론을 출력하지 않고 대신 그 함수를 호출한다. 그러면 그 함수에서 `stderr`로 적당한 문자열을 찍어야 한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `error()` | 스레드 안전성 | MT-Safe locale |
| `error_at_line()` | 스레드 안전성 | MT-Unsafe race: error_at_line/error_one_per_line locale |

내부 `error_one_per_line` 변수에 접근이 이뤄진다. (어떤 동기화도 없지만 `int`를 한 번 쓰는 것이므로 충분히 안전할 것이다.) 그리고 `error_one_per_line`이 0 아닌 값으로 설정돼 있는 경우에는 마지막으로 찍은 파일 이름과 행 번호를 담는 (사용자에게 노출되지 않는) 정적 변수들에 동기화 없는 접근 및 변경이 이뤄진다. 갱신이 원자적이지 않으며 취소를 비활성화하기 전에 이뤄지기 때문에 두 변수 중 하나만 변경되고 실행이 중단될 수 있다. 그 외에는 `error_at_line()`이 `error()`와 거의 똑같다.

## CONFORMING TO

이 함수 및 변수들은 GNU 확장이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## SEE ALSO

<tt>[[err(3)]]</tt>, <tt>[[errno(3)]]</tt>, <tt>[[exit(3)]]</tt>, <tt>[[perror(3)]]</tt>, <tt>[[program_invocation_name(3)]]</tt>, <tt>[[strerror(3)]]</tt>

----

2021-03-22
