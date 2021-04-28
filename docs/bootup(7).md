## NAME

bootup - 시스템 부팅 과정

## DESCRIPTION

리눅스 시스템 부팅에는 여러 다양한 요소들이 관련돼 있다. 시스템 전원이 들어온 직후 시스템 펌웨어에서 최소한의 하드웨어 초기화를 하고서 영속 저장 장치에 저장된 부트 로더(가령 `systemd-boot(7)`나 **GRUB**[1])로 제어를 넘기게 된다. 그러면 부트 로더가 디스크의 (또는 네트워크에서 가져온) OS 커널을 부르게 된다. EFI나 여타 종류의 펌웨어를 쓰는 시스템에서는 그 펌웨어가 직접 커널을 적재할 수도 있다.

커널에서 (선택적으로) 인메모리 파일 시스템을 마운트한다. `dracut(8)`으로도 많이 만드는 그 파일 시스템에서 루트 파일 시스템을 찾는다. 요즘은 일반적으로 initramfs를 쓰는데, 커널이 부팅할 때 tmpfs 기반의 경량 인메모리 파일 시스템에 압축 아카이브를 푼다. 과거에는 인메모리 블록 장치(ramdisk)를 쓰는 보통 파일 시스템을 사용했으며, 그래서 지금도 두 개념을 설명할 때 "initrd"라는 이름을 쓴다. 커널과 initrd/initramfs 이미지를 메모리로 적재하는 건 부트 로더나 펌웨어지만 그걸 파일 시스템으로 해석하는 건 커널이다. `systemd(1)`를 이용해 실제 시스템과 비슷한 방식으로 initrd 안의 서비스들을 관리할 수도 있다.

루트 파일 시스템을 찾아서 마운트한 다음 initrd는 루트 파일 시스템에 저장된 (`systemd(1)` 같은) 호스트의 시스템 관리자로 제어를 넘긴다. 그러면 시스템 관리자가 나머지 하드웨어들을 모두 조사하고, 필요한 파일 시스템들을 모두 마운트하고, 설정된 서비스들을 모두 띄우는 역할을 맡는다.

시스템 셧다운 시 시스템 관리자는 서비스들을 모두 정지하고, 파일 시스템들을 모두 언마운트하고 (기반 저장 장치를 분리하고), (선택적으로) 루트 파일 시스템과 그 저장소를 언마운트/분리하는 initrd 코드로 되돌아간다. 마지막 단계로 시스템 전원이 꺼진다.

시스템 부트 과정에 대한 추가 내용을 <tt>[[boot(7)]]</tt>에서 볼 수 있다.

## 시스템 관리자 부팅

부팅 때 OS 이미지의 시스템 관리자는 시스템 동작에 필요한 필수 파일 시스템과 서비스, 드라이버의 초기화를 맡는다. `systemd(1)` 체계에서 이 과정은 여러 분리된 단계들로 나눠져 있으며 타깃 유닛 형태로 드러나게 된다. (타깃 유닛에 대한 자세한 정보는 `systemd.target(5)`을 보라.) 부팅 과정이 고도로 병렬화되어 있기 때문에 특정 타깃 유닛에 도달하는 순서가 확정적이지 않지만 그래도 정해진 순서 구조를 충실히 따른다.

systemd은 시스템을 시작할 때 default.target이 의존하는 모든 유닛들을 (그리고 그 유닛이 의존하는 것들까지 재귀적으로) 활성화한다. 일반적으로 default.target은 graphical.target이나 multi-user.target의 별칭일 뿐이며, 어느 쪽인지는 시스템이 그래픽 UI와 텍스트 콘솔 중 어느 쪽을 쓰게 구성되어 있는지에 따라 정해진다. 당겨 오는 유닛들 간에 적용되는 순서 관계를 단순화하기 위해 여러 가지 잘 알려진 타깃 유닛들이 있는데 `systemd.special(7)`에서 볼 수 있다.

다음 도표는 그 잘 알려진 유닛들과 부팅 로직에서의 위치를 개괄적으로 보여 준다. 화살표는 어떤 유닛이 당겨지는, 즉 다른 유닛에 선행하는 유닛이라는 것을 보여 준다. 도표 상단에 가까운 유닛이 아래에 가까운 유닛보다 먼저 시작된다.

```text
                                      cryptsetup-pre.target
                                                   |
(다양한 저수준                                     v
 API VFS 마운트:                   (다양한 cryptsetup 장치...)
 mqueue, configfs,                                 |    |
 debugfs, ...)                                     v    |
  |                                  cryptsetup.target  |
  |  (다양한 스왑                                  |    |    remote-fs-pre.target
  |   장치...)                                     |    |     |        |
  |    |                                           |    |     |        v
  |    v                       local-fs-pre.target |    |     |  (네트워크 파일 시스템)
  |  swap.target                       |           |    v     v                 |
  |    |                               v           |  remote-cryptsetup.target  |
  |    |  (다양한 저수준      (다양한 마운트 및    |             |              |
  |    |   서비스: udevd,      fsck 서비스...)     |             |    remote-fs.target
  |    |   tmpfiles, random            |           |             |             /
  |    |   seed, sysctl, ...)          v           |             |            /
  |    |      |                 local-fs.target    |             |           /
  |    |      |                        |           |             |          /
  \____|______|_______________   ______|___________/             |         /
                              \ /                                |        /
                               v                                 |       /
                        sysinit.target                           |      /
                               |                                 |     /
        ______________________/|\_____________________           |    /
       /              |        |      |               \          |   /
       |              |        |      |               |          |  /
       v              v        |      v               |          | /
   (다양한        (다양한      |  (다양한             |          |/
    타이머...)     경로...)    |   소켓...)           |          |
       |              |        |      |               |          |
       v              v        |      v               |          |
 timers.target  paths.target   |  sockets.target      |          |
       |              |        |      |               v          |
       v              \_______ | _____/         rescue.service   |
                              \|/                     |          |
                               v                      v          |
                           basic.target        *rescue.target*   |
                               |                                 |
                       ________v____________________             |
                      /              |              \            |
                      |              |              |            |
                      v              v              v            |
                  display-     (그래픽 UI에    (다양한 시스템    |
              manager.service     필요한          서비스)        |
                      |         다양한 시스템       |            |
                      |           서비스)           v            v
                      |              |           *multi-user.target*
 emergency.service    |              |              |
         |            \_____________ | _____________/
         v                          \|/
*emergency.target*                   v
                             *graphical.target*
```

부트 타깃으로 흔히 쓰는 타깃 유닛들이 \*강조\*되어 있다. 이 유닛들은 목표 타깃으로 삼기에 적당한데, 예를 들어 커널 명령행 옵션 `systemd.unit=`(`systemd(1)` 참고)에 주거나 심볼릭 링크로 default.target이 그 유닛을 가리키게 할 수 있다.

timers.target은 basic.target에서 비동기적으로 당겨 온다. 그래서 타이머 유닛들에서 부팅 후반에야 사용 가능한 서비스에 의존할 수 있다.

## 사용자 관리자 시작

시스템 관리자에서 각 사용자마다 user@*uid*.service 유닛을 시작해서 사용자별로 비특권 `systemd` 인스턴스를 따로 띄우는데, 이것이 사용자 관리자다. 시스템 관리자와 마찬가지로 사용자 관리자는 default.target이 당겨 오는 유닛들을 시작한다. 다음 도표는 잘 알려진 사용자 유닛들을 개괄적으로 보여 준다. 비그래픽 세션에는 default.target을 쓴다. 사용자가 그래픽 세션에 로그인할 때마다 로그인 관리자가 graphical-session.target 타깃을 실행하며, 이를 이용해 그래픽 세션에 필요한 유닛들을 당겨 온다. 사용자가 특정 하드웨어를 이용할 수 있을 때 (오른편에 있는) 여러 타깃들이 시작된다.

```text
    (다양한            (다양한           (다양한
     타이머...)         경로...)          소켓...)     (사운드 장치들)
        |                  |                 |               |
        v                  v                 v               v
  timers.target      paths.target     sockets.target    sound.target
        |                  |                 |
        \______________   _|_________________/         (블루투스 장치들)
                       \ /                                   |
                        V                                    v
                  basic.target                          bluetooth.target
                        |
             __________/ \_______                      (스마트카드 장치들)
            /                    \                           |
            |                    |                           v
            |                    v                      smartcard.target
            v            graphical-session-pre.target
(다양한 사용자 서비스들)         |                       (프린터들)
            |                    v                           |
            |            (그래픽 세션을 위한 서비스들)       v
            |                    |                       printer.target
            v                    v
    *default.target*     graphical-session.target
```

## 최초 램디스크(INITRD)에서의 부팅

최초 램디스크(initial RAM disk) 구현(initrd)도 systemd로 구성할 수 있다. 이 경우 initrd 내의 부팅은 다음 구조를 따른다.

systemd에서 /etc/initrd-release 파일을 확인해서 initrd에서 돌고 있다는 걸 알아낸다. initrd에서 기본 타깃은 initrd.target이다. basic.target에 도달할 때까지는 시스템 관리자 부팅(위 내용 참고)과 동일하게 부팅 과정이 시작된다. 거기서부터 systemd는 특별한 타깃인 initrd.target으로 접근한다. 파일 시스템을 마운트하기 전에 시스템이 하이버네이션에서 복귀하는 것인지 아니면 정상 부트를 진행할 것인지 결정해야 한다. 이 결정은 systemd-hibernate-resume@.service에서 이뤄지고 local-fs-pre.target 전에 완료되어야 하며, 그래서 확인이 끝나기 전에는 어떤 파일 시스템도 마운트할 수 없다. 루트 장치가 사용 가능해지면 initrd-root-device.target에 도달한다. 그 루트 장치를 /sysroot에 마운트할 수 있으면 sysroot.mount 유닛이 활성화되고 initrd-root-fs.target에 도달한다. initrd-parse-etc.service 서비스에선 가능한 /usr 마운트와 `x-initrd.mount` 옵션이 표시된 추가 항목들을 /sysroot/etc/fstab에서 탐색한다. 발견한 모든 항목을 /sysroot 아래에 마운트하며, 그러면 initrd-fs.target에 도달한다. initrd-cleanup.service 서비스는 initrd-switch-root.target으로 이어지는데, 그 과정에서 정리 서비스들이 돌 수 있다. 최종 단계로 initrd-switch-root.service가 활성화되고, 그러면 시스템이 루트를 /sysroot로 바꾸게 된다.

```text
                                : (시작은 위와 동일)
                                :
                                v
                          basic.target
                                |                                 emergency.service
         ______________________/|                                         |
        /                       |                                         v
        |            initrd-root-device.target                   *emergency.target*
        |                       |
        |                       v
        |                  sysroot.mount
        |                       |
        |                       v
        |             initrd-root-fs.target
        |                       |
        |                       v
        v            initrd-parse-etc.service
(자체적인 initrd                |
  서비스들...)                  v
        |             (sysroot-usr.mount 및
        |              fstab 옵션으로 표시된
        |                다양한 마운트
        |              x-initrd.mount...)
        |                       |
        |                       v
        |                initrd-fs.target
        \______________________ |
                               \|
                                v
                           initrd.target
                                |
                                v
                      initrd-cleanup.service
                     initrd-switch-root.target
                         타깃으로 이어짐
                                |
                                v
         ______________________/|
        /                       v
        |        initrd-udevadm-cleanup-db.service
        v                       |
(자체적인 initrd                |
  서비스들...)                  |
        \______________________ |
                               \|
                                v
                    initrd-switch-root.target
                                |
                                v
                    initrd-switch-root.service
                                |
                                v
                         호스트 OS로 이행
```

## 시스템 관리자 셧다운

systemd의 시스템 셧다운 역시 다양한 타깃 유닛들로 이뤄져 있으며 약간의 순서 구조가 있다.

```text
                                   (모든 시스템   (모든 파일 시스템
                                    서비스와의      마운트, 스왑
                                       충돌)         cryptsetup
                                         |           장치 등과의
                                         |              충돌)
                                         |                |
                                         v                v
                                  shutdown.target    umount.target
                                         |                |
                                         \_______   ______/
                                                 \ /
                                                  v
                                           (다양한 저수준
                                              서비스들)
                                                  |
                                                  v
                                            final.target
                                                  |
            _____________________________________/ \_________________________________
           /                         |                        |                      \
           |                         |                        |                      |
           v                         v                        v                      v
systemd-reboot.service   systemd-poweroff.service   systemd-halt.service   systemd-kexec.service
           |                         |                        |                      |
           v                         v                        v                      v
   *reboot.target*           *poweroff.target*          *halt.target*         *kexec.target*
```

흔히 쓰는 시스템 셧다운 타깃들이 \*강조\*되어 있다.

참고로 `systemd-halt.service(8)`, systemd-reboot.service, systemd-poweroff.service, systemd-kexec.service는 시스템 및 서버 관리자(PID 1)를 (systemd-shutdown 바이너리에 구현된) 시스템 셧다운 두 번째 단계로 전환시키게 된다. 그 단계에서는 더이상 서비스나 유닛 개념을 고려하지 않고 단순하고 견고한 방식으로 남은 파일 시스템이 있으면 언마운트하고, 남은 프로세스가 있으면 죽이고, 남은 기타 자원이 있으면 해제한다. 그 시점에서 보통 응용과 자원은 일반적으로 이미 종료되고 해제되어 있으며, 그래서 두 번째 단계는 위에서 설명한 유닛 기반의 첫 번째 셧다운 단계에서 어떤 이유로 멈추거나 해제하지 못한 것들에 대한 안전망 역할을 할 뿐이다.

## SEE ALSO

`systemd(1)`, <tt>[[boot(7)]]</tt>, `systemd.special(7)`, `systemd.target(5)`, `systemd-halt.service(8)`, `dracut(8)`

## NOTES

1. GRUB

    <https://www.gnu.org/software/grub/>

----

systemd 247
