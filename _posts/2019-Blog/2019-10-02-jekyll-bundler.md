---
layout: post
title: "[Jekyll] Jekyll기반 github page를 운영을 위한 도구의 설치"
date: 2019-10-2
excerpt: "Jekyll기반 github page를 운영을 위한 도구의 설치해본다."
tag:
- blog
comments: true
typora-root-url: ../../
---

# Jekyll기반 github page를 운영을 위한 도구의 설치

Jekyll 기반 블로그를 만들어놓고 다른 컴퓨터에서 포스팅을 할 때마다 새로 깔고 지우고 하는 작업을 반복하다가 지쳐서 좀 정리를 하기 위해 만든 글입니다. 따라서 github page 등록 절차나 jekyll의 설치법 또는 리눅스에서의 설치 방법에 대해서는 누락되어 있는 점을 양해 해주시길 바랍니다.

## rubyInstaller 설치

먼저, 윈도우의 경우에 `rubyInstaller`를 통해서 설치하는 것이 가장 좋습니다. 따라서 [이 사이트](https://rubyinstaller.org/)에 들어가서 `Download`를 누르고 자신의 사양에 맞는 버전을 선택하시면 됩니다. (2019-10-02 기준 [Ruby+Devkit 2.6.4-1 (x64)](https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-2.6.4-1/rubyinstaller-devkit-2.6.4-1-x64.exe)을 설치함)

![rubyInstaller](/assets/img/res/2019-Blog/rubyinstaller.PNG)

그리고 인스톨러를 키고 아래의 창이 뜨면 **UTF-8을 외부 인코딩의 default**로 맞추는 작업을 수행해주도록 합니다.

![rubyInstaller](/assets/img/res/2019-Blog/usedefault.PNG)

이후에 Install 버튼을 누르고 설치를 진행하도록 합니다. dev-kit인 경우에는 약간 더 시간이 오래 걸릴 수 있으므로 느긋하게 기다려 주시면 됩니다.

그러면 'MSYS 관련 설치를 할 것인가?'에 관해서 나올 것입니다.

![msys](assets/img/res/2019-Blog/msys.PNG)

이때, 체크 박스에 체크를 하고 Finish를 눌러 주시면 됩니다. 그러면 아래와 같은 창이 뜰 것입니다.

![msys-2](/assets/img/res/2019-Blog/msys-2.PNG)

만약 본인이 필요한 내용이 무엇인지를 정확히 안다면 하나를 선택해서 돌리면 되지만 모른다면 `엔터`를 눌러서 알아서 동작하도록 만들어주면 됩니다. 이후, 시간이 한참 지나면 설치가 완료되었다는 아래와 같은 창이 뜰 것입니다.

![msys-3](/assets/img/res/2019-Blog/msys-3.PNG)

추가적으로 설치하고 싶으신 게 있으시다면 1 ~ 3에 해당하는 숫자를 입력하고 치면 됩니다. 하지만 그렇지 않은 경우에는 그냥 엔터치고 빠져나오시면 됩니다. 그리고 powershell을 아무 데서 열고 다음의 명령어를 쳐보도록 합니다.

```powershell
gem --version
```

이 명령어로 아래와 같은 내용이 출력 되면 정상적으로 설치가 된 것입니다.

```powershell
3.0.3
```

이제 본인들이 가지고 있는 jekyll 블로그의 디렉터리의 최상위 폴더로 이동하도록 합니다. 제 블로그의 경우에는 `blackingj.github.io`이기 때문에 이 폴더로 갑니다. 여기서 아래의 명령을 쳐주도록 합니다.

```powershell
gem install bundler
```

이렇게 bundler를 설치를 완료를 하면 동일하게 `bundle version`을 쳐서 정상 설치가 되었는지를 확인하고,  업데이트를 아래와 같은 명령어를 쳐서 수행해주도록 합니다.

```powershell
bundle update
```

여기까지 했다면 모든 준비가 끝난 것입니다. 이제는 아래와 같은 명령어를 쳐서 실제로 웹사이트에서 어떻게 보여지는 지를 확인하기 위해서 다음의 명령어를 쳐주도록 합니다.

```powershell
bundle exec jekyll serve
```

이때, `wdm`을 설치하라고 뜨면 다음의 명령어를 쳐주시면 됩니다.

```powershell
gem install wdm
```

그러면 다음과 같은 창이 당신을 반겨 줄 것입니다.

![result](/assets/img/res/2019-Blog/result.PNG)

