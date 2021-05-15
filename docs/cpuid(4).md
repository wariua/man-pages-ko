## NAME

cpuid - x86 CPUID 접근 장치

## DESCRIPTION

CPUID는 x86 CPU에 대한 정보 질의를 위한 인터페이스를 제공한다.

이 장치에 <tt>[[lseek(2)]]</tt>이나 <tt>[[pread(2)]]</tt>로 접근해서 적절한 CPUID 레벨을 선택하고 16바이트 단위로 읽어 들인다. 16바이트 넘게 읽는 것은 연속한 여러 레벨을 읽는 것이다.

파일 위치의 하위 32비트를 입력 `%eax`로 쓰고 파일 위치의 상위 32비트를 입력 `%ecx`로 쓴다. 후자는 `eax=4` 같은 때 `eax` 레벨을 나타내기 위한 것이다.

이 드라이버에서 `/dev/cpu/CPUNUM/cpuid` 파일을 사용하는데, 여기서 `CPUNUM`은 부번호다. SMP 박스에서 `/proc/cpuinfo`에 나열된 대로 `CPUNUM`번 CPU로 접근하게 된다.

이 파일은 사용자 `root`나 그룹 `root` 구성원만 읽을 수 있게 보호된다.

## NOTES

인라인 어셈블러를 이용해 프로그램에서 CPUID 인스트럭션을 직접 실행할 수도 있다. 하지만 이 장치를 쓰면 프로세스 친화성을 바꾸지 않고도 편하게 모든 CPU에 접근할 수 있다.

`cpuid`의 정보 대부분을 커널이 `/proc/cpuinfo`나 `/sys/devices/system/cpu` 하위 디렉터리에서 가공된 형태로 알려준다. 이 장치를 통한 CPUID 직접 접근은 예외적인 경우에만 쓰게 될 것이다.

`cpuid` 드라이버는 자동으로 적재되지 않는다. 모듈을 쓰는 커널에선 사용 전에 다음 명령으로 모듈을 따로 적재해야 할 수도 있다.

```text
$ modprobe cpuid
```

더 많은 입력 레지스터를 필요로 하는 CPUID 기능은 지원하지 않는다.

아주 오래된 x86 CPU들은 CPUID를 지원하지 않는다.

## SEE ALSO

`cpuid(1)`

Intel Corporation, Intel 64 and IA-32 Architectures Software Developer's Manual Volume 2A: Instruction Set Reference, A-M, 3-180 CPUID reference.

Intel Corporation, Intel Processor Identification and the CPUID Instruction, Application note 485.

----

2019-08-02
