---
layout: post
title: "[Linux Shell] Linux 쉘(sh)에서의 if문 사용"
date: 2019-10-2
excerpt: "Linux 쉘(sh)에서 if문을 사용 해보도록 한다."
tag:
- shell script
comments: true
typora-root-url: ../../
---

# Linux shell에서의 if문 사용

쉘(sh)는 여러모로 유용합니다. 이러한 쉘 스크립트를 미리 만들어놓으면 많은 귀찮은 작업들을 쉽게 수행할 수 있습니다. 본 게시물에서 유의해야 할 점은 Bash가 아니라 sh라는 점입니다. 쉘(sh)을 사용하는 이유는 개인적인 기호이기에 둘 중 뭐가 낫느냐고 하면 원하시는 것을 아무거나 선택하시면 될 것 같습니다.

쉘에서의 if는 조금 독특합니다. BASIC에서나 볼 법한 문법을 하고 있는 것을 알 수 있습니다. 이를테면, 가장 단순한 if문은 다음과 같이 제작 가능합니다.

```sh
VALUE=0
if [ $VALUE -eq 1 ]
then
	echo "\$VALUE is 1"
else
	echo "\$VALUE is not 1"
fi
```

여기서 `[$value -eq 1]`이라고 적는 경우에는 오류가 발생할 수 있으니, **<u>반드시 앞뒤로 공백</u>**을 만들어서 `[ $value -eq 1  ]`이 될 수 있도록 제작해야 합니다. 그리고 echo에 `\$VALUE`라고 적었는 데, 이렇게 한 이유는 `$VALUE`로 한 경우 `echo` 출력 값이 `0 is 1`과 같이 될 수 있기 때문입니다. 하지만 우리는 `$VALUE`라는 출력을 보고 싶으므로 `\`를 사용해서 출력을 유도하도록 합니다.

그리고 식에서 나타나는 `-eq`는 동등을 의미합니다. `if` 문에서 많이 사용되는 연산자는 아래와 같습니다.

| 항 목 | true 반환 조건                                            |
|:-----:| --------------------------------------------------------- |
| -eq   | 두 값의 같이 경우                                         |
| -ne   | 두 값이 다른 경우                                         |
| -lt   | 오른쪽 값보다 왼쪽 값이 작은 경우                         |
| -le   | 오른쪽 값보다 왼쪽 값이 작거나 같은 경우                  |
| -gt   | 오른쪽 값보다 왼쪽 값이 큰 경우                           |
| -ge   | 오른쪽 값보다 왼쪽 값이 크거나 같은 경우                  |
| -z    | 문자열의 길이가 0인 경우                                  |
| -n    | 문자열의 길이가 0이 아닌 경우                             |
| ==    | 두 개의 문자열이 동일한 경우                              |
| !=    | 두 개의 문자열이 서로 다른 경우                           |
| <     | 왼쪽의 문자열이 오른쪽의 문자열보다 정렬 시 선행되는 경우 |
| >     | 오른쪽의 문자열이 왼쪽의 문자열보다 정렬 시 선행되는 경우 |
{:.table .table-bordered}

| 항 목 | 내 용                         | 예제                                    |
|:-----:| ----------------------------- | --------------------------------------- |
| &&    | 두 논리 식에 AND를 수행       | `if [ condition 1 ] && [ condition 2 ]` |
| \|\|  | 두 논리 식에 OR을 수행        | `if [ condition 1 ] || [ condition 2 ]` |
| !     | 논리 식의 결과값에 NOT을 수행 | `if [ ! condition ]`                    |
{:.table .table-bordered}

만약에 좀 더 컴팩트하게 작성하고 싶은 경우에는 다음과 같이 작성을 하면 됩니다.

```sh
VALUE=0
if [ $VALUE -eq 1 ]; then
	echo "\$VALUE is 1"
else
	echo "\$VALUE is not 1"
fi
```

이러한 쉘 파일을 실행하고자 할 때에 두 가지 방법이 있습니다. 어떤 임의의 `sample.sh` 쉘 파일을 만들어서 돌린다고 할 때에 첫 번째 방법은 아래와 같습니다.

```sh
sh sample.sh
```

하지만 매 번 `sample.sh`를 `sh sample.sh`와 같이 쓰는 것은 손을 낭비하는 일이기 때문에, 좀 더 간결하게 만들어야 합니다. 간결하게 만들려면 먼저 파일 안에 `#!/bin/sh`를 추가해주도록 합니다. 자고로 `#`은 주석을 표현하는 표시이고 `!/bin/sh`는 해당 위치(`/bin/sh`)의 프로그램을 기반으로 이 파일을 돌려라는 것을 의미합니다. 지금의 경우에는 쉘로 돌려라는 것을 명시합니다. 즉, 정리하면 아래와 같이 파일을 구성하면 됩니다.

```sh
#!/bin/sh
# file-name: sample.sh

VALUE=0
if [ $VALUE -eq 1 ]; then
	echo "\$VALUE is 1"
else
	echo "\$VALUE is not 1"
fi
```

이러고 바로 `./sample.sh`를 수행하는 경우에 정상적으로 동작하지 않을 수 있습니다. 그런 경우에는 아래와 같은 명령을 수행해주면 됩니다.

```bash
chmod 755 sample.sh
```

`755` 모드는 `111 101 101`이므로 소유자는 읽기, 쓰기, 실행의 권한을 가지고 그룹 및 익명 사용자는 읽기, 실행의 권한을 가지게 하는 모드입니다. (일반적으로, 우분투에서 파일을 만들면 `664` 모드로 만들어 집니다.)

여기까지 하면 `./sample.sh`를 통해서 실행을 하실 수 있게 되셨을 겁니다. 좀 더  복잡하게 작성하면 다음과 같이 작성할 수 있습니다.

```sh
#!/bin/sh
# file-name: sample.sh

read VALUE

if [ $VALUE -eq 1 ]; then
	echo "$VALUE is 1"
elif [ $VALUE -gt 1 ] && [ $VALUE -le 5 ]; then
	echo "1 < $VALUE ≤ 5"
else
	echo "$VALUE > 5 or $VALUE < 1"
fi
```

이 프로그램은 표준 입력으로 부터 받은 `VALUE` 변수 값을 판단하여 결과를 보여주는 것입니다. 만약 `3`을 입력을 한다면 출력으로 아래와 같이 나오게 됩니다.

```bash
1 < 3 ≤ 5
```

이러한 쉘에서의 `if`문이 직관적으로 어디에 사용될 지 잘 모르실 수도 있을 것 같습니다. 아래와 같은 자동 실행기를 제작하는 데에 사용될 수 있습니다.

```sh
#!/bin/sh
BZ_IMAGE_LOCATION=linux/arch/x86_64/boot/bzImage
LISTEN_PORT=1234

if [ $1 = "--load" ]
then
        BZ_IMAGE_LOCATION=image/$2
fi

if [ $1 = "--original" ]
then
BZ_IMAGE_LOCATION=original/linux/arch/x86_64/boot/bzImage
fi

echo "[1] loaded location ==> $BZ_IMAGE_LOCATION"
echo "[2] listen $LISTEN_PORT"

sudo ./qemu-nvme/x86_64-softmmu/qemu-system-x86_64 \
    -hda ubuntu.img \
    -m 16G -smp 20 -cpu host --enable-kvm \
    -vnc :2 \
        -drive file=ocssd.img,id=myocssd,if=none \
    -device nvme,drive=myocssd,lnum_pu=10,lstrict=1,meta=16,mc=3,serial=foo \
    -net user,hostfwd=tcp::$LISTEN_PORT-:22 \
    -net nic \
    -kernel $BZ_IMAGE_LOCATION -append 'root=/dev/sda1 console=ttr0 nokaslr'

```

이 프로그램은 터미널에서 받은 인자(`$1`, `$2`)를 바탕으로 해서 이미지를 교체하는 자동 QEMU 실행기입니다. 이 실행기의 파일 이름이 `run`인 경우에 아래와 같이 적게 되면 기존의 리눅스 커널 이미지가 올라가게 됩니다.

```bash
./run --original
```

반대로 아래와 같이 적는 경우에는 2번째 인자 값을 바탕으로 커널 이미지를 불러오도록 합니다.

```bash
./run --load linux_4_10.img
```

결과적으로, 이런 쉘의 조건문을 적절하게 쓰면 내가 원하는 파일이나 매크로를 수행할 수 있는 것을 알 수 있습니다.