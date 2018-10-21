---
layout: post
title: "[컴파일러] 정규 언어"
date: 2018-10-21
excerpt: "정규 언어에 관해서 알아보도록 한다."
tag:
- compiler
comments: true
---

문법을 설계하는 데 있어서 모듈화는 좋은 전략입니다. 문법 표현식을 설계하는 경우에 숫자와 식별자는 터미널의 요소로 나옵니다. 이를테면, 아래와 같이 가능합니다.

```text
E -> E + T | E - T | T
T -> num | id
```

여기서 `num, id`는 모듈화된 문법입니다. 이런 모듈화된 설계 전략은 수행문(statement), 함수, 그 외 등등의 문법을 설계할 때 적용 할 수 있습니다.

```text
S -> if (expr) S
   | while (expr) S
   | expr;
```

# 정규 표현식

어휘 분석(lexical analysis)은 컴파일의 첫 번째 과정입니다. 어휘 분석 기능은 문자열에서 단어를 뽑아내기 때문에 다른 과정에 비교해 쉽습니다. 이러한 어휘 분석의 핵심 이론은 **정규 표현식(regular expression)**입니다. 정규 표현식에서 입력은 문자로 구성된 문자열이고 출력은 문자열의 속성이 있는 토큰(token with attributes)입니다. 이는 파서에 의해 요구되는 토큰을 반환합니다. 즉, 아래와 같은 모델을 가지고 있다고 할 수 있습니다.

![lexical](/assets/img/res/2018-compiler/5/lexical.png)

## 용어

각 용어에 관해서 알아보도록 하겠습니다.

- **토큰(token)**: 같은 종류의 단어들 집합을 의미합니다. (e.g. $$id$$와 $$num$$이 있습니다.)
- **어휘 항목(lexeme)**: 토큰 안에 있는 단어를 의미합니다. (e.g. "123"은 $$num$$의 어휘 항목입니다.)
- **패턴(pattern)**: 토큰에 의해 기술(記述)되는 사항을 의미합니다.

## 형식 언어의 기초

먼저 정규 표현식을 알기 위해서는 형식 언어를 제대로 알아야 합니다. 형식 언어는 기호, 알파벳, 문자열, 언어의 구성함을 지난 시간에 배웠습니다. 여기서 문자열의 하위 항목으로는 접두사(prefix), 접미사(suffix), 부분 문자열(substring), 부분 열(subsequence)이 있습니다. 그러면 여기서 "something"이라는 단어의 접두사, 접미사, 부분 열을 찾아보도록 하겠습니다. 그리고 $$\epsilon$$이나 $$\lambda$$는 **공집합(empty set)이 아니라 빈 문자열(empty string)을 의미**합니다.

```text
# of prefix : 9
[ε, s, so, som, some, somet, someth, somethi, somethin]
# of suffix : 9
[ε, g, ng, ing, hing, thing, ething, mething, omething]
# of substring : 45
[ε, s, so, ..., o, om, ome, ....]
# of subsequence : 512
[ε, s, so, sm, se, ...]
```

여기서 유의할 점은 부분 문자열하고, 부분 열은 다르다는 것입니다. 부분 문자열은 **문자열에서 문자가 연속**이 되어야 하지만 부분 열은 **연속될 필요가 없습니다.** 따라서 전자의 개수는 $${n(n+1) } \over {2}$$이 되고, 후자의 개수는 $$2^n$$이 됩니다. 그리고 혹여 진(proper)부분집합을 찾는다고 할 때는 구한 개수들에서 $$-1$$만 해주시면 됩니다.

그리고 이러한 문자열들에 대해서 수행할 수 있는 연산으로 2가지가 존재합니다.

- 결합(concatenation): 문자열 $$s$$, $$t$$에 대해서 결합하는 표시는 $$st$$와 같이 합니다.
- 거듭제곱법(exponentiation): 문자열 $$s$$와 $$s$$를 결합하는 경우 $$s^2$$라고 표기합니다.

## 언어 연산들

언어도 하나의 집합입니다. 그러므로 일반적인 집합 연산을 그대로 사용할 수 있습니다. 그리고 이러한 언어 연산으로 문자열 연산은 확장될 수 있습니다.

결합 $$LM$$은 문자열들의 결합으로 이루어진 언어라고 할 수 있습니다.

$$LM =\{st\mid s \in L \cap t \in M\}$$

클레이니 폐쇄(Kleene closure) $$L^*$$은 0 또는 그 이상의 결합에 대한 합집합이라 할 수 있습니다.

$$L^* = \bigcup_{n = 0}^{\infty}L^n$$

양의 폐쇄(positive closure) $$L^+$$는 1 또는 그 이상의 결합에 대한 합집합이라 할 수 있습니다.

$$L^+ = \bigcup_{n = 1}^{\infty}L^n$$

## 정규 표현식

**정규 언어(regular language)**는 Type 3언어로 언어를 표현하는 방법으로 정규 표현식으로 표현된 언어를 지칭합니다. $$L(r)$$은 정규 표현 $$r$$로 쓰인 언어를 의미합니다. 이에 따라서 몇몇 정규 표현식의 정의들을 확인해보겠습니다.

- $$\epsilon : L(\epsilon) = \{\epsilon\}$$&nbsp;
- $$a : L(a) = {a}$$&nbsp;
- $$r\mid s : L(r\mid s)=L(r) \cup L(s)$$&nbsp;
- $$rs : L(rs) = L(r) \cap L(s)$$&nbsp;
- $$r^* : L(r^*) = \{L(r)\}^*$$&nbsp;

아래는 정규 표현에 대한 언어 추측의 예시입니다.

- $$a\mid b = \{a, b\}$$&nbsp;
- $$(a\mid b)(a\mid b) = {aa,ab,ba,bb}$$&nbsp;
- $$(a\mid b)^* = \{\epsilon, a,b,aa,bb,ab,ba,bb, \cdots \}$$&nbsp;
- $$(a\mid b)^* {a} =$$ 문자열의 마지막이 $$a$$로 끝납니다.
- $$(a\mid b)^* {abb} =$$ 문자열의 마지막이 $$abb$$로 끝납니다.

이러한 정규표현식을 가지고 표현식에 이름을 부여하는 방식으로 정규 정의(regular definition)합니다. 정규 정의는 **재귀적으로 정의**될 수 없습니다. 아래가 정규 정의의 예시입니다.

```text
digit -> 0|1|...|9
digits -> digit digit*
opt_frac -> .digits|ε
opt_expo -> (E(+|-|ε)digits)|ε
num -> digits opt_frac oppt_expo
```

이런 정규 표현 방법 중에서 짧게 표현하는 방법들이 몇 개가 있습니다.

- $$r^+ : rr^*$$&nbsp;
- $$r? : (r\mid \epsilon)$$&nbsp;
- $$[abc]: a\mid  b\mid c$$&nbsp;
- $$[a-z]:a\mid b\mid  \cdots\mid z$$&nbsp;

이에 따라, 위의 내용을 재작성하면 아래와 같이 작성 가능합니다.

```text
digit -> [0-9]
digits -> digit+
opt_frac -> (. digits)?
opt_expo -> (E(+|-)?digits)?
num -> digits opt_frac opt_expo
```

그리고 몇몇 정규표현으로 우리가 표현할 수 없는 언어들이 있습니다. 예를 들어, $$\{wcw \mid  w \in L(a\mid b)^* \} \ni abbcabb$$와 같은 것은 정규 표현으로 정의할 수 없고 문맥 자유 문법(Type 2)에 따라서 정의할 수 있습니다.