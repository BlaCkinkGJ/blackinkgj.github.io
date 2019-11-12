---
layout: post
title: "[IoT] 라즈베리 파이에 IoTivity 설치"
date: 2019-11-12
excerpt: "Redis를 설치해본다."
tag:
- IoT
- raspberryPi
comments: true
---

# OCF 및 IoTivity란?

OCF(Open Connectivity Foundation)는 사물인터넷을 구현 시 REST 구조 기반으로 경량형 Coap 프로토콜을 사용해서 IoT 장비들끼리 연결 및 자원을 공유하여 상호 제어할 수 있도록 해주는 표준 플랫폼 기술입니다. 이러한 OCF는 기존의 OIC(Open Interconnect Consortiun)와 UPnP 포럼이 통합해서 만든 기업 표준화 단체입니다[^1].

이 재단에서 제정된 표준 규격을 따르는 프로그램을 손쉽게 개발할 수 있도록 [IoTivity]( https://iotivity.org/ )라고 불리는 플랫폼을 제공합니다. IoTivity는 고성능 장치를 위한 IoTivity와 저사양 장치를 위한 IoTivity-Lite로 구분되며, Apache 라이센스 2.0을 준수하므로 누구든 마음대로 사용할 수 있는 장점이 있습니다.

그리고 이러한 OCF 표준을 따르는 장치를 만든다고 하여 종료되는 것이 아니라 OCF 표준 단체에서 해당 장치가 OCF 규격에 맞는 장치임을 반드시 인정을 받아야 합니다.

# SCONS

IoTivity를 설치하기 전에는 `scons`라고 불리는 빌드 툴에 관해서 알아야 합니다. `scons`는 `cmake`나 `Makefile`과 같은 Python을 기반으로 하는 빌드 시스템입니다. `scons`를 실행하기 위해서는 `SConstruct`랑 `SConscript`를 필요로 합니다. `scons`가 실행되면 `SConstruct`를 먼저 읽도록 합니다. `SConstructor`은 다음과 같이 구성됩니다.

```bash
SConscript("SConscript")
```

`SConscript`에 해당하는 파일의 문자열을 `SConscript(<file>)` 의 `<file>`에 넣도록 합니다. 그리고 `SConscript` 파일이 실질적으로 컴파일을 수행하는 파일로서 임의의 `source.cpp` 파일에 대한 프로그램을 제작하는 `SConscript` 파일은 아래와 같습니다.

```bash
env = Environment()

env.Append(LIBPATH=['/usr/local/lib'])
env.Append(LIBS=['hiredis'])
env.Append(CPPPATH=['/usr/local/include/hiredis'])
env.Append(CPPDEFINES=['TEST_MODE'])
env.Append(CCFLAGS=['-Wall', '-Werror'])

env.Program('source.cpp')
```

여기서 `env = Environment()`는 빌드 환경을  가져오는 것입니다. 여기서 `LIBPATH`, `LIBS`들은 `gcc`로 빌드 할 때의 `-L`과 `-l`에 해당하는 내용입니다. 나머지 내용도 `-I`, `-D`와 부가적인 플래그를 지칭합니다. 이렇게 환경을 설정을 하면 `env.Program('source.cpp')`를 하면 리스트의 가장 처음에 있는 파일의 이름(`source`)으로 프로그램이 빌드 되게 됩니다.

좀 더 자세한 내용을 알고 싶으시면 [참고 자료](https://scons.org/doc/1.0.0/HTML/scons-user/ )를 확인해주시길 바랍니다.

# IoTivity

IoTivity를 빌드하기 위한 패키지를 설치하기 위해 아래의 명령어를 수행하도록 합니다.

```bash
curl https://openconnectivity.github.io/IOTivity-setup/install.sh  | bash
```

이 다음에 github에서 라즈베리 파이 iotivity 샘플을 복제해서 가져옵니다. 주의 사항으로 `-b`를 주지 않고 `master`를 가져오면 안 돌아갈 수 있기 때문에, 반드시 `-b` 옵션을 부여해 `iotivity` 브랜치에 있는 내용을 가져오도록 합니다.

```bash
git clone https://github.com/2nd-Chance/Fire-evacuation-guidance-system-IoT.git -b iotivity
mv Fire-evacuation-guidance-system-IoT/iotivity/ ~/ && rm -rf Fire-evacuation-guidance-system-IoT/
```

그리고 추가적으로 필요한 패키지 및 소스를 받도록 합니다.

```bash
cd iotivity && sudo apt install wiriginpi # (wiringpi는 라즈베리파이의 GPIO를 쉽게 다룰 수 있게 해줌)
git clone https://github.com/01org/tinycbor.git extlibs/tinycbor/tinycbor -b v0.4.1
git clone https://github.com/ARMmbed/mbedtls.git extlibs/mbedtls/mbedtls -b mbedtls-2.4.2
```

준비가 되었으면 빌드를 수행하도록 됩니다. 혹시 빌드 발생 시에 오류가 발생하면 `scons -c`를 통해 이전 빌드 내용을 제거하고 다시 빌드를 수행해주시길 바랍니다.

```bash
scons resource/examples/os_examples/ TARGET_TRANSPORT=IP SECURED=0
```

그러면 `out/linux/armv71/release/resource/example/os_examples/` 폴더에 실행 파일이 있을 것입니다. `ocf_server_light/LightServer`를 먼저 실행하고, 또 다른 셸에  `ocf_client/Client`를 실행하도록 합니다.

참고 자료
---

[^1]: http://iotforum.kr/board1/read.asp?bdNum=101&sc_field=title&sc_word=OCF&bdCode=13380