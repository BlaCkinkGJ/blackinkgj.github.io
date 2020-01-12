---
layout: post
title: "[Zsh] Zsh 및 oh my zsh 설치"
date: 2020-1-12
excerpt: "Zsh 및 oh my zsh를 설치해본다."
tag:
- blog
comments: true
typora-root-url: ../../
---

# Zsh 및 oh my zsh

리눅스에서 데비안 환경(e.g. 우분투)에 기반하여 본 문서는 작성되었습니다.

## Zsh란?

Z shell(Zsh)은 Bash와 같이 유닉스 쉘의 일종으로 1990년 처음 개발되었습니다.[^1] 주요 특징으로는 다음과 같습니다.[^2]

1. 실행 중인 모든 Shell은 명령어의 history를 쉘 끼리 공유합니다.
2. 간단한 설정을 통해 문법 오류를 정정해줍니다. (e.g. `gut` → `git`)
3. 다양한 테마를 지원합니다.

특히, 많은 창을 열어서 쓰는 입장에서는 `1` 기능이 너무 좋아서 저는 사용하고 있습니다.

## Zsh 설치

Zsh의 설치는 간단합니다.

```bash
sudo apt install -y zsh
```

이 명령 하나면 Zsh가 설치됩니다. 이 다음에 기본 값으로 설정된 bash에서 zsh로 변환하는 과정이 필요합니다. 따라서 다음의 명령을 수행해주시길 바랍니다.

```bash
chsh -s `which zsh`
```

이는 기본 쉘 값을 zsh의 위치로 맞춰주는 명령입니다. 단, 유의사항으로 `chsh -s 'which zsh'`가 아니라는 점입니다.  그리고 `exit`을 통해 쉘을 한 번 나간 후에 **<u>다시 쉘을 실행</u>**시켜주시길 바랍니다. 그러면 다음과 같은 화면이 뜨게 됩니다.

```bash
Your Hardware Enablement Stack (HWE) is supported until April 2023.
Last login: ################## from #######
This is the Z Shell configuration function for new users,
zsh-newuser-install.
You are seeing this message because you have no zsh startup files
(the files .zshenv, .zprofile, .zshrc, .zlogin in the directory
~).  This function can help you with a few settings that should
make your use of the shell easier.

You can:

(q)  Quit and do nothing.  The function will be run again next time.

(0)  Exit, creating the file ~/.zshrc containing just a comment.
     That will prevent this function being run again.

(1)  Continue to the main menu.

(2)  Populate your ~/.zshrc with the configuration recommended
     by the system administrator and exit (you will need to edit
     the file by hand, if so desired).

--- Type one of the keys in parentheses ---
```

여기서 그냥 `enter` 키를 쳐주시면 알아서 `~/.zshrc`라는 Zsh를 위한 설정파일을 만들어 줍니다. 그리고 다음의 명령을 쳐서 bash에서 Zsh로 변경이 되었는 지 확인을 해보도록 합니다.

```bash
echo $SHELL
```

이때, `Zsh`가 나온다면 제대로 수행한 것입니다.

## Oh My Zsh의 설치

Oh My Zsh는 Zsh의 설정을 다루는 플러그인으로 Zsh를 보다 쉽고 편하게 사용하게 해줍니다.[^3] 다음의 명령을 치면 자동으로 설치가 이루어집니다. 만약 `curl`이 설치되어 있지 않는 경우에는 `sudo apt install curl`을 통해서 설치해주시길 바랍니다.

```bash
curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh
```

그리고 설치를 한 후에 `zsh`를 커맨드 창에 치면 Oh My Zsh가 적용된 쉘을 만나볼 수 있습니다. 이 쉘은 아래와 같은 모습을 띄고 있습니다.

![zsh install](https://www.ivaylopavlov.com/wp-content/uploads/2017/04/Screenshot-2017-04-30-00.43.48-768x402.png)

자동으로 문법 체크를 하는 기능을 키고 싶으시다면 `setopt correct` 명령을 주면 자동으로 문법 체크를 해줍니다.

## Zsh 테마의 설치

Zsh 테마를 설치하고 싶은 경우에는 먼저 [사이트](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes)에서 원하는 테마를 찾도록 합니다. 여기서 저는 [geometry](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes#geometry_)(아래 그림 참조) 테마를 적용하도록 하겠습니다. 그 경우 아래의 절차를 따르면 됩니다.

![zsh geometry](https://raw.githubusercontent.com/frmendes/geometry/master/screenshots/geometry.png)

먼저 [사이트](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes)의 글에서 [geometry](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes#geometry) 항목을  찾아서 See [repo](https://github.com/frmendes/geometry) for source. 라고 적힌 부분에서 `repo` 하이퍼링크를 눌러주도록 합니다.

각 저장소 마다 적혀있는 installation manual에 따라 설치를 해주도록 합니다. 저는 다음의 순서로 설치를 진행하였습니다.

```bash
cd $HOME
git clone https://github.com/geometry-zsh/geometry
echo "source geometry/geometry.zsh" >> ~/.zshrc
```

## 명령어가 echo되어 나올 때

만약 명령어가 echo되어 나온다면 `~/.zshrc`에서 주석 처리되어 있는 `DISABLE_AUTO_TILE="true"`의 주석을 해제시켜주면 됩니다.





---

[^1]: https://groups.google.com/forum/#!msg/alt.sources/tVgN49u8Ax4/7VgQlHZ4bJMJ
[^2]: https://en.wikipedia.org/wiki/Z_shell#cite_note-6
[^3]: https://nolboo.kim/blog/2015/08/21/oh-my-zsh/