---
layout: post
title: "tcmu-runner를 이용해서 Zoned Storage를 생성하고 테스트해보자."
date: 2020-07-02
excerpt: "tcmu-runner를 이용해서 Zoned Storage를 생성하고 테스트해봅시다."
tag:
- container
comments: true
typora-root-url: ../../
---

# Zoned Storage의 생성 및 테스트

본 자료는 Fedora 32(linux kernel 5.6.0)에 기반하여 작성되었으며, 이 [자료](https://zonedstorage.io/projects/tcmu-runner/)를 참고했습니다.

## Zoned Storage 생성

먼저, tcmu-runner를 사용하기 위해서는 커널을 컴파일을 할 때에 `make menuconfig`를 통해서 `Generic Target Core Mode(TCM) and ConfigFS Infrastructure`를 enable을 해줘야 합니다. 아마 일반적인 설정 파일의 경우에는 `<M>`으로 설정이 되어 있을 것입니다. 이를 `<*>`로 `y`키를 눌러서 변경해줘야 합니다.

그리고 아래의 명령을 통해서 `tcmu-runner`와 관련 유틸리티를 받아주도록 합니다.

```bash
dnf install tcmu-runner
dnf install targetcli
```

그리고 아래와 같이 쉘 파일을 작성하고, `disk_on.sh`라고 명명한 후에 실행이 가능하게 모드(e.g. `chmod 775 ./disk_on.sh`)를 변경해주도록 합니다. 이때, 현재 저는 nvme 디스크가 마운트 된 /mnt/nvme에 raw 파일이 만들어질 수 있도록 `cfgstring`을 설정했습니다.

```bash
#!/bin/bash

if [ $# != 5 ]; then
        echo "Usage: $0 <disk name> <cap (GB)> HM|HA <zone size (MB)> <conv zones num>"
        exit 1;
fi

dname="$1"
cap="$2"
model="$3"
zs="$4"
cnum="$5"

naa="naa.50014059cfa9ba75"

# Setup emulated disk
cat << EOF | targetcli

cd /backstores/user:zbc
create name=${dname} size=${cap}G cfgstring=model-${model}/zsize-${zs}/conv-${cnum}@/mnt/nvme/${dname}.raw
cd /loopback
create ${naa}
cd ${naa}/luns
create /backstores/user:zbc/${dname} 0
cd /
exit

EOF

sleep 1
disk=`lsscsi | grep "TCMU ZBC device" | cut -d '/' -f3 | sed 's/ //g'`
echo "mq-deadline" > /sys/block/"${disk}"/queue/scheduler
```

이를 실행하려면 아래와 같이 작성해서 실행하면 `/dev/sda`에 Zoned Storage가 생기게 됩니다.

```bash
sudo ./disk_on.sh zbc0 25 HM 256 10
```

여기서 `zbc0`은 디스크의 이름에 해당하고, `25`는 디스크의 크기, `HM`은 Host Managed의 약자입니다. 그리고 `256`은 하나의 zone의 크기를 의미하고, Sequential Write가 아닌 Conventional Write(일반 SSD식 write)를 하는 Zone의 갯수를 10개로 설정하는 것을 의미합니다. 자세한 내용은 [링크](https://zonedstorage.io/projects/tcmu-runner/)를 참조해주시길 바랍니다.

## Zoned Storage의 제거

Zoned Storage를 제거하는 코드는 `disk_off.sh`라고 명명한 후에 생성하는 코드와 같은 모드로 설정한다음 다음과 같이 적으시고, zbc가 `/dev/sda`인 경우에는 `sudo ./disk_off.sh /dev/sda`라고 치면 됩니다.

```bash
#!/bin/bash

if [ $# != 1 ]; then
    echo "Usage: $0 <disk name (e.g. zbc0)"
    exit 1;
fi

dname="$1"

naa="naa.50014059cfa9ba75"

# Delete emulated disk
cat << EOF | targetcli

cd /loopback/${naa}/luns
delete 0
cd /loopback
delete ${naa}
cd /backstores/user:zbc
delete ${dname}
cd /
exit

EOF
```

## libzbc의 설치와 테스트

zbc가 정상 작동하는 지를 확인하기 위해서 libzbc를 설치했습니다. 설치 과정은 다음과 같습니다. 먼저, 관련 패키지를 설치해주도록 합니다.

```bash
sudo dnf install autoconf autoconf-archive automake libtool
```

그리고 아래의 명령을 통해서 설치를 해주도록 합니다. 여기서 코어 수가 12개이면 `-j12`라고 쓰고, 4이면 `-j4`라고 하면 됩니다.

```bash
sh ./autogen.sh
./configure
make -j12 --with-test
```

마지막으로, 만약 생성된 Zoned Storage의 위치가 `/dev/sda`인 경우에는 `cd test`를 한 후에 아래의 명령을 수행하면 됩니다.

```bash
sudo ./zbc_test.sh /dev/sda
```

