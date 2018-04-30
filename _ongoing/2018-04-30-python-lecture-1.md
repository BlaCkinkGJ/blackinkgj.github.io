---
layout: post
title: "[파이썬] 1주차 : 기본 연산 및 자료형 "
date: 2018-3-27
excerpt: "기본 연산 및 자료형에 관해서 알아보도록 한다."
tag:
- python
---

## 오늘 배울 내용
* Python의 설치 방법
* 변수
* 자료형
* 연산자
* 응용 예제
* Q & A

## 파이썬의 설치 방법
Python 2.7을 기준으로 설명할 예정이므로 Python 2.7을 설치해 주시길 바랍니다.
설치 방법은 아래의 순서대로 진행을 하면 됩니다.
1. https://www.python.org/downloads/ 링크를 들어가도록 합니다.

2. `Download Python 2.7.14`를 클릭하도록 합니다.
  ![python installation site](/assets/img/res/2018-python/step1.png)
3. 다운로드 된 `python-2.7.14.msi`를 클릭하여 설치하도록 합니다.

4. 설치 파일을 실행을 시키고 `Next`를 누르다가 `Customize Python`이 나오면 제일 하단의 `Add python.exe to Path`에 체크를 하도록 합니다.
  ![add python.exe to path](/assets/img/res/2018-python/step2.png)

5. 종료 후, 명령 프롬프트(혹은 Powershell) 을 켜서 `python --version`을 입력을 하고, `Python 2.7.14`가 뜨면 성공적으로 설치한 것입니다.
  ![python version check](/assets/img/res/2018-python/step3.png)

여기까지 하셨으면 여러분은 성공적으로 파이썬 2.7을 설치한 것입니다.

## 변수
모든 프로그래밍 언어에는 변수라고 있습니다. 먼저 수학에서 정의되는 변수(變數)는 수식에 따라 변하는 값을 의미합니다. 이를 테면,
<center>$$5 = x + 3 \to x = 2$$</center>

<center>$$5 = x + 2 \to x = 3$$</center>

에서 볼 수 있듯 어느 임의의 $$x$$는 수식에 따라 값이 변하는 대상을 지칭 합니다. 그렇다면 이런 