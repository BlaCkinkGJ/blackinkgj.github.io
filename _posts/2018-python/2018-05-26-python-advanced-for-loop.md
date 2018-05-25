---
layout: post
title: "[파이썬] 2차원 리스트의 for문 접근"
date: 2018-5-26
excerpt: "for문으로 2차원 리스트의 효율적인 접근 방법"
tag:
- python
---
# 2차원 배열의 for문 접근

*주의 ! 이 문서는 파이썬 2.x 버전을 기준으로 작성되었습니다.*

python에서 2차원 리스트를 다루는 것은 Numpy를 사용하지 않는다고 가정하는 경우 다른 언어에 비해 쉽다고 말하기는 어렵습니다.

이를 테면, `A = [[1, 2, 3], [4, 5, 6, 7]]`이라는 2차원 리스트가 있다고 생각하는 경우, 이 값을 접근을 하는 데 있어서 가장 쉬운 방법은 아래와 같은 방법으로 접근하는 것이라 할 수 있습니다.

```python
for i in A:
	for j in i:
		print j,
```
이 경우 출력은

```text
1 2 3 4 5 6 7
```
로 나오게 됩니다.

하지만 저 방식을 통해서는 A의 원소의 값을 가져올 수는 있지만 수정을 할 수 없다는 단점이 있습니다.

그렇다면, 값을 변경을 하고 싶을 때의 접근 방법은 어떻게 하는 것이 좋을까? 첫 번째는 `len`을 사용해서 푸는 방법 입니다. `len`을 사용해서 먼저 A 안에 존재하는 리스트의 갯수를 구한 후에 그 값을 바탕으로 각각의 리스트에 접근을 하는 방법이 있습니다. 아래와 같은 방식으로 구현이 될 수 있습니다.

```python
for i in xrange(len(A)):
	for j in xrange(len(A[i])):
		A[i][j] = 1

print A
```

이렇게 A 값의 수정을 하여 아래와 같은 결론이 나오게 됩니다.
```text
[[1, 1, 1], [1, 1, 1, 1]]
```

하지만 위의 코드는 조금 지저분 하고, 명확히 그 내용을 알기가 어렵습니다. 그래서 제가 추천하는 방식은 아래와 같습니다.

```python
for idx_i, val_i in enumerate(A):
	for idx_j, val_j in enumerate(val_i):
		print val_j,
		A[idx_i][idx_j] = 1
print A
```

이렇게 enumerate를 사용하면 가독성이 좀 더 향상되면서 인덱스와 값이 나누어지게 되어, 불필요한 A값의 변경을 막습니다. 그리고 이와 동시에 원하는 경우 값의 수정도 진행을 할 수 있게 됩니다. 저 프로그램을 실행하면 아래와 같은 결론이 나오게 됩니다.

```text
1 2 3 4 5 6 7
[[1, 1, 1], [1, 1, 1, 1]]
```

물론 이 방식은 제가 선호하는 방식이기는 하지만
원하시면 이 외에도 다양한 방법이 파이썬에는 존재합니다.

결과적으로, 파이썬에는 사실 하나의 단순 작업 처리에도 매우 다양한 방법이 존재하기 때문에 본인에 가장 맞는 방법 하나를 본인의 것으로 하시면 됩니다.