---
layout: post
title: "[컴퓨터 구조] CPU Structure and Function"
date: 2018-06-20
excerpt: "CPU가 어떻게 구성되고 동작하는 지를 알아보도록 한다."
tag:
- Computer Architecture
comments: true
---

출처 : Willian Stallings. (2013). Computer Organization and Architecture. London:Pearson

<!-- ![Instruction](/assets/img/res/2018-CompArch/CPU/simpleInstr.png) -->

# CPU Structure and Function

## Structure

CPU는 무조건 fetch(instruction), interpret(instruction), fetch(data), process(data), write(data)의 역할을 할 있어야하고, 이런 CPU의 주요 구성 요소로 ALU, 제어 유닛, 레지스터로 구성됩니다.

이러한 CPU는 임시 공간으로 레지스터라는 것을 가집니다. 이것은 2 종류가 존재하고 이들을 정리하면 아래와 같은 표로 나타낼 수 있습니다.

| User Visible Registers | Control and Status Registers |
|------------------|------------------------|
| General Purpose | Program Counter |
| Data              | Instruction Decoding Register |
| Address           | Memory Address Register |
| Condition Codes | Memory Data Register |
| | Program Status Word(PSW) | 
{:rules="groups"}

여기 있는 레지스터 중 PSW 레지스터의 경우는 현재 상태를 저장하는 것으로 디버깅에 유용하게 만들어 줍니다. 이 연산은 ALU의 결과를 비롯한 각종 결과에 대한 설정을 하게 됩니다.

## Data Flow

데이터의 흐름(Data Flow)을 끊는 데에 인터럽트라는 개념이 있습니다.  이것이 작동을 하면 아래와 같은 절차가 수행됩니다.

1. Program Counter(이하 PC)의 내용이 MBR로 이동하도록 합니다.
2. 특별한 주소 공간(예를 들어, 스택 포인터)이 MAR로 이동됩니다.
3. 제어 유닛 : 쓰기
4. MBR이 메모리에 작성되게 됩니다.
5. PC는 인터럽트를 처리하는 루틴(routine)의 주소를 가져오도록 합니다.

즉, 인터럽트가 왔다는 것은 PC에 따르는 절차를 무시하고 인터럽트를 발생시킨 다른 주소가 먼저 실행될 수 있도록 만듭니다.

이것 외에 데이터 흐름이라고 하면 execute, fetch 등이 있습니다. 하지만 이러한 기존의 방식은 CPU를 완벽하게 활용