---
title: "[Data Structure] 더블 링크드 리스트"
date: 2026-03-10 21:00:00 +0900
categories: [Data Structure, List]
tags: [Data Structure, List, Doubly Linked List]
description: "더블 링크드 리스트 구현"
toc: true
---

## 더블 링크드 리스트 (Doubly Linked List)

더블 링크드 리스트는 링크드 리스트의 확장 구조로, 각 노드가 이전 노드와 다음 노드를 모두 가리키도록 만든 자료구조이다.

단일 링크드 리스트는 다음 노드만 참조할 수 있기 때문에 헤드에서 테일까지 단방향으로만 이동할 수 있다.  
하지만 더블 링크드 리스트는 이전 노드의 주소도 저장하기 때문에 양방향 탐색이 가능하다.

단, 각 노드가 이전 노드와 다음 노드의 주소를 모두 저장해야 하므로 단일 링크드 리스트보다 더 많은 메모리를 사용한다.

---

## 더블 링크드 리스트 구조

노드는 다음과 같은 구조를 가진다.

```
[ 이전 노드 | 데이터 | 다음 노드 ]
```

각 노드는 다음 정보를 가진다.

- 데이터를 저장하는 필드
- 이전 노드를 가리키는 포인터
- 다음 노드를 가리키는 포인터

---

## 더블 링크드 리스트 주요 연산

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
        self.prev: "Node" = None
        self.next: "Node" = None
```

타입 힌트를 사용해 `Node` 타입을 명시하면 IDE에서 자동 완성이나 타입 인식에 도움이 된다.  
프로그램 성능에는 영향을 주지 않지만 코드 가독성을 높일 수 있다.

---

## 더블 링크드 리스트 클래스 정의

```python
class DoublyLinkedList:
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
        new_node.prev = self.tail
        self.tail.next = new_node
        self.tail = new_node

    self.size += 1
    return True
```

노드가 비어있는지 확인할 때 `self.head == None` 대신 `size == 0`을 사용하면 코드 의도를 조금 더 직관적으로 표현할 수 있다.

---

## 노드 삭제 함수 (remove)

```python
def remove(self, data):

    if self.size == 0:
        return False

    cur_node = self.head

    while cur_node != None and cur_node.data != data:
        cur_node = cur_node.next

    if cur_node == None:
        return False

    if cur_node == self.head:

        if self.size == 1:
            self.__init__()
            return True
        else:
            self.head.next.prev = None
            self.head = self.head.next
            cur_node.next = None

    elif cur_node == self.tail:

        self.tail = cur_node.prev
        self.tail.next = None

    else:

        cur_node.prev.next = cur_node.next
        cur_node.next.prev = cur_node.prev

    self.size -= 1
    return True
```

---

## 노드 삽입 함수 (insert)

```python
def insert(self, data, index):

    if self.size == 0 or index >= self.size:
        self.append(data)
        return True

    elif index <= 0:

        new_node = Node(data)
        self.head.prev = new_node
        new_node.next = self.head
        self.head = new_node

    else:

        cur_node = self.head

        for _ in range(index - 1):
            cur_node = cur_node.next

        new_node = Node(data)

        new_node.next = cur_node.next
        new_node.prev = cur_node

        cur_node.next.prev = new_node
        cur_node.next = new_node

    self.size += 1
    return True
```

---

## 노드 탐색 함수 (search)

```python
def search(self, data):

    cur_node = self.head

    while cur_node != None:

        if cur_node.data == data:
            return True

        cur_node = cur_node.next

    return False
```

---

# 전체 코드

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.prev:Node = None
        self.next:Node = None

class DoublyLinkedList:
    def __init__(self):
        self.head:Node = None
        self.tail:Node = None
        self.size = 0

    def append(self, data):
        new_node = Node(data)
        if self.size == 0:
            self.head = new_node
            self.tail = new_node
        else:
            new_node.prev = self.tail
            self.tail.next = new_node
            self.tail = new_node
        self.size += 1
        return True

    def remove(self, data):
        if self.size == 0:
            return False
        cur_node = self.head
        while cur_node != None and cur_node.data != data:
            cur_node = cur_node.next
        if cur_node == None:
            return False
        if cur_node == self.head:
            if self.size == 1:
                self.__init__()
                return True
            else:
                self.head.next.prev = None
                self.head = self.head.next
                cur_node.next = None
        elif cur_node == self.tail:
            self.tail = cur_node.prev
            self.tail.next = None
        else:
            cur_node.prev.next = cur_node.next
            cur_node.next.prev = cur_node.prev
        self.size -= 1
        return True

    def insert(self, data, index):
        if self.size == 0 or index >= self.size:
            self.append(data)
            return True
        elif index <= 0:
            new_node = Node(data)
            self.head.prev = new_node
            new_node.next = self.head
            self.head = new_node
        else:
            cur_node = self.head
            for _ in range(index - 1):
                cur_node = cur_node.next
            new_node = Node(data)
            new_node.next = cur_node.next
            new_node.prev = cur_node
            cur_node.next.prev = new_node
            cur_node.next = new_node
        self.size += 1
        return True

    def search(self, data):
        cur_node = self.head
        while cur_node != None:
            if cur_node.data == data:
                return True
            cur_node = cur_node.next
        return False

    def print(self):
        if self.size == 0:
            return False
        cur_node = self.head
        print("[노드 번호 | 데이터 | 이전 노드 값 | 다음 노드 값 ]")
        for num in range(self.size):
            print(f"[ {num} | {cur_node.data} | {cur_node.prev} | {cur_node.next} ]")
            cur_node = cur_node.next

if __name__ == "__main__":
    dll = DoublyLinkedList()
    print(dll.append(1))
    print(dll.append(2))
    print(dll.append(3))
    print(dll.append(4))
    dll.print()
    print(dll.search(3))
    print(dll.remove(3))
    print(dll.search(3))
    dll.print()
    print(dll.insert(3, 2))
    dll.print()
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

## 참고 자료

- 박상현, 『이것이 자료구조+알고리즘이다』, 한빛미디어