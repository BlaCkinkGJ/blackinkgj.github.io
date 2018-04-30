---
layout: post
title: "[파이썬] 3주차 : 함수 "
date: 2018-3-27
excerpt: "파이썬의 함수를 사용해보도록 한다."
tag:
- python
---

## 오늘 배울 내용
* 함수란 무엇인가?
* 함수의 기본 사용 방법 
* 함수의 응용 사용 방법
* 연습 문제



## 함수란 무엇인가?

먼저 함수(function)의 정의를 수학적으로 알아봅시다. 수학적으로 함수는 아래와 같이 정의 됩니다.

> A function is a relation that uniquely associates members of one set with members of another set.[^1]

즉, 어떤 원소와 원소간의 관계에 대한 정의라고 볼 수 있습니다. 

이런 수학적인 함수는 대체로 아래와 같은 모양을 띕니다.

<center>$$y = f(x)$$</center>

이것은 어떤 임의의 값 $$x$$ 를 $$f$$라는 함수에 넣으면 $$y$$가 나옴을 저희는 잘 알고 있습니다. 이런 역할을 파이썬을 비롯한 여타 프로그래밍 언어에서의 함수도 수행 합니다. 

그렇다면 이런 함수를 왜 사용할까요? 아래의 프로그램 2개를 비교하면서 알아보도록 하겠습니다.  먼저 함수를 전혀 사용하지 않은 코드입니다.

```python
import random

acc_below = 0
acc_above = 0

lower_limit = 0
higher_limit = 1

iter = 10000
for i in xrange(iter):
    x = random.uniform(lower_limit, higher_limit)
    exp_y = x
    y = random.uniform(lower_limit, higher_limit)
    
    if(y <= exp_y) : acc_below += 1
    else           : acc_above += 1

print acc_below/float(iter)
```

이것의 역할을 위의 코드로는 직관적으로 이해하기 어렵습니다. 다음으로 함수를 사용한 코드를 보도록 하겠습니다.

```python
import random

def my_func(x): return x

def integral(lower_limit, higher_limit, function):
    acc_below = 0
    acc_above = 0
    iter = 1000
    for i in xrange(iter):
        x = random.uniform(lower_limit, higher_limit)
        exp_y = function(x)
        y = random.uniform(lower_limit, higher_limit)
    
        if(y <= exp_y) : acc_below += 1
        else           : acc_above += 1
        
    return acc_below/float(iter)

print integral(0, 1, my_func)
```

이 코드의 역할을 이해하는 데에는 다른 모든 것을 읽을 필요 없이 `print integral(0, 1, my_func)`이 코드만 보면 됩니다. 여기서 우리는 `integral`이 영어로 적분을 의미함을 이미 알고 있습니다. 그렇다면 우리는 괄호 안에 있는 `0, 1, my_func`가 무엇을 의미하는 지 감이 오실겁니다. 저는 이를 보고 먼저 `0, 1`은 적분의 범위일 것이고, `my_func`는 적분의 대상이 되는 (수학적인 의미의)함수일 것이라고 유추하였습니다.

실제로 `integral`이라는 함수가 수행하는 것은 정적분이 맞습니다. 명령어의 나열로 된 위의 코드 같은 경우 그것이 무슨 역할을 하는 지 직관적으로 알기 힘듬에 반해 아래의 코드는 간단한 영어 지식만으로 해당 코드가 어떤 동작을 할 것이라는 것을 유추할 수 있습니다.

그리고 다음의 경우를 생각해보도록 하겠습니다. 지금 현재 저 위의 코드들은 $$y=x$$그래프에 대한 `0 ~ 1`구간의 정적분 입니다. 갑자기 직장 상사가 와서 정적분의 범위를 `0 ~ 2`로 변경하고 $$y = x$$에 한 번 그리고 $$y = x + 1$$에 대해 해달라고 했습니다. 하지만 불행하게도 우리의 코드는 너무 커져버려서  `print` 구문을 제외한 나머지는 찾기도 힘든 상태라고 가정하겠습니다. 그런 경우 위의 명령어의 나열로 구성된 프로그램의 추가ㆍ수정은 열심히 코드 전체를 찾아서 내용을 복사해서 제일 밑에 똑같은 내용을 붙여넣기를 한 후에 적분 범위에 해당하는 변수 값을 바꾼 후에 함수 부분 또 찾은 다음에 수정한 후 변수가 중복되지 않는 지 확인하면서 만들어야 합니다. 말로는 정확히 감이 안잡힌다면 직접 코드로 보여드리겠습니다.

```python
import random

acc_below = 0
acc_above = 0

lower_limit = 0
higher_limit = 2 # 1 -> 2 수정됨

iter = 10000
for i in xrange(iter):
    x = random.uniform(lower_limit, higher_limit)
    exp_y = x
    y = random.uniform(lower_limit, higher_limit)
    
    if(y <= exp_y) : acc_below += 1
    else           : acc_above += 1

print acc_below/float(iter)


acc_below_boss = 0 # boss로 수정됨
acc_above_boss = 0 # boss로 수정됨

lower_limit_boss = 0 # boss로 수정됨
higher_limit_boss = 2 # 1 -> 2 수정됨, boss로 수정됨

iter_boss = 10000
for i in xrange(iter):
    x = random.uniform(lower_limit, higher_limit)
    exp_y = x + 1 # x -> x + 1이 됨
    y = random.uniform(lower_limit, higher_limit)
    
    if(y <= exp_y) : acc_below_boss += 1 # boss로 수정됨
    else           : acc_above_boss += 1 # boss로 수정됨

print acc_below_boss/float(iter_boss)
```

정말 어렵고 이상한 코드가 완성됩니다. 하지만 아래와 같이 함수로 만든 경우 코드의 밑에 아래와 같이 추가 및 수정하면 작업은 끝나게 됩니다.

```python
import random

def my_func(x): return x

def integral(lower_limit, higher_limit, function):
    acc_below = 0
    acc_above = 0
    iter = 1000
    for i in xrange(iter):
        x = random.uniform(lower_limit, higher_limit)
        exp_y = function(x)
        y = random.uniform(lower_limit, higher_limit)
    
        if(y <= exp_y) : acc_below += 1
        else           : acc_above += 1
        
    return acc_below/float(iter)

print integral(0, 2, my_func) # 1 -> 2로 수정됨

def boss_func(x): return x + 1 # 추가

print integral(0, 2, boss_func) # 추가
```

휠씬 짧고 이해하기 쉽습니다. 만약 저런 작업 요청이 10번이 들어오면 아래는 20줄만 추가하면 되지만 위에 것은 아마 족히 100줄이 넘어갈 것입니다. 그리고 추가적으로 변수 이름을 바꾸고 하느라고 무의미하게 많은 시간을 소모하게 될 것입니다.  따라서 함수에 관해서 정리하자면, 아래와 같습니다.

1. 세부적인 구현에 대해 생각을 하지 않고 원하는 기능을 쉽게 사용하기 위해서[^2]
2. 프로그램의 유지 보수를 쉽게 하기 위해서
3. 코드의 재사용성을 높이기 위해서

이제 이런 함수를 어떻게 구현을 하는 지 알아보도록 하겠습니다.



## 함수의 기본 사용 방법

함수는 아래와 같은 구조를 가집니다.

```python
def function(parameter):
    ...
    return something
```

한 번 구조를 차근차근 분석해보도록 하겠습니다. 먼저 `def`라는 키워드는 `define`의 줄임말로 '어떠한 함수를 정의하길...' 이라고 이해하시면 됩니다. 이런 키워드는 모든 함수에서 다 사용됩니다. 이것은 개발자 간의 약속이라고 생각 하시면 됩니다. 다음으로 `parameter` 의 경우 여기에는 함수의 인자가 될 것들이 들어가면 됩니다. 수학에서도 $$f(x, y) = x^2 + y^2$$과도 같은 방식으로 함수를 정의할 수 있듯이, 이 매개변수는 원하는 만큼의 변수를 넣을 수 있습니다. 함수에서 사용될 외부의 값들이 다 들어간다고 생각하시면 됩니다. 그리고 `...`에는 함수가 무슨 역할을 하는 지를 작성하시면 되고, `return something`은 함수에서 계산된 `something` 값을 반환을 합니다. 그런데 `parameter`와 `return`에서 주의할만한 독특한 특징이 있습니다. 저 `return`과 `parameter`는 **없어도 정상적으로 작동합니다.** 심지어 한술 더 떠서 `return`은 **자기 자신을 반환**을 할 수 있습니다. 전자는 그래도 대충 느낌이라도 오는 데 후자는 느낌도 잘 안 올 겁니다. 자세히 뭔 소리인지는 뒤에 응용에서 보도록 하겠습니다. 

먼저 앞에서 이야기한 독특한 특징도 더해서 함수를 작성해서 이해를 해보도록 합시다. 함수의 사용으로 아래와 같은 예가 있을 수 있습니다.

```python 
global_value = "I am global"

def Add(a, b):
    return a + b

def He_said_(what):
    print 'He said, "'+what+'"'

def global_value_says():
    print global_value

print Add(3, 5) # 8 출력
    
He_said_("Hello~")
He_said_("Beautiful!")
He_said_("Awful!")

global_value_says() # I am global 출력
```

먼저 `function`의 이름이 `Add`인 함수를 보면 이는 `parameter`로 `a`와 `b`를 받고 더한 값을 `return` 합니다. 그리고 아래의 `He_said_`와 `global_value_says`를 보면 알 수 있듯이, `return`만 없어도 잘 동작하고 `return`, `parameter` 둘 다 없어도 잘 동작합니다. 이러한 함수를 사용하기 위해서는 **함수의 정의 다음에 오면** 됩니다. 이런 식으로 함수를 작성을 하게 되면 코드를 쉽게 사용할 수 있도록 만들 수 있습니다. 추가적으로 만약 함수 밖에 있는 변수[^3]를 사용하고 싶은 경우에는 **함수의 정의보다 먼저** 변수가 배치하면 사용할 수 있습니다. 이런 작성 방식을 통해 앞선 강의에서 배운 연산자와 변수를 적절하게 조합을 해서 다양한 것을 만들 수 있게 됩니다.



## 함수의 응용 사용 방법

> 이 부분은 기본적인 프로그래밍 언어에 대한 어느 정도 이해가 되었음을 전제로 설명하는 부분입니다.

만약 어렵다고 느끼시면 나중에 보셔도 큰 문제는 없습니다. 파이썬에서 함수의 `parameter`에는 못 들어가는 것이 없습니다. 그렇다면 a와 b의 값을 교환하는 `swap` 함수를 만든다고 생각해봅시다. 그럼, 아래의 함수는 `swap`의 역할을 성공적으로 수행할까요?

```python
def swap(a, b):
    tmp = a
    a = b
    b = tmp

a = 1
b = 2
swap(a, b)
print a
print b
```

결과는 a는 1이 나오고 b는 2가 나옵니다. 그 이유는 파이썬이 **call by object reference**이기 때문입니다.[^4] 만약 이 수업을 듣기 전에 call by value나 call by reference는 들어봤어도 call by object reference는 처음 들을 것입니다. 파이썬은 형에 종속되어 변수가 생성되지 않습니다. 따라서, 함수는 이런 변수나 값을 받을 때 형 추론의 과정을 가지게 됩니다. 이 때, 해당 값이 **immutable(불변의)**이면 값이 우리가 익히 아는 call by value와 같이 인자가 함수에 전달이 되게 됩니다. 이런 immutable한 대표적인 형으로는 **integer, string, tuple**의 경우가 있습니다. 이와 반대로, **mutable**이면 call by reference 처럼 인자가 전달이 되게 됩니다. 이런 mutable한 대표적인 형으로는 **list**가 있습니다.

그렇다면 이런 내용을 아는 상태에서 위를 분석을 하면 위 코드에서 `a`와 `b`는 integer 형으로 immutable입니다. 따라서 swap의 매개변수로 immutable로 지정되어 들어갑니다. 따라서 `a = b`라는 구문은 `a`가 immutable 이므로 매개 변수  `a`가 아닌 또 다른 `a`라는 지역 변수가 생성된 것으로 파악해야 합니다. 동일한 이치로 `b = tmp`는 매개 변수 `b`가 아닌 또 다른 `b`라는 지역 변수가 생성된 것으로 생각행야 합니다.

그렇다면 이런 swap의 구현은 어떻게 해야할까? 결국 immutable의 속성을 유지한 채로 값을 변경하기 위해서는 함수에게 값을 반환(return)을 받아서 해야 함을 알 수 있습니다. 따라서 아래와 같은 방법으로 진행을 할 수 있습니다.

```python
def swap(a, b):
    return (b, a)

a = 1
b = 2
a, b = swap(a, b)
print a
print b
```

`swap`을 immutable하니 tuple로 함수에서 반환을 해서 a, b에 값을 넣을 수 있는 방법으로 위의 문제를 해결 할 수 있습니다.

다음으로 재귀적 용법이 있습니다. 이 방법은 어떤 함수가 자기 스스로를 부르는 방법으로 앞서 언급한 독특한 특징 중 하나입니다. 이는 수학 공식을 구현하는 데 자주 사용되는 방법입니다. 예를 들어,

<center>$$f(x) = f(x-1) + f(x-2)$$</center>

인 피보나치 수열을 구하는 공식을 구현한다고 가정 하겠습니다. 이 경우 간단하게는 반복문으로 구현할 수 있지만, 그러한 구현 방식은 직관적이지 않은 단점이 있습니다. 이 때, 재귀적 용법을 사용하면 코드가 매우 깔끔해지게 됩니다. 먼저 반복문으로 구현한 N번째 피보나치 수를 출력하는 프로그램입니다.

```python
def fibo(N):
    prev = 1
    total = 1
    for i in xrange(N-1):
        temp = total
        total = total + prev
        prev = temp
    return total

print fibo(10)
```

직관적으로 내부 구현을 이해하기가 힘듭니다. 함수의 이름으로는 추론할 수 있을 지 몰라도 쉽게 와닫지는 않습니다. 이에 반해, 재귀적 용법으로 구현을 해보도록 하겠습니다.

```python
def fibo(N):
    if N == 0 or N == 1:
        return 1
    else:
        return fibo(N - 1) + fibo(N - 2)

print fibo(10)
```

재귀적으로 구현을 하면 훨씬 위의 코드에 비해 가독성이 상승을 함을 알 수 있습니다. 하지만 슬프게도 이러한 방법은 심각한 메모리 낭비와 속도의 저하를 일으키게 됩니다. 따라서 상황을 잘 파악해서 적절하게 이를 활용하는 것이 중요합니다.[^5]

그리고 이번 강의에서는 다루지 않고 추후에 다룰 계획인 `lambda` 함수도 미리 연구해보는 것을 추천합니다.



## 연습 문제

당신의 친한 친구가 애용하던 계산기의 연산 버튼이 고장나서 곤란을 겪고 있습니다. 이런 모습을 당신은 마냥 지켜보지 못하고 마침내 계산기를 고쳐주기로 했습니다. 이 친구의 계산기를 분석해보니 버튼이 `add, minus, multi, div, result`의 5개의 버튼이 있으며, 각각의 버튼에 대한 사용법은 `add(arg1, arg2)`,  `minus(arg1, arg2)`, `multi(arg1, arg2)`, `div(arg1, arg2)`, `result(op, arg1, arg2)` 라고 적혀져 있었습니다. 근데, 이 친구의 계산기는 독특한 특징을 가졌는 데 남들이 다 가지고 있는 `print` 버튼이 없었습니다.  예를 들어, 이 친구의 계산기는 다음과 같이 사용 가능합니다.

```python
result(add, 2, 3)
```

위의 경우 결과 값이 **5**가 나오게 됩니다. 



## 각주

[^1]: http://mathworld.wolfram.com/Function.html
[^2]: 이를 보고 추상화(abstraction)이라고 합니다.
[^3]: 이를 보고 전역 변수(global variable)이라고 합니다.
[^4]: https://www.python-course.eu/passing_arguments.php
[^5]: 이유를 알고싶다면 *Computer Systems: A Programmer’s Perspective, 3rd Edition, Pearson* 책을 참고바랍니다.