---
layout: post
title:  "스택 큐 우선순위 큐 최대힙 최소힙"
date:   2015-04-18T14:25:52-05:00
author: Ben Centra
categories: Jekyll
tags:	jekyll welcome
cover:  "/assets/instacode.png"
---
<!-- ---
layout: single
title:  "스택 큐 우선순위 큐 최대힙 최소힙"
excerpt: "알고리즘 코딩 테스트 책 정리"
categories: coding
tag: [stack, queue, Python]
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
sidebar:
  nav: "docs"
--- -->


# 스택(stack)
- FILO : 가장 처음에 넣은 데이터가 가장 마지막에 빠짐
- LIFO : 가장 마지막에 넣은 데이터가 가장 먼저 빠짐
- DFS 등 다른 알고리즘에서 사용되는 자료구조
- 넣고 뺄 때 시간 복잡도는 O(1), N개라면 O(N)
- 파이썬에서 기본 자료형인 리스트로 구현
append()를 쓰면 삽입
pop()을 쓰면 삭제

# 큐(queue)
  - FIFO : 가장 처음에 넣은 데이터가 가장 먼저 빠짐
  - LILO : 가장 마지막에 넣은 데이터가 가장 마지막에 빠짐
  - BFS에서 쓰이는 자료구조
  - 넣고 뺄 때 시간 복잡도 O(1), N개라면 O(N)
  - 파이썬에서 `from queue import Queue`
  `q = Queue()` 에서 멀티스레딩 환경까지 고려해서 스레드 간에 안전한 방식으로 동작하는 모듈
  굳이 이런 모듈을 사용하지 않고 덱을 많이 사용
      
      `from collections import deque`
    
  - 덱 : Double - Ended Queue의 약자  
    덱은 앞뒤 구분없이 어느 쪽으로든 넣고 뺄 수 있다.
        
# 우선순위 큐(Priority Queue)
  - 내부적으로 힙(heap)이라는 완전 이진 트리 (complete binary tree)으로 되어 있다.
  - 힙 트리의 루트 노드(root node)에는 데이터들 중에 가장 큰 값(최대 힙) 혹은 가장 작은 값(최소 힙)을 담게 됨.
  - 항상 루트에 가장 큰 값 또는 가장 작은 값이 위치하도록 자동으로 갱신
  - 시간 복잡도 O(logN)
  - 힙정렬(heap sort) : 무작위로 숫자를 힙에 전부 넣고 하나씩 빼면 정렬된 결과를 얻을 수 있음.
  `from queue import PriorityQueue` : 멀티스레딩 환경까지 고려해서 안전한 방식
  - `import heapq` : 멀티스레딩 환경을 고려하지 않은 대신 더 빠른 모듈
  리스트를 하나 만들고 그 안에 데이터를 저장하는 방식
  `heapq.heappush()`로 값을 넣고 인자로 리스트와 넣어줄 값을 전달.
  값을 뺄 때는 `heapq.heappop()`사용
  - `heapq.heappush(heap, item)` : item을 heap에 추가.
  - `heapq.heappop(heap)` : heap에서 가장 작은 원소를 pop & return. 비어 있는 경우 IndexError
  - `heapq.heapify(x)` : 리스트 x를 즉각적으로 heap으로 변환.

# 최대힙과 최소힙
  - Python의 heapq는 최소힙, c++의 priority_queue는 최대힙
  - Python에서 최대힙을 만들고자 할 때, 부호를 바꿔 저장.
  - 데이터를 넣기 전에 전부 부호를 반대로 바꿔서 넣으면 순서가 뒤집혀 저장되므로 최대힙
  - pop한 값은 다시 부호를 바꾸면 원래의 값

[프로그래머스 [더 맵게] 문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/42626#)

```python
# 정확도 테스트 100점, 효율성 테스트 0점
def solution(scoville, K):
    answer = 0
    d = 0
    for i in scoville:
        if i>=K:
            d += 1
        if d == len(scoville):
            return 0
    
    while 1:
        scoville=sorted(scoville)
        scoville.append(scoville[0]+(scoville[1]*2))
        scoville.pop(0)
        scoville.pop(0)
        answer+=1
        c = 0
        if len(scoville) <=1 and scoville[0] < K:
            answer = -1
            break
        for i in scoville:
            if i>=K:
                c += 1
        if c == len(scoville):
            break
    return answer

# --- 정답 ---
# heapq을 사용해야함.
import heapq

def solution(scoville, K):
  heapq.heapify(scoville)

  answer = 0
  if scoville[0] >= K:
    return answer

  while scoville[0] < K:
    if len(scoville) == 1:
      return -1

    min_scoville = heapq.heappop(scoville)
    min2_scoville = heapq.heappop(scoville)

    heapq.heappush(scoville, min_scoville + min2_scoville*2)
    answer += 1

  return answer
```