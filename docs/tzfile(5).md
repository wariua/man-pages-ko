## NAME

tzfile - 시간대 정보

## DESCRIPTION

<tt>[[tzset(3)]]</tt>에서 쓰는 시간대 정보 파일들은 보통 `/usr/share/zoneinfo` 같은 이름의 디렉터리 아래에 있다. 이 파일들은 인터넷 RFC 8536에서 설명하는 형식을 쓴다. 각 파일은 8비트 바이트들의 열이다. 파일 안에선 네트워크 순서(빅 엔디언, 상위 바이트 우선)의 한 개 이상 바이트의 열로 이진 정수를 표현하는데, 모든 비트를 사용하며, 부호 있는 이진 정수는 2의 보수로 표현한다. 불리언은 0(거짓) 아니면 1(참)인 한 바이트짜리 이진 정수로 표현한다. 그 형식은 다음 필드들을 담은 44바이트 헤더로 시작한다.

* 시간대 정보 파일임을 나타내는 4바이트짜리 ASCII 열 "TZif".

* 파일 형식 버전을 나타내는 한 바이트. (2017년 현재 ASCII NUL이거나, "2"이거나, "3".)

* 향후 용도를 위해 예비된 0을 담은 15바이트.

* 다음 순서로 4바이트짜리 정수 값 여섯 개.

    `tzh_ttisutcnt`
    :   파일에 저장된 UT/로컬 표시자 수. (UT는 Universal Time이다.)

    `tzh_ttisstdcnt`
    :   파일에 저장된 표준/벽시계 표시자 수.

    `tzh_leapcnt`
    :   파일에 데이터 항목이 저장돼 있는 윤초 수.

    `tzh_timecnt`
    :   파일에 데이터 항목이 저장돼 있는 전이(transition) 시간 수.

    `tzh_typecnt`
    :   파일에 데이터 항목이 저장돼 있는 지역 시간 종류 개수. (0이 아니어야 함.)

    `tzh_charcnt`
    :   파일에 저장된 시간대 축약명 문자열들의 바이트 수.

위 헤더 다음으로 다음 필드들이 오는데, 헤더 내용에 따라 길이가 정해진다.

* 오름차순으로 정렬된 4바이트 부호 있는 정수 값 `tzh_timecnt` 개. 이 값들은 네트워크 바이트 순서로 기록된다. 지역 시간 계산 규칙이 바뀌는 전이 시간으로 쓰인다. (<tt>[[time(2)]]</tt> 반환 값으로 나타냄.)

* 1바이트 부호 없는 정수 값 `tzh_timecnt` 개. 파일에 기술된 여러 지역 시간 종류들 중 어느 게 같은 인덱스의 전이 시간부터 다음 전이 시간 전까지의 기간에 연계돼 있는지 알려 준다. (단 마지막 시간 종류 항목은 아래 기술된 POSIX 스타일 TZ 문자열과의 일관성 검사를 위해 있을 뿐이다.) 이 값들이 다음 필드에서 인덱스 역할을 한다.

* 다음처럼 정의된 `ttinfo` 항목 `tzh_typecnt` 개.

        struct ttinfo {
             int32_t        tt_utoff;
             unsigned char  tt_isdst;
             unsigned char  tt_desigidx;
        };

    각 구조체는 네트워크 바이트 순서로 된 4바이트 부호 있는 정수 값 `tt_utoff`, 1바이트 불리언 값 `tt_isdst`, 1바이트 값 `tt_desigidx`로 이뤄진다. 각 구조체에서 `tt_utoff`는 UT에 더해야 할 초 수이고, `tt_isdst`는 <tt>[[localtime(3)]]</tt>에서 `tm_isdst`를 설정해야 하는지 여부를 나타내며, `tt_desigidx`는 파일에서 `ttinfo` 구조체(들) 다음에 오는 시간대 축약명 문자열 배열에서의 인덱스 역할을 한다. 32비트 클라이언트에서 오버플로 없이 부호를 바꿀 수 있도록 하기 위해 `tt_utoff` 값은 절대 -2\*\*31과 같지 않다. 그리고 현실적으로 쓰일 때 `tt_utoff`는 [-89999, 93599] 범위에 있다. (즉, -25시간보다 크고 26시간보다 작다.) 그래서 POSIX에서 요구하는 범위인 [-24:59:59, 25:59:59]를 이미 지원하는 구현체라면 쉽게 지원할 수 있다.

* 네트워크 바이트 순서로 된 4바이트 값 `tzh_leapcnt` 쌍. 각 쌍의 첫 번째 값은 윤초가 일어나는 음수 아닌 시간이고 (<tt>[[time(2)]]</tt> 반환 값으로 나타냄), 두 번째는 그 시간부터의 기간 동안에 적용할 윤초 *총* 개수를 나타내는 부호 있는 정수다. 시간 오름차순으로 쌍들이 저장돼 있다. 각 전이는 양수 또는 음수 1초 만큼의 윤초이며, 전이 간에는 항상 최소 28일 빼기 1초만큼의 간격을 둔다.

* 1바이트 불리언으로 저장된 표준/벽시계 표시자 `tzh_ttisdstcnt` 개. 지역 시간 종류와 연계된 전이 시간들이 표준 시간과 지역 (벽시계) 시간 중 어느 쪽으로 지정돼 있는지 나타낸다.

* 1바이트 불리언으로 저장된 UT/지역 표시자 `tzh_ttisutcnt` 개. 지역 시간 종류와 연계된 전이 시간들이 UT 시간과 지역 시간 중 어느 쪽으로 지정돼 있는지 나타낸다. UT/지역 표시자가 설정돼 있으면 대응하는 표준/벽시계 표시자도 설정돼 있어야 한다.

표준/벽시계 및 UT/지역 표시자는 TZif 파일의 전이 시간을 규칙이 결여된 POSIX 방식 TZ 문자열을 통해 지정된 다른 시간대에 적합한 전이들로 변환하기 위해 설계되었다. 예를 들어 TZ="EET-2EEST"이고 TZif 파일 "EET-2EEST"가 없는 경우에 미리 정해진 "posixrules"라는 TZif 파일에서 전이 시간을 가져다 조정한다는 것이었는데, 그 파일은 이 용도로만 존재하며 UT 오프셋이 다른 파일인 "Europe/Brussels" 파일의 사본이다. POSIX에서 이런 구식 변환 동작에 대해 명세하고 있지 않으며, 설치본마다 기본 규칙들이 다를 수 있다. 그리고 2037년 후 타임스탬프에 대해 이 기능을 지원하는 알려진 구현체가 전혀 없으므로 (가령) 그리스 시간을 원하는 사용자라면 폭넓은 하위 호환성이 중요하면 TZ="Europe/Athens"를 지정하면 되고, POSIX 준수가 필수고 다른 타임스탬프들을 정확하게 다룰 필요가 없으면 TZ="EET-2EEST,M3.5.0/3,M10.5.0/4"로 대처하면 된다.

`tzh_timecnt`가 0이거나 시간 인자가 파일에 기록돼 있는 첫 번째 전이 시간보다 앞인 경우에 <tt>[[localtime(3)]]</tt> 함수는 보통 파일의 첫 번째 `ttinfo` 구조체를 사용한다.

### 버전 2 형식

버전 2 형식의 시간대 파일에서는 위의 헤더 및 데이터 다음에 두 번째 헤더 및 데이터가 오는데, 전이 시간과 윤초 시간에 각각 8바이트를 쓴다는 점을 빼곤 동일한 형식이다. (윤초 개수는 여전히 4바이트다.) 두 번째 헤더 및 데이터 다음에는 개행으로 감싼 POSIX TZ 환경 변수 스타일 문자열이 오는데, 파일에 저장된 마지막 전이 시간 후의 순간들을 (파일에 전이가 없다면 모든 순간들을) 처리할 때 쓰기 위한 것이다. 그 순간들에 대한 POSIX 표현이 존재하지 않는 경우에는 POSIX 스타일 TZ 문자열이 비어 있다. (즉 개행 사이에 아무것도 없다.) POSIX 스타일 TZ 문자열이 비어 있지 않은 경우에는 그 여덟 바이트 데이터로 된 전이 시간이 있다면 마지막 전이 시간 후의 지역 시간 종류와 그 TZ 문자열이 부합해야 한다. 예를 들어 그 문자열이 "WET0WEST,M3.5.0,M10.5.0/3"라고 하고, 마지막 전이 시간이 7월에 있다면 그 전이의 지역 시간 종류에서 UT보다 한 시간 동쪽이고 "WEST"로 줄여 쓰는 일광 절약 시간을 명시해야 한다. 또한 전이가 최소 한 번은 있을 때 무한대 과거부터 가장 이른 전이 시간 전까지의 기간에는 시간 종류 0번이 연계된다.

### 버전 3 형식

버전 3 형식의 시간대 파일에서는 `newtzset(3)`에서 기술하는 POSIX TZ 형식에 대한 두 가지 소규모 확장을 그 POSIX TZ 스타일 문자열에서 쓸 수 있다. 첫째로, 전이 시간의 시간 부분이 POSIX에서 요구하는 0에서 24까지의 부호 없는 값이 아니라 -167에서 167까지 범위의 부호 있는 값일 수 있다. 둘째로, DST가 1월 1일 00:00에 시작해서 12월 31일 24:00에 일광 절약 시간과 표준시 차이를 더한 때에 끝나는 경우에는 DST가 연중 내내 효력이 있다.

### 연동 관련 사항

향후 형식이 바뀌면서 더 많은 데이터가 덧붙을 수 있다.

버전 1 파일은 구식 형식이며 피하는 게 좋다. 2038년 후의 전이 시간을 지원하지 않는다. 버전 1만 지원하는 읽기 프로그램에선 계산한 버전 1 데이터 블록 다음으로 이어지는 데이터를 모두 무시해야 한다.

쓰기 프로그램에선 전이 시간들을 정확히 명세하기 위해 TZ 문자열 확장이 필요하다면 버전 3 파일을 생성하는 게 좋다. 그게 아니라면 버전 2 파일을 생성하는 게 좋다.

버전 1 헤더와 데이터 블록에서 규정하는 시간 변화들의 열이 버전 2+ 헤더와 데이터 블록, 그리고 마지막 부분에서 규정하는 시간 변화들의 연속 부분열이게 하는 게 좋다. 이렇게 하면 그 연속 부분열 내의 타임스탬프들에 대해 구식 버전 1 읽기 프로그램과 최신 읽기 프로그램의 의견이 일치할 수 있게 된다. 또한 구식 읽기 프로그램을 지원하지 않으려는 쓰기 프로그램에서 버전 1 데이터 블록의 `tzh_timecnt`에 0을 써서 공간을 절약할 수 있게 된다.

시간대 명칭을 최소 세(3) 글자에 여섯(6) 글자를 넘지 않는 ASCII 문자로 하고, 영자와 숫자, "-", "+"만 쓰는 게 좋다. 시간대 축약명에 대한 POSIX 요구 사항과의 호환성을 위한 것이다.

버전 2 또는 3인 파일을 읽을 때 읽기 프로그램은 건너뛰는 목적 말고는 버전 1 헤더와 데이터 블록 내용을 무시하는 게 좋다.

읽기 프로그램에서 파일 유효성을 검사할 때 헤더와 데이터 블록들의 총 길이를 계산해서 그게 모두 실제 파일 크기 안에 들어가는지 확인해 보는 게 좋다.

### 흔한 연동 이슈

이 절에선 TZif 파일을 읽고 쓸 때의 흔한 문제들을 설명한다. 이 문제들 대부분은 구식 읽기 프로그램에서 이용할 TZif 파일을 생성할 때의 문제이다. 이 절의 목표는 이렇다.

* TZif 쓰기 프로그램이 파일을 출력할 때 구식이거나 버그 있는 TZif 읽기 프로그램의 흔한 문제들을 피할 수 있게 돕는다.

* TZif 읽기 프로그램이 미래의 TZif 쓰기 프로그램에서 생성한 파일을 읽을 때 흔한 문제들을 피할 수 있게 돕는다.

* TZif 형식이 바뀔 때 어떤 종류의 문제가 발생했는지를 미래의 명세 작성자가 알 수 있게 돕는다.

새로운 TZif 형식들이 정의되었을 때의 설계 목표 하나는 읽기 프로그램의 설계보다 파일이 이후 TZif 버전이더라도 읽기 프로그램이 TZif 파일을 성공적으로 쓸 수 있게 하는 것이었다. 완벽한 호환성을 달성할 수 없는 경우에는 문제들을 잘 안 쓰는 타임스탬프들로 제한하려는, 그리고 쓰기 프로그램에서의 간단하고 불완전한 해결책들을 통해 구식 버전 읽기 프로그램에서도 이용 가능한 새 버전 데이터를 생성할 수 있게 하려는 시도가 이뤄졌다. 그런 호환성 문제들과 해결책, 그리고 읽기 프로그램들에 흔한 여타 버그들을 이 절에 기록해 둔다.

다음은 몇 가지 TZif 관련 호환성 문제들이다.

* 어떤 읽기 프로그램은 버전 1 데이터만 확인한다. 불완전한 해결책은 쓰기 프로그램에서 가능하면 많이 버전 1 데이터를 출력하는 것이다. 그렇지만 읽기 프로그램에서 설령 자체 타임스탬프가 32비트짜리더라도 버전 1 데이터는 무시하고 버전 2+ 데이터를 쓰는 편이 좋다.

* 버전 2에 맞게 설계된 어떤 읽기 프로그램이 TZ 스타일 문자열의 POSIX 확장을 해석할 수 없어서 버전 3 파일의 마지막 전이 뒤의 타임스탬프를 잘못 처리할 수도 있다. 불완전한 해결책은 쓰기 프로그램에서 필요한 것보다 많은 전이를 출력해서 버전 2 읽기 프로그램에서 먼 미래의 타임스탬프만 잘못 처리하게 하는 것이다.

* 버전 2에 맞게 설계된 어떤 읽기 프로그램은 영구 일광 절약 시간을 (가령 영구 동부 일광 절약 시간(-04)을 나타내는 TZ 문자열 "EST5EDT,0/0,J365/25"를) 지원하지 않는다. 불완전한 해결책은 쓰기 프로그램에서 동쪽 다음 시간대의 표준시로 (가령 영구적 대서양 표준시(-04)인 "AST4"로) 대체하는 것이다.

* 어떤 읽기 프로그램은 마지막 부분을 무시하고 대신 마지막 전이의 시간 종류를 가지고 미래 타임스탬프를 추측한다. 불완전한 해결책은 쓰기 프로그램에서 필요한 것보다 많은 전이를 출력하는 것이다.

* 어떤 읽기 프로그램은 첫 번째 전이 전의 타임스탬프에 시간 종류 0번을 쓰지 않는다. 경험적 규칙에 따라 시간 종류를 추정하는데, 그래서 항상 시간 종류 0번을 선택하지는 않기 때문이다. 불완전한 해결책은 쓰기 프로그램에서 이른 시간에 아무것도 하지 않는 첫번째 전이를 출력하는 것이다.

* 어떤 읽기 프로그램은 타임스탬프가 -2\*\*31보다 작지 않은 첫 번째 전이 전의 타임스탬프를 잘못 처리한다. 특히 32비트 타임스탬프만 지원하는 읽기 프로그램에서, 예를 들어 32비트로는 일부만 표현 가능한 64비트 전이를 처리할 때 이 문제가 발생하기 쉽다. 불완전한 해결책은 쓰기 프로그램에서 아무것도 하지 않는 전이를 타임스탬프 -2\*\*31에 출력하는 것이다.

* 어떤 읽기 프로그램은 타임스탬프가 가장 작은 부호 있는 64비트 값인 전이를 잘못 처리한다. -2\*\*59보다 작은 타임스탬프는 권장하지 않는다.

* 어떤 읽기 프로그램은 "<"나 ">"가 들어간 POSIX 방식 TZ 문자열을 잘못 처리할 수 있다. 불완전한 해결책은 쓰기 프로그램에서 알파벳 문자만 들어간 시간대 축약명에 "<"나 ">" 사용을 피하는 것이다.

* 많은 읽기 프로그램은 ASCII 아닌 문자가 들어간 시간대 축약명을 잘못 처리한다. 그런 문자들은 권장하지 않는다.

* 어떤 읽기 프로그램은 3글자보다 짧거나 6글자보다 길거나 영자와 숫자, "-", "+"가 아닌 ASCII 문자가 들어간 시간대 축약명을 잘못 처리할 수 있다. 그런 축약명은 권장하지 않는다.

* 어떤 읽기 프로그램은 일괄 절약 시간 UT 오프셋이 대응하는 표준 시간의 UT 오프셋보다 작게 지정돼 있는 TZif 파일을 잘못 처리한다. 그런 읽기 프로그램은 POSIX TZ 문자열 "IST-1GMT0,M10.5.0,M3.5.0/1"과 동등한 방식을 쓰는 아일랜드 같은 곳을 지원하지 못하는데, 거기선 여름에 표준시(IST, +01)를 따르고 겨울에 일광 절약시(GMT, +00)를 따른다. 불완전한 해결책은 쓰기 프로그램에서 POSIX TZ 문자열 "GMT0IST,M3.0.0/1,M10.5.0"과 동등한 방식을 출력해서 표준시와 일광 절약시를 뒤바꾸는 것이다. 이 해결책은 어느 때 일광 절약 시간을 쓰는지를 잘못 표시하지만 UT 오프셋들과 시간대 축약명은 올바로 적고 있다.

연동 문제 중 일부는 읽기 프로그램의 버그이며, 다음은 읽기 프로그램 개발자를 위한 주의 사항이라 할 수 있다.

* 어떤 읽기 프로그램은 음수 타임스탬프를 지원하지 않는다. 배포하는 응용을 개발하는 사람은 1970년 전 데이터도 다룰 필요가 있다는 점을 유념할 필요가 있다.

* 어떤 읽기 프로그램은 음수 아닌 타임스탬프의 첫 번째 전이 전의 타임스탬프를 잘못 처리한다. 특히 음수 타임스탬프를 지원하지 않는 읽기 프로그램에서 이 문제가 발생하기 쉽다.

* 어떤 읽기 프로그램은 "-08"처럼 "+"나 "-", 숫자를 담은 시간대 축약명을 잘못 처리한다.

* 어떤 읽기 프로그램은 전통적인 -12시간에서 +12시간 범위에서 벗어난 UT 오프셋을 잘못 처리한다. 그래서 그 범위 밖에 있는 키리티마티 같은 곳을 지원하지 못한다.

* 어떤 읽기 프로그램은 UT 기준 [-3599, -1] 초 범위의 UT 오프셋을 잘못 처리하는데, 오프셋을 3600으로 정수 나눗셈 해서 0이 나오면 시간 부분을 "+00"로 표시한다.

* 어떤 읽기 프로그램은 1시간, 또는 15분, 또는 1분의 배수가 아닌 UT 오프셋을 잘못 처리한다.

## SEE ALSO

<tt>[[time(2)]]</tt>, <tt>[[localtime(3)]]</tt>, <tt>[[tzset(3)]]</tt>, `tzselect(8)`, `zdump(8)`, `zic(8)`.

Olson A, Eggert P, Murchison K. The Time Zone Information Format (TZif). 2019 Feb. Internet RFC 8536 <https://www.rfc-editor.org/info/rfc8536> doi:10.17487/RFC8536 <https://doi.org/10.17487/RFC8536>.

----

2020-04-27