---
layout: post
title: "[완전탐색]-브루트 포스 순열 조합"
date: 2023-03-07 08:43:59
categories: Coding
tags: [Python]
---

### 브루트 포스 Brute-force

- 브루트 포스 : ‘무차별 대입’이라는 뜻

“N개의 수를 입력 받아 그 중 두 수를 뽑았을 때, 합이 가장 클 때?”

2중 for문을 하는 경우 시간 복잡도 O(N^2)이고 N의 범위가 커지면 시간 초과 발생.

→ 효율적으로 하기 위해 N개의 수를 입력 받아 배열에 넣고 정렬시킨 뒤 가장 큰 값 2개를 뽑으면 됨.

정렬로 인해 시간 복잡도 O(NlogN) , 기본 정렬 함수는 시간 복잡도가 O(NlogN)

[코딩테스트 연습 - 숫자의 표현 | 프로그래머스 스쿨 (programmers.co.kr)](https://school.programmers.co.kr/learn/courses/30/lessons/12924)

```python
# 완전 탐색
def solution(n):
    answer = 1
    
    for i in range(1,n//2+1):
        cnt = 0
        for j in range(i, n):
            cnt+=j
            if cnt == n:
                answer+=1
                break
            elif cnt > n:
                break
            else:
                continue
    return answer
```

### 순열 permutation

- n개의 수 중에서 r개를 뽑아 줄을 세우는 총 방법의 수 : $_nP_k = {n!\over (n-r)!}$
- C++ : next_permutation이라는 STL 사용. do-while 문을 사용.
- Python : itertools의 permutations를 import 후 사용.

```python
from itertools import permutations
arr = [0, 1, 2, 3]
for i in permutations(arr, 4):
	print(i)
```

### 조합 combination

- n개의 수 중에서 r개를 뽑는 가지 수 : $_nC_r = {n! \over (n-r)!r!}$
- C++ :  STL은 없으므로 next_permutation을 이용하거나 재귀함수로 구현.
- Python : itertools의 combination을 import 후 사용.

```python
from itertools import combination
arr = [0, 1, 2, 3]
for i in combination(arr, 2):
	print(i)
```

[백준문제](https://www.acmicpc.net/problem/3040) - [3040] **백설 공주와 일곱 난쟁이**

```python
# 9명의 난쟁이의 키를 합한 다음, 차이를 구한 것.
arr = []
for i in range(9):
    arr.append(int(input()))
arr.sort(reverse=False)
ans = sum(arr) - 100
for i in range(len(arr)):
    for j in range(i+1, len(arr)):
        if arr[i] + arr[j] == ans:
            for k in range(len(arr)):
                if k == j or k == i:
                    pass
                else:
                    print(arr[k])
            exit()
            
# 조합을 사용하여 푼 것

from itertools import combinations
arr = []
for _ in range(9):
    arr.append(int(input()))
    for i in combinations(arr, 7):
        if sum(i) == 100:
            for j in sorted(i):
                print(j)
            break
```