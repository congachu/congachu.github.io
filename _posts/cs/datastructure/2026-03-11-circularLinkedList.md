---
title: "[Data Structure] 환형 링크드 리스트"
date: 2026-03-11 17:00:00 +0900
categories: [Data Structure, List]
tags: [Data Structure, List, Circular Linked List]
description: "환형 링크드 리스트 구현"
toc: true
---

## 환형 링크드 리스트 (Circular Linked List)

환형 링크드 리스트는 링크드 리스트의 한 형태로, 리스트의 마지막 노드가 다시 첫 번째 노드를 가리키도록 만든 구조이다.

일반적인 링크드 리스트는 마지막 노드의 `next` 값이 `None`이지만,  
환형 링크드 리스트에서는 마지막 노드가 다시 `head`를 참조한다.

또한 이 구현에서는 **이전 노드와 다음 노드를 모두 저장하는 구조**를 사용했기 때문에  
더블 링크드 리스트 형태의 환형 구조라고 볼 수 있다.

이 구조에서는 다음과 같은 특징이 있다.

- 리스트의 끝이 존재하지 않는다.
- 어떤 노드에서 시작해도 리스트 전체를 순회할 수 있다.
- `head` 또는 `tail` 중 하나만 알고 있어도 전체 접근이 가능하다.

---

## 환형 링크드 리스트 구조

노드는 다음과 같은 형태를 가진다.

```
[ 이전 노드 | 데이터 | 다음 노드 ]
```

마지막 노드와 첫 노드는 서로 연결된다.

```
head <-> ... <-> tail
 ^                 |
 |_________________|
```

---

## 환형 링크드 리스트 주요 연산

- 노드 생성
- 노드 삭제
- 노드 탐색
- 노드 삽입
- 노드 추가

---

## 노드 구현

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next: "Node" = None
        self.prev: "Node" = None
```

---

## 환형 링크드 리스트 클래스 정의

```python
class CircularLinkedList:
    def __init__(self):
        self.head: Node = None
        self.tail: Node = None
        self.size = 0
```

---

## 노드 추가 함수 (append)

```python
def append(self, data):

    new_node = Node(data)

    if self.size == 0:
        self.head = new_node
        self.tail = new_node

    else:
        self.tail.next = new_node
        new_node.prev = self.tail
        self.tail = new_node

    self.head.prev = self.tail
    self.tail.next = self.head

    self.size += 1
    return True
```

---

## 노드 삭제 함수 (remove)

```python
def remove(self, data):

    if self.size == 0:
        return False

    cur_node = self.head
    count = 0

    while count < self.size and cur_node.data != data:
        cur_node = cur_node.next
        count += 1

    if cur_node == None:
        return False

    cur_node.next.prev = cur_node.prev
    cur_node.prev.next = cur_node.next

    self.size -= 1
    return True
```

---

## 노드 삽입 함수 (insert)

```python
def insert(self, data, index):

    if index >= self.size:
        self.append(data)
        return True

    elif index <= 0:

        new_node = Node(data)

        new_node.next = self.head
        new_node.prev = self.tail

        self.head.prev = new_node
        self.head = new_node

    else:

        cur_node = self.head

        for _ in range(index - 1):
            cur_node = cur_node.next

        new_node = Node(data)

        new_node.next = cur_node.next
        new_node.prev = cur_node

        cur_node.next = new_node

    self.size += 1
    return True
```

---

## 노드 탐색 함수 (search)

```python
def search(self, data):

    cur_node = self.head

    for _ in range(self.size):

        if cur_node.data == data:
            return True

        cur_node = cur_node.next

    return False
```

---

## 시간복잡도

| 연산 | 시간복잡도 |
|-----|-----|
| 접근 | O(n) |
| 탐색 | O(n) |
| 삽입 | O(1) |
| 삭제 | O(1) |

(노드를 알고 있는 경우 기준)

---

## 장점

- 어떤 노드에서도 리스트 전체 순회 가능
- 순환 구조가 필요한 알고리즘에 유리
- 큐나 라운드 로빈 스케줄링 등에 활용 가능

---

## 단점

- 구현이 단순 링크드 리스트보다 복잡
- 순환 구조 때문에 종료 조건을 잘 관리해야 함

---

## 참고 자료

- 박상현, 『이것이 자료구조+알고리즘이다』, 한빛미디어