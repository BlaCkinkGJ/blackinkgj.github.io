---
layout: post
title: "[uftrace] uftrace의 tui를 사용해보도록 하자."
date: 2019-12-24
excerpt: "uftrace의 tui를 사용해보도록 하자"
tag:
- uftrace
comments: true
typora-root-url: ../../
---

# uftrace의 TUI(Text User Interface)를 사용해보도록 하자

## 지난 글

1. [[uftrace] uftrace의 설치와 사용법](https://blackinkgj.github.io/uftrace-installation/)

## TUI?

TUI는 Text User Interface의 약자로 mdir과 같이 텍스트를 기반으로 하는 사용자 인터페이스를 의미합니다.

![mdir image](https://macsplex.com/files/attach/images/11642/449/028/7f5e7718c16c67892e966c7c75f60cde.png)

이는 명령줄에 기반한 사용에 비해 훨씬 간편하게 사용할 수 있는 특징이 있으며, 직관적으로 상태를 파악할 수 있는 장점이 있습니다. 이러한 tui를 uftrace에서도 지원을 합니다.

## TUI를 사용하기 위한 준비

먼저 TUI를 사용하기에 앞서서 [링크](https://github.com/hishamhm/htop)에 있는 내용을 아래와 같이 원하는 위치에 복제를 하도록 합니다.

```bash
git clone https://github.com/hishamhm/htop.git
```

이 다음에 `cd htop`을 하여 `htop` 폴더로 이동하고, 아래와 같은 명령을 수행하도록 합니다.

```bash
sudo apt install -y dh-autoreconf
./autogen.sh
./configure
```

앞선 자료에서 언급했듯이 uftrace를 돌리기 위해서는 `-pg` 옵션이 컴파일 할 때에 반드시 필요하다고 했습니다. 따라서 `-pg` 옵션을 주기 위해서 `Makefile`을 열어 `CFLAGS = -D_GNU_SOURCE -D_DEFAULT_SOURCE ...`를 찾아 `-pg`를 넣어주도록 한 후에 `make`를 수행해주시면 됩니다.

그러면 htop 바이너리가 만들어진 것을 알 수 있습니다.

## TUI의 사용

먼저, `uftrace record -a ./htop`을 통해서 프로그램을 실행해주도록 합니다. 그리고 `uftrace tui -t 1ms`을 해주면 아래와 같은 결과가 나타나게 됩니다.

![image-20191224115121829](/assets/img/res/2019-uftrace/tui-1.png)

그리고 십자키 또는 vi 에디터와 같이 h,j,k,l 키를 사용하면 이동할 수 있습니다. 문제는 이것으로는 개괄만 알 수 있다는 것입니다. 따라서 좀 더 상세히 확인할 수 있는 키를 찾아봐야 하고, 이를 위해서 `h` 키를 입력해주면 아래와 같은 창이 나타나게 됩니다.

![image-20191224115437435](/assets/img/res/2019-uftrace/tui-2.png)

이를 테면, `R`을 입력하면 이전 문서에서 설명한 report 내용이 나오게 됩니다. 그리고 내용 중에 내가 필요로 하는 내용만 따로 추출할 수 있도록 `s` 키도 지원합니다. `s`키를 잘 활용하면 내가 원하는 내용만 따로 추출할 수 있습니다.

이를 `sudo uftrace record -a -k ./htops`를 하면 커널 내부 함수도 후킹을 할 수 있으며, `uftrace tui -t 1us`을 하면 좀 더 세밀하게 그 내용을 확인할 수 있으며, `c`와 같은 커맨드를 활용해서 기존의 replay와 달리 좀 더 컴팩트한 결과를 보고, 코드 분석에 사용할 수 있습니다.

## 다음 글

1. [[uftrace] uftrace를 사용한 시각화](https://blackinkgj.github.io/uftrace-visualization/)
2. [[uftrace] 기존 바이너리 fio 트레이싱](https://blackinkgj.github.io/uftrace-fio/)