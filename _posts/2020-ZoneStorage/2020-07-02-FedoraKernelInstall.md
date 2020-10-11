---
layout: post
title: "[Linux] 리눅스에 커스텀 커널 설치 방법"
date: 2020-07-02
excerpt: "커스텀 커널을 제작하고 리눅스에 설치해보도록 한다."
tag:
- container
comments: true
typora-root-url: ../../
---

# 커널 다운로드 과정

먼저, 커스텀 커널을 git에서 받도록 합니다. [리눅스 커널 아카이브](www.kernel.org)에서도 받을 수 있지만 한국의 경우에는 그 속도가 느리기 때문에 github에 들어가서 [torvalds/linux](https://github.com/torvalds/linux)에서 커널을 다운로드를 받도록 합니다.

이때, 커널 버전 전체가 필요한 경우에는 그냥 `git clone https://github.com/torvalds/linux.git`을 수행하면 됩니다. 하지만 만약 특정 커널 버전(e.g. v5.2)이 필요한 경우에는 아래의 명령을 수행하도록 합니다.

```bash
git clone -b v5.2 --single-branch --depth 1 https://github.com/torvalds/linux.git
```

여기서 `--depth 1`은 커널의 커밋 가져오는 것의 횟수를 최소화하기 위해서 사용하는 명령입니다. 만약 해당 버전의 커널 커밋이 전부 필요한 경우에는 저 부분은 삭제해도 상관없습니다.

이제 리눅스 커널 파일들이 생겼으니 컴파일을 하고 설치하는 과정을 거치도록 하겠습니다.

# Ubuntu에서의 커스텀 커널 설치 방법

## 빌드 도구 설치

본 자료는 [링크](https://medium.com/@simonconnah/how-to-compile-a-custom-linux-kernel-on-ubuntu-19-04-9b623ece5922) 내용에 기반되어 작성되었습니다. 커스텀 커널을 설치하기 위해서는 커널 빌드할 파일들을 받아야 하기 때문에 아래의 명령을 통해서 커널 빌드할 파일들을 받아주도록 합니다.

```bash
sudo apt update && sudo apt upgrade
sudo apt install libncurses5-dev flex bison libssl-dev wget
```

## 설정 파일 생성

이 다음에 방금 다운로드 받은 리눅스 커널 파일들이 들어있는 디렉터리 위치로 `cd linux`를 통해 이동하도록 합니다.  그 후에 현재 컴퓨터의 커널 설정 파일을 가져오도록 합니다.

```bash
cp /boot/config-$(uname -r) .config
```

그리고 만약 설정 내용을 바꿔야 한다면 `make menuconfig` 명령을 통해 `.config`를 로드하고 설정을 하도록 합니다.

## 최초 컴파일

여기까지 되었으면 자신이 만든 것임을 표기하기 위해서 `vi Makefile`로 `Makefile`을 열어 아래와 같이 원하는 태그를 붙여주도록 합니다.

```bash
EXTRAVERSION = -BlaCkinkGJ
```



 `grep -c processor /proc/cpuinfo`를 통해서 코어의 갯수를 파악하도록 합니다. 이때, 파악된 코어의 갯수가 12개 인경우에는 아래의 절차를 진행하도록 합니다.

```bash
make oldconfig
make -j12
sudo make modules_install -j12
sudo make install -j12
sudo update-grub
```

`sudo update-grub`을 하지 않으면 커널이 등록되지 않으므로 반드시 실시해주도록 합니다.

## 기본 커널로 설정

하지만 이 경우에커스텀 커널이 아니라 기존의 커널이 기본값으로 되어 있어 부팅 때마다 계속 바꿔줘야 하는 문제가 있습니다. 따라서, 이를 해결하기 위해서는 아래의 작업을 실시해야 합니다.

```bash
grep -A100 submenu /boot/grub/grub.cfg | grep menuentry
```

이를 하면 아래와 같이 menuentry들이 나타납니다.

```bash
submenu 'Ubuntu용 고급 설정' $menuentry_id_option 'gnulinux-advanced-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e' {
        menuentry '리눅스 Ubuntu가 있는, 5.3.0-61-generic입니다' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-61-generic-advanced-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e' {
        menuentry 'Ubuntu, with Linux 5.3.0-61-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-61-generic-recovery-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e' {
        menuentry '리눅스 Ubuntu가 있는, 5.3.0-59-generic입니다' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-59-generic-advanced-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e' {
        menuentry 'Ubuntu, with Linux 5.3.0-59-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-59-generic-recovery-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e' {
        menuentry '리눅스 Ubuntu가 있는, 5.3.0-53-generic입니다' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-53-generic-advanced-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e' {
        menuentry 'Ubuntu, with Linux 5.3.0-53-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-53-generic-recovery-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e' {
```

여기서 먼저 `Ubuntu용 고급 설정`의 `$menuentry_id_option` 값과 원하는 커널의 `$menuentry_id_option` 값을 확인해서 `>`로 병합합니다. 예를 들어, 5.3.0-53-generic인 경우에는 아래와 같이 된다고 보면 됩니다.

```bash
gnulinux-advanced-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e>gnulinux-5.3.0-53-generic-advanced-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e
```

이 값을 잘 기억하고 `vi /etc/default/grub`을 실행해주도록 합니다. 여기서 `GRUB_DEFAULT` 부분을 아래와 같이 변경해주도록 합니다. 유의 사항, 절대로 `>` 사이에 공백을 만드셔서는 안됩니다.

```bash
GRUB_DEFAULT="gnulinux-advanced-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e>gnulinux-5.3.0-53-generic-advanced-d47dbd0e-2b57-4064-98f6-41e8ffbd2c2e"
GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=0
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0"
GRUB_CMDLINE_LINUX=""
```

`sudo update-grub`, `sudo reboot`를 수행하면 커스텀 커널로 재부팅이 되는 것을 확인할 수 있습니다. 

## 재컴파일 시

그리고 커널을 지속적으로 컴파일하고 적용을 하는 경우에는 수정 후에 아래의 과정을 진행하면 됩니다.

```bash
make -j12
sudo make modules_install -j12
sudo make install -j12
```

# Fedora에서의 커스텀 커널 설치 방법

## 빌드 도구 설치

본 자료는 [링크](https://asamalik.fedorapeople.org/tmp-docs-preview/quick-docs/kernel/build-custom-kernel/) 내용에 기반하여 작성되었습니다. 먼저, 커널을 컴파일을 하기 위한 빌드 도구들을 설치합니다. 우분투와 달리 아래의 과정을 통해서 진행하도록 합니다. (여담으로 `dnf` 미러 서버 속도를 높이기 위해서는 `/etc/dnf/dnf.conf` 파일에 `fastestmirror=true`를 설정해주시면 됩니다.)

```bash
sudo dnf install fedpkg
fedpkg clone -a kernel
cd kernel
sudo dnf builddep kernel.spec
```

이렇게 하면 커널 빌드에 필요한 도구들을 다 설치할 수 있습니다. 만약 `make xconfig`를 수행하고자 한다면 추가로 아래의 명령을 수행해주시면 됩니다.

```bash
sudo dnf install qt3-devel libXi-devel gcc-c++
```

그리고 빌드를 위해서 유저가 `/etc/pesign/users`에 등록될 수 있도록 아래의 명령을 통해서 추가해주도록 합니다.

```bash
sudo /usr/libexec/pesign/pesign-authorize
```

추가로 재빌드의 속도를 높여주기 위해서 `ccache`도 설치해주도록 합니다.

```bash
sudo dnf install ccache
```

## 설정 파일 생성

우분투에서처럼 커널 컴파일을 위해서 `cd linux`를 통해 아까 받은 리눅스 커널 파일의 디렉터리에 들어가도록 합니다. 그리고 우분투와 비슷하게 config 파일을 아래와 같이 가져온 후에 혹시 수정하는 경우에는 `make menuconfig`를 통해서 수정을 진행하도록 합니다.

```bash
cp /boot/config-`uname -r`* .config
```

## 최초 컴파일

먼저, 우분투와 동일하게 `Makefile`에서 `EXTRAVERSION`을 원하는 이름으로 설정해주도록 합니다. 그리고 커널 컴파일은 거의 우분투와 동일하나, 약간의 단계가 좀 더 많습니다.

```bash
make oldconfig
make bzImage
make modules
sudo make modules_install
sudo make install
```

## 기본 커널로 설정

기본 커널 설정 방식은 우분투보다 좀 더 쉽습니다. 먼저, root 권한을 획득한 후에 `grubby --default-kernel`을 통해서 현재 설정된 index가 무엇인지를 확인하고 아래의 명령을 수행하여 index를 확인합니다.

```bash
grubby --info=ALL
```

그러면 아래와 같은 출력이 나오게 됩니다.

```bash
index=0
kernel="/boot/vmlinuz-5.6.19-300.fc32.x86_64"
args="ro resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap rhgb quiet"
root="/dev/mapper/fedora_localhost--live-root"
initrd="/boot/initramfs-5.6.19-300.fc32.x86_64.img"
title="Fedora (5.6.19-300.fc32.x86_64) 32 (Thirty Two)"
id="b1395f352d3b4836af31ef849e8878a0-5.6.19-300.fc32.x86_64"
index=1
kernel="/boot/vmlinuz-5.6.6-300.fc32.x86_64"
args="ro resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap rhgb quiet"
root="/dev/mapper/fedora_localhost--live-root"
initrd="/boot/initramfs-5.6.6-300.fc32.x86_64.img"
title="Fedora (5.6.6-300.fc32.x86_64) 32 (Thirty Two)"
id="b1395f352d3b4836af31ef849e8878a0-5.6.6-300.fc32.x86_64"
index=2
kernel="/boot/vmlinuz-5.6.0-grad+"
args="ro resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap rhgb quiet"
root="/dev/mapper/fedora_localhost--live-root"
initrd="/boot/initramfs-5.6.0-grad+.img"
title="Fedora (5.6.0-grad+) 32 (Workstation Edition)"
id="b1395f352d3b4836af31ef849e8878a0-5.6.0-grad+"
index=3
kernel="/boot/vmlinuz-0-rescue-b1395f352d3b4836af31ef849e8878a0"
args="ro resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap rhgb quiet"
root="/dev/mapper/fedora_localhost--live-root"
initrd="/boot/initramfs-0-rescue-b1395f352d3b4836af31ef849e8878a0.img"
title="Fedora (0-rescue-b1395f352d3b4836af31ef849e8878a0) 32 (Thirty Two)"
id="b1395f352d3b4836af31ef849e8878a0-0-rescue"
```

여기서 만약 default 커널이 우리가 원한 것이 아닌 경우에는 기본값을 변경해줘야 합니다. 이를테면, `/boot/vmlinuz-5.6.0-grad+`를 설정하고 싶은 경우에는 아래와 같이 수행해주면 됩니다.

```bash
grubby --set-default /boot/vmlinuz-5.6.0-grad+
```

이러면 기본값으로 원하는 커널로 변경되게 됩니다.

## 재컴파일 시

커널을 수정하고 재컴파일을 하는 경우에는 아래의 과정만 진행하면 됩니다.

```bash
make bzImage
make modules
sudo make modules_install
sudo make install
```





