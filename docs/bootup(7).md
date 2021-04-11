## NAME

bootup - 시스템 부팅 과정

## DESCRIPTION

시스템 부트에는 여러 다양한 요소들이 관련되어 있다. 시스템 전원이 들어온 직후 시스템 BIOS에서 최소한의 하드웨어 초기화를 하고 나서 영속 저장 장치에 저장된 부트 로더로 제어를 넘기게 된다. 그러면 부트 로더는 디스크에서 (또는 네트워크에서) OS 커널을 호출하게 된다. 리눅스의 경우에 이 커널은 (선택적으로) 최초 램디스크 이미지(initrd)를 풀어서 실행한다. 이 이미지는 `dracut(8)` 등으로 생성하며 (여기선 아마 `systemd(1)`를 이용해) 루트 파일 시스템을 찾는다. 루트 파일 시스템을 찾아서 마운트 한 다음 initrd는 OS 이미지에 저장된 (`systemd(1)` 같은) 호스트의 시스템 관리자에게 제어를 넘긴다. 그러면 시스템 관리자가 나머지 하드웨어들을 모두 조사하고 필요한 파일 시스템들을 모두 마운트 하고 설정된 서비스들을 모두 시작하는 일을 맡는다.

시스템 정지 시 시스템 관리자는 서비스들을 모두 정지시키고, 파일 시스템들을 모두 언마운트 하고 (기반 저장 기술들을 떼어 내고), (선택적으로) 루트 파일 시스템과 거기 위치한 저장소를 언마운트 하고 떼어 내는 initrd 코드로 되돌아간다. 마지막 단계로 시스템 전원을 끈다.

시스템 부트 과정에 대한 추가적인 내용을 <tt>[[boot(7)]]</tt>에서 찾을 수 있다.

## 시스템 관리자 부팅

부팅 때 OS 이미지 상의 시스템 관리자는 시스템 동작에 필요한 필수 파일 시스템과 서비스, 드라이버의 초기화를 맡는다. `systemd(1)` 시스템에서 이 과정은 다양한 분리된 단계들로 쪼개져서 타겟 유닛들로 노출되어 있다. (타겟 유닛에 대한 자세한 정보는 `systemd.target(5)`을 보라.) 부팅 과정이 고도로 병렬화 되어 있어서 특정 타겟 유닛들에 도달하는 순서가 확정적이지 않다. 하지만 그래도 제한된 양의 순서 구조를 충실히 따른다.

systemd가 시스템을 구동할 때 default.target이 의존하는 모든 유닛들을 (그리고 그 의존 유닛들이 의존하는 것들까지 재귀적으로) 활성화한다. 일반적으로 default.target은 graphical.target이나 multi-user.target의 에일리어스일 뿐이며, 어느 쪽인지는 시스템이 그래픽 UI로 구성되어 있는지 텍스트 콘솔로만 구성되어 있는지에 따라 결정된다. 당겨 오는 유닛들 간에 최소한의 순서 관계만을 강제하기 위해 잘 알려진 타겟 유닛들이 여럿 있다. `systemd.special(7)`에 나열되어 있다.

다음 도표는 그 잘 알려진 유닛들과 부팅 로직 내의 위치를 개괄적으로 보여 준다. 화살표는 어떤 유닛이 다른 유닛 앞으로 당겨져 가는지를 기술한다. 도표 위쪽에 가까운 유닛들이 아래에 가까운 유닛들보다 먼저 시작된다.

```text
local-fs-pre.target
         |
         v
 (다양한 마운트 및    (다양한 스왑    (다양한 cryptsetup
  fsck 서비스...)      장치들...)         장치들...)         (다양한 저수준       (다양한 저수준
         |                  |                  |              서비스: udevd,      API VFS 마운트:
         v                  v                  v             tmpfiles, random     mqueue, configfs,
  local-fs.target      swap.target     cryptsetup.target    seed, sysctl, ...)      debugfs, ...)
         |                  |                  |                    |                    |
         \__________________|_________________ | ___________________|____________________/
                                              \|/
                                               v
                                        sysinit.target
                                               |
          ____________________________________/|\________________________________________
         /                  |                  |                    |                    \
         |                  |                  |                    |                    |
         v                  v                  |                    v                    v
      (다양한            (다양한               |                 (다양한          rescue.service
    타이머들...)        경로들...)             |                소켓들...)               |
         |                  |                  |                    |                    v
         v                  v                  |                    v              rescue.target
   timers.target      paths.target             |             sockets.target
         |                  |                  |                    |
         v                  \_________________ | ___________________/
                                              \|/
                                               v
                                         basic.target
                                               |
          ____________________________________/|                                 emergency.service
         /                  |                  |                                         |
         |                  |                  |                                         v
         v                  v                  v                                _emergency.target_
     display-          (그래픽 UI에         (다양한
 manager.service          필요한         시스템 서비스)
         |                다양한               |
         |            시스템 서비스)           v
         |                  |          _multi-user.target_
         |                  |                  |
         \_________________ | _________________/
                           \|/
                            v
                 _graphical.target_
```

부트 타겟으로 흔히 쓰이는 타겟 유닛들을 \_강조 표시\_하였다. 이 유닛들은 목표 타겟으로 하기에 좋은 선택지이다. 예를 들어 `systemd.unit=` 커널 명령행 옵션(`systemd(1)` 참고)에 전달하거나 심볼릭 링크로 default.target이 그 유닛을 가리키게 하면 된다.

timers.target은 basic.target과 비동기적으로 당겨 온다. 이렇게 하면 타이머가 부팅 후반에서야 사용 가능해지는 서비스에 의존할 수 있다.

## 최초 램디스크에서의 부팅 (INITRD)

systemd를 사용해서도 최초 램디스크(initial RAM disk) 구현(initrd)을 구성할 수 있다. 이 경우에 initrd 내의 부팅은 다음 구조를 따른다.

initrd 내에서 기본 타겟은 initrd.target이다. basic.target에 도달할 때까지는 시스템 관리자 부팅(위 내용 참고)과 동일하게 부팅 과정이 시작한다. 거기서부터 systemd는 특수한 타겟인 initrd.target으로 접근한다. 파일 시스템을 마운트 하기 전에 시스템이 하이버네이션에서 복귀할 것인지 아니면 정상 부트를 진행할 것인지 결정해야 한다. 이 결정은 systemd-hibernate-resume@.service에서 이뤄지며, local-fs-pre.target 전에 완료되어야 한다. 따라서 그 확인이 끝나기 전에는 어떤 파일 시스템도 마운트 할 수 없다. 루트 장치가 사용 가능해지면 initrd-root-device.target에 도달한다. 그 루트 장치를 /sysroot에 마운트 할 수 있으면 sysroot.mount 유닛이 활성화되고 initrd-root-fs.target에 도달한다. 서비스 initrd-parse-etc.service에서는 /sysroot/etc/fstab에서 가능한 /usr 마운트와 `x-initrd.mount` 옵션이 표시된 추가 항목들을 탐색한다. 발견한 모든 항목들을 /sysroot 아래에 마운트 하며, 그러면 initrd-fs.target에 도달한다. 서비스 initrd-cleanup.service는 initrd-switch-root.target으로 격리되는데(isolate), 거기서 정리 서비스가 돌 수 있다. 최종 단계로 initrd-switch-root.service가 활성화되고, 그러면 시스템이 /sysroot로 루트를 전환하게 된다.

```text
                                               : (시작은 위와 동일)
                                               :
                                               v
                                         basic.target
                                               |                                 emergency.service
                        ______________________/|                                         |
                       /                       |                                         v
                       |            initrd-root-device.target                   _emergency.target_
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
                       |             fstab 옵션으로 표시된
                       |               다양한 마운트들
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
                                       다음 타겟으로 격리
                                    initrd-switch-root.target
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

## 시스템 관리자 정지

systemd에서의 시스템 정지 역시 다양한 타겟 단위들로 이뤄져 있으며 어떤 최소한의 순서 구조가 적용된다.

```text
                                   (모든 시스템     (모든 파일
                                    서비스와의     시스템 마운트,
                                       충돌)      스왑, cryptsetup
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
   _reboot.target_           _poweroff.target_          _halt.target_         _kexec.target_
```

흔히 쓰는 시스템 정지 타겟들이 강조되어 있다.

## SEE ALSO

`systemd(1)`, <tt>[[boot(7)]]</tt>, `systemd.special(7)`, `systemd.target(5)`, `dracut(8)`

----

systemd 234
