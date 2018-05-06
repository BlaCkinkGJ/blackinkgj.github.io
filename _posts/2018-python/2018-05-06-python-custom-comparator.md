---
layout: post
title: "[파이썬] 2개 이상의 원소 내용에 대한 정렬 방법"
date: 2018-5-6
excerpt: "2개 이상의 원소를 비교 및 정렬하도록 한다."
tag:
- python
---
# 2개 이상의 원소 내용에 대한 정렬 방법

오늘 파이썬 정렬에 관해서 확인을 하던 도중
재밌는 문제를 발견하게 되었습니다.

>임의의 데이터가 [[4,'c'], [10, 'b'], [13, 'b'], [1, 'd'], [10, 'a']]로 구성되어 있다. 이 내용을 [[13, 'b'], [10, 'a'], [10, 'b'], [4, 'c'], [1, 'd']]과 같은 결과가 나오도록 정렬을 해라.

처음 제가 생각한 방법은 아래와 같았습니다.
```python
# [case 1]
from operator import itemgetter

sample = [[4,'c'], [10, 'b'], [13, 'b'], [1, 'd'], [10, 'a']]

temp = sorted(sample, key=itemgetter(1), reverse = False)
result = sorted(temp, key=itemgetter(0), reverse = True)

print result
```
이 방법은 결과 값은 원하는 데로 나왔습니다. 하지만 이 방식의 문제점으로 추가적으로 변수를 사용해야 한다는 점과  정렬 함수를 두 번이나 실행하는 문제점을 안고 있습니다.

그래서 이를 해결하기 위해 파이썬 레퍼런스를 찾아보았습니다. 파있너에서 기본적으로 제공하는 `sorted`라는 함수는 custom comparator를 제작하는 것을 도와주는 `cmp`라는 parameter가 있는 것을 알게 되었습니다. 따라서, 이를 이용하기로 하였고 결과 아래와 같은 코드를 만들 수 있었습니다.
```python
# [case 2]
#-*- coding:UTF-8 -*-

def compare(x, y):
	if(x[0] < y[0]): # x[0] 값이 y[0]값 보다 작으면
		return 1 # y 내용을 앞으로 보냄
	elif(x[0] > y[0]):
		return -1
	else: # x[0] 값이 y[0]값과 동일하면
		if(x[1] < y[1]): # x[1]과 y[1]을 비교해서 y[1]이 크면
			return -1 # x 내용을 앞으로 보냄
		elif(x[1] > y[1]):
			return 1
		else:
			return 0

sample = [[4,'c'], [10, 'b'], [13, 'b'], [1, 'd'], [10, 'a']]
print sorted(sample, cmp=compare)
```
이렇게 하면 원하는 결과를 낼 수 있습니다. 코드에 대해서 부연 설명을 하자면
`x[0]`는 `[[4, 'c']]`에서 `4`값을 가져오는 역할을 합니다.

이를 테면, **Quick Sort**로 `sorted`가 구현되어 있다고 가정하겠습니다.
파이썬의 Quick Sort로 동일한 구현을 할 수 있습니다.
```python
# [reference]
def partition(A, p, r, c):
    x = A[r]
    i = p - 1
    for j in xrange(p, r):
        if c(A[j], x): # 중요 구문
            i = i + 1
            (A[i], A[j]) = (A[j], A[i])
    (A[i + 1], A[r]) = (A[r], A[i + 1])
    return i + 1

def quicksort(A, p, r, c):
    if p < r:
        q = partition(A, p, r, c)
        quicksort(A, p, q-1, c)
        quicksort(A, q + 1, r, c)

def sorted(list, compare_function):
    temp = list
    p = 0
    r = len(temp) - 1
    quicksort(temp, p, r, compare_function)
    return temp


list = [[4,'c'], [10, 'b'], [13, 'b'], [1, 'd'], [10, 'a']]

def cmp(x, y):
    if(x[0] < y[0]):
        return False
    elif(x[0] > y[0]):
        return True
    else:
        if(x[1] <= y[1]):
            return True
        else:
            return False

print sorted(list, cmp)
```
함수를 전달을 해서 위와 같이 구현이 가능합니다. 여기서 중요한 부분은 `cmp`와 인자로 있는 `c`를 중요하게 보셔야 합니다. `cmp`는 위에서 `compare`와 거의 동일하게 구현되어 있습니다. 하지만 quicksort의 구현 방식을 최대한 변경하지 않는 범위에서 동일하게 만들기 위해서 반환 값을 `boolean`으로 만들었습니다. 

여기서 `중요 구문`을 보면 `c(A[j], x)`로 구성되어 있습니다. 일반적으로 우리가 아는 quicksort는 저 부분이 오름차순이냐 내림차순이냐에 따라서 $$ A[j] \le x $$ 혹은 $$A[j] \ge x$$로 작성되어 있지만 굳이 그렇게 할 필요 없이 저 부분을 `boolean`반환 함수를 넣어서 비교를 진행을 하면 그것에 기초하여 정렬이 됩니다.

이런 방식으로 위의 `sorted`가 구현되어 있다고 생각하시면 되실 것 같습니다. 결과적으로, `[case 2]`같이 작성을 하면 우리가 원하는 2개 이상의 원소에 대해 정렬을 할 수 있음을 알 수 있습니다.

### 추가
Python 3.x의 경우 `sorted`에 `cmp = [compare function]` 이런 식의 사용이 불가능 합니다. 따라서 python 3.x의 경우는 아래와 같이 작성을 해서 수행을 하면 됩니다.
```python
from functools import cmp_to_key

def compare(x, y):
	if(x[0] < y[0]): # x[0] 값이 y[0]값 보다 작으면
		return 1 # y 내용을 앞으로 보냄
	elif(x[0] > y[0]):
		return -1
	else: # x[0] 값이 y[0]값과 동일하면
		if(x[1] < y[1]): # x[1]과 y[1]을 비교해서 y[1]이 크면
			return -1 # x 내용을 앞으로 보냄
		elif(x[1] > y[1]):
			return 1
		else:
			return 0

sample = [[4,'c'], [10, 'b'], [13, 'b'], [1, 'd'], [10, 'a']]
print(sorted(sample, key=cmp_to_key(compare)))
```