## NAME

libc - 리눅스의 표준 C 라이브러리들 소개

## DESCRIPTION

흔히 "표준 C 라이브러리"의 줄임말로 "libc"라는 용어를 쓴다. 표준 C 라이브러리는 모든 C 프로그램에서 (또 때로는 다른 언어로 된 프로그램에서) 사용할 수 있는 표준 함수들의 라이브러리다. 어떤 역사적 이유 때문에 (아래 참고) 리눅스에서 "libc"라는 말로 표준 C 라이브러리를 가리킬 때는 약간의 모호함이 있다.

### glibc

리눅스에서 단연 가장 널리 쓰이는 C 라이브러리는 GNU C 라이브러리(http://www.gnu.org/software/libc/)이며 *glibc*라고 많이 부른다. 요즘 모든 주요 리눅스 배포판에서 쓰는 C 라이브러리이며, 그 세부 내용들이 *man-pages* 프로젝트의 관련 페이지들에 (주로 매뉴얼의 섹션 3에) 기록돼 있는 C 라이브러리이기도 하다. glibc 매뉴얼을 통해서도 glibc의 문서를 볼 수 있는데, `info libc` 명령으로 가능하다. glibc 1.0이 1992년 9월에 출시되었다. (그 전에 0.x 출시가 있었다.) glibc의 다음 큰 버전인 2.0이 1997년 초에 출시되었다.

보통은 경로명 `/lib/libc.so.6`이 (또는 비슷한 뭔가가) glibc 라이브러리 위치를 가리키는 심볼릭 링크이다. 그 경로명을 실행하면 glibc가 시스템에 설치된 버전에 대한 여러 정보를 표시하게 된다.

### Linux libc

1990년 초중반 얼마 동안 *Linux libc*가 있었는데, 당시 glibc 개발이 리눅스에서의 요구를 충족시키지 못한다고 느낀 리눅스 개발자들이 만든 glibc 1.x 포크였다. 이 라이브러리를 (혼란스럽게도) 그냥 "libc"라고 부르는 경우가 많았다. Linux libc는 큰 버전으로 2, 3, 4, 5를 출시했고 작은 버전들도 여럿 출시했다. Linux libc 4는 a.out 바이너리 형식을 쓰는 마지막 버전이자 (원시적인) 공유 라이브러리 지원을 제공한 첫 번째 버전이었다. Linux libc 5는 ELF 바이너리 형식을 지원하는 첫 번째 버전이었다. 이 버전에서 공유 라이브러리 soname으로 `libc.so.5`를 썼다. 한동안은 Linux libc가 여러 리눅스 배포판의 표준 C 라이브러리였다.

하지만 Linux libc 활동의 원래 동기에도 불구하고 (1997년에) glibc 2.0이 출시됐을 때 그게 Linux libc보다 낫다는 게 분명했고, Linux libc를 쓰던 주요 리눅스 배포판들이 모두 이내 glibc로 되돌아갔다. Linux libc 버전들과의 혼동을 피하기 위해 glibc 2.0 및 이후에서는 공유 라이브러리 soname으로 `libc.so.6`을 사용했다.

오래전 일어난 Linux libc에서 glibc 2.0으로의 전환 이후로 *man-pages*에서는 더 이상 Linux libc에 대한 내용을 신경써서 기록하지 않는다. 그럼에도 불구하고 몇몇 매뉴얼 페이지에 *libc4* 및 *libc5*를 언급하는 형태로 남아있는 Linux libc에 대한 정보들의 흔적을 볼 수 있다.

### 기타 C 라이브러리들

리눅스에는 덜 흔하게 쓰이는 다른 C 라이브러리들이 여러 가지 있다. 그 라이브러리들은 일반적으로 제공 기능과 메모리 사용량 모두가 glibc보다 작으며, 임베디드 리눅스 시스템 개발 등을 위해 작은 바이너리를 만드는 데 쓰는 경우가 많다. 그런 라이브러리들로 *uClibc* (http://www.uclibc.org/), *dietlibc* (http://www.fefe.de/dietlibc/), *musl libc* (http://www.musl-libc.org/) 등이 있다. 그 라이브러리들의 세부 내용은 그쪽 *man-pages* 프로젝트에서 다룬다.

## SEE ALSO

<tt>[[syscalls(2)]]</tt>, <tt>[[getauxval(3)]]</tt>, <tt>[[proc(5)]]</tt>, <tt>[[feature_test_macros(7)]]</tt>, `man-pages(7)`, <tt>[[standards(7)]]</tt>, <tt>[[vdso(7)]]</tt>

----

2016-12-12
