---
layout: post
title: "[컴파일러] 이동-감축 구문분석"
date: 2018-10-21
excerpt: "이동-감축 구문분석에 관해서 알아보도록 한다."
tag:
- compiler
comments: true
---

# 상향식 구문분석

구문분석 트리의 생성 방향에 따라서 구문분석의 방법은 트리가 위에서 아래로 생기는 하향식(Top-Down) 구문분석과 아래에서 위로 생기는 상향식 구문분석(Bottom-Up)으로 나눕니다. 그리고 파서의 구조에 따라서 테이블 드리븐(table-driven) 방식과 하드 와이어(hard-wired) 방식으로 나뉩니다. 이를 정리하면 다음과 같은 표로 정리 할 수 있습니다.

|  | 하향식 | 상향식 |
|:----:|:---:|:---:|
| 테이블 드리븐| LL 구문분석 | LR 구문분석 |
| 하드 와이어 | 재귀 하향 구문분석[^1] | - |
{:.table .table-bordered}

여기서 LR 구문분석을 다른 말로 **이동-감축 구문분석(shift-reduce parsing)**이라고 합니다. LR 구문분석은 유효한 토큰 문자열에 대해서 우측 파스(right parse)에 따른 시작 기호로 변환하는 방법입니다. 여기서 유효한 토큰을 **문장**이라고 합니다.

여기서 **파스(parse)**는 규칙 생성 과정(또는 규칙 번호)에 유도(derivation)를 적용하는 과정이라고 생각하시면 됩니다. 이 중 **우측 파스**는 최우단 유도(rightmost derivation)의 반대 생성규칙 과정이라고 생각하면 됩니다. 예를 들어, 다음과 같은 문법에서 `abbcde` 문자를 찾는 상향식 구문분석이 어떻게 일어나는지 확인해보도록 하겠습니다.

```text
S -> aABe ...............(1)
A -> Abc|b ..............(2),(3)
B -> d ..................(4)
```

이들의 상향식 구문분석은 아래와 같은 순서로 진행됩니다.

| 우측 파스 | 규칙 번호 |
|:----:|:---:|
|`S -> abbcde` |`(3)`|
|`S -> aAbcde` |`(2)`|
|`S -> aAde` |`(4)`|
|`S -> aABe` |`(1)`|
{:.table .table-bordered}

구문분석이 `(3)->(2)->(4)->(1)`로 이루어짐을 알 수 있습니다. 여기서 우리는 시작 기호부터 차례로 감축(reduction)하면서 구문분석을 진행해갔습니다. 그리고 이러한 감축의 순서를 역순으로 하면 **최우단 유도**가 일어남을 알 수 있습니다. **결론적으로 상향식 구문분석의 목적은 유도를 역순으로 구성하는 것에 있습니다.**

## 핸들(Handle)

문자열을 오른쪽에서부터 읽어가는 과정에서 **최우단 유도**의 역순의 생성규칙 중에 일치되어 감축되는 부분 문자열을 보고 핸들이라고 합니다. 이는 논 터미널에 의해서 감축되는 그것으로 생각하면 됩니다.

이러한 핸들이 우측 문장형태의 부분이기 때문에, 이는 세 가지(우측 문장형태, 위치, 부분 문자열 그 자체)로 정의될 수 있습니다. 이러한 문장형태는 시작 기호로부터 유도된 문법 기호의 문자열로 0개에서 그 이상의 유도과정으로 만들어질 수 있습니다. 다음의 유도과정에서 $$\beta$$가 우측 문장형태에서 핸들이 됩니다. 그리고 여기서 $$w$$는 반드시 터미널 기호만 가지고 있어야 합니다. $$*$$은 과정 생략을 의미합니다.

$$\alpha \beta w: S \Rightarrow^{*}_{rm} \alpha A w \Rightarrow^{*}_{rm} \alpha \beta w$$

그리고 감축의 과정을 핸들 대체(handle-pruning)라고 합니다. 위의 식에서 $$\alpha \beta w$$의 실제로 핸들은 $$A \to \beta$$가 핸들이 됩니다. 하지만 우리는 일반적으로 $$\beta$$를 핸들이라고 합니다. 만약 문법이 모호하다면 핸들이 주어진 우측 문장형태에 대해서 유일하지 않을 수 있습니다.

# 이동-감축 구문분석

이동-감축 파서에서 상태(state)는 $$(T, I)$$의 쌍으로 정의됩니다. 여기서 $$T$$는 스택의 내용을 의미하고, $$I$$는 남은 입력을 의미합니다. 이는 때때로 파서의 배치(configuration)라고도 부릅니다. 초기 배치는 $$(\$, w\$)$$로 됩니다. 그리고 최후 배치는 $$(\$S,\$)$$가 됩니다. 여기서 $$\$ $$는 스택의 바닥(bottom)과 입력의 마지막을 의미합니다.

## 연산

이동-감축 파서의 연산은 총 4개가 존재합니다.

1. **이동(shift)**: 현재 입력 기호을 스택에 넣도록 합니다.
2. **감축(reduce)**: 논 터미널의 스택의 최상단(핸들)을 교환하도록 합니다.
3. **수용(accept)**: 입력 문자열이 유효함을 알리도록 합니다.
4. **오류(error)**: 입력 문자열에 문제가 있음을 알리도록 합니다.

다음과 같은 문법 규칙에서 어떻게 이동 감축 구문분석이 어떻게 일어나는지 확인하도록 하겠습니다.

```text
E -> E + T | T -- (1), (2)
T -> T * F | F -- (3), (4)
F -> (E)|id ----- (5), (6)
```

여기서 우리는 $$id_1, id_2$$를 구문분석한다고 하겠습니다. 여기서 행동은 다음에 적용될 명령을 의미합니다.

| 스택 | 입력 | 행동 |
| :----- | -----: |:------:|
|$$ \$ $$|$$id_1 * id_2\$ $$ | shift |
|$$ \$ id_1 $$|$$* id_2\$ $$ | (6) reduce |
|$$ \$ F$$|$$* id_2\$ $$ | (4) reduce |
|$$ \$ T$$|$$* id_2\$ $$ | shift |
|$$ \$ T*$$|$$id_2\$ $$ | shift |
|$$ \$ T*id_2$$|$$\$ $$ | (6) reduce |
|$$ \$ T*F$$|$$\$ $$ | (3) reduce |
|$$ \$ T$$|$$\$ $$ | (1) reduce |
|$$ \$ E$$|$$\$ $$ | accept |
{:.table}

## 실용 접두사

실용 접두사(viable prefix)는 우측 문장형태의 최우단 핸들의 우측 끝을 지나쳐서 진행되지 않는 우측 문장형태의 접두사이고 이는 빈 문자열($$\epsilon$$)도 포함합니다. 이는 이동-감축 파서의 스택에서 우측 형태의 접두사로 나타나게 됩니다. 예를 들어, 위의 문법 규칙을 바탕으로 확인을 해보겠습니다. 어떤 유도가 다음과 같다고 가정을 하도록 하겠습니다.

```text
E => F*id => (E)*id
```

`(E)*id`의 구문 분석 과정에서 스택에 `(, (E, (E)`는 들어갈 수 있지만 `(E)*`은 들어갈 수 없습니다. 왜냐하면 `(E)`가 파서이기 때문입니다. 만약에 이 이상 스택에 넣기 위해서는 감축하도록 해야합니다. 즉, 여기서 스택에 나타날 수 있는 `(, (E, (E)`이 실용 접두사라고 할 수 있습니다.

# 충돌

이동 감축 파서에서 행동을 특정할 수 없는 상황을 보고 충돌(conflict)이라고 합니다. 이는 문법이 모호함과 같은 것들에 의해 부정확한 구문분석 결과나 더 진행하지 못하게 만듭니다. 따라서 이런 경우 충돌을 해결하기 위해 문법을 변경하도록 해야 합니다. 이런 충돌의 종류로는 2가지가 있습니다.

1. **이동-감축 충돌**: 파서가 이동과 감축을 판단하지 못할 때 벌어집니다.
2. **감축-감축 충돌:** 파서가 핸들을 판단하지 못할 때 벌어집니다.

## 이동-감축 충돌

아래와 같은 문법에서 이동 감축 충돌이 벌어지게 됩니다.

```text
S -> if E then S
   | if E then S else S
   | other
E -> true
   | false
```

입력이 `if true then false then other else other`인 경우를 산정하는 경우에 아래와 같은 일이 벌어질 수 있습니다.

```text
($,if true then false then other else other$)
...
($if E then E then S, else other$)
```
여기서 `else other`을 이동을 할 수도 있지만 `if E then E then S`를 감축할 수도 있습니다. 이런 상황을 보고 이동-감축 충돌이라고 합니다. 이에 대한 해결책으로는 `else`가 `then`과 연관이 되어있음을 우리는 알 수 있으므로 입력에 대한 이동을 우선시시켜주는 문법을 추가해줌으로 해결할 수 있습니다.

## 감축-감축 충돌

아래와 같은 문법에서 감축 충돌은 벌어질 수 있습니다.

```text
S     -> id(plist)
       | E := E
plist -> plist, P
       | p
P     -> id
E     -> id(elist)
       | id
elist -> elist, E
       | E
```

이 경우 입력이 `id(id)`인 경우에 `P->id`, `E->id` 중에 뭐가 먼저 실행이 되어야 하는지가 모호하게 됩니다.


# $$LR(k)$$

이는 구문분석 방법의 종류입니다. 'L'은 왼쪽에서 오른쪽으로 스캔을 함을 의미하고, 'R'은 최우단 유도를 역순(우측 파스)으로 구성한다는 것을 의미합니다. 음수가 아닌 정수 $$k$$는 선견(lookahead) 기호의 개수를 의미합니다. 만약 $$LR(k)$$라는 문법이 있다면 $$LR(k)$$ 파서에 의해서 구문 분석이 된 문자열을 생성하라는 것을 의미합니다.

여기서 **선견 기호**는 스캔에 의한 구문분석 행동이 결정되기 전에 가져와 지는 기호을 이야기합니다. 개념은 스캐너와 매우 유사하나 구문 분석된 문맥의 문자라는 것보단 토큰이라 보는 것이 맞습니다.

---
[^1]: recursive-descent parser