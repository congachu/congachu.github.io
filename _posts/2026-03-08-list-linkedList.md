---
title: "[Data Structure] 리스트와 링크드 리스트"
date: 2026-03-08 23:00:00 +0900
categories: [Data Structure, List]
tags: [Data Structure, List, Linked List]
description: "리스트 개념과 링크드 리스트 구현"
toc: true
---

---

리스트는 우리 삶에서 자주 사용된다.
전화기에 저장된 연락처, 죽기 전에 꼭 해보고 싶은 일을 담은 버킷 리스트, 구매할 물품을 정리한 장보기 목록 등 다양한 형태로 활용된다.

이처럼 유용한 리스트 개념은 소프트웨어 코드에서도 매우 자주 사용된다.

---

# 리스트 (List)

## 리스트의 개념

리스트는 이름에서 유추할 수 있듯이 **목록 형태로 이루어진 데이터 구조**이다.

리스트의 각 요소를 **노드(Node)** 라고 부른다.
노드는 이후 스택, 큐, 트리 등 다양한 자료구조를 공부하면서 계속 등장하는 중요한 개념이다.

리스트에는 다음과 같은 용어가 있다.

* **헤드(Head)** : 리스트의 첫 번째 노드
* **테일(Tail)** : 리스트의 마지막 노드

리스트의 길이는 **헤드부터 테일까지 존재하는 노드의 개수**를 의미한다.

---

# 배열과 리스트 비교

배열 역시 리스트처럼 데이터 목록을 관리할 수 있다.

대부분의 프로그래밍 언어에서는 배열을 기본적으로 제공하기 때문에 별도로 구현하지 않아도 된다는 장점이 있다.

하지만 배열에는 다음과 같은 단점이 있다.

* 배열은 **생성 시 반드시 크기를 지정해야 한다**
* 배열은 **생성 후 크기를 변경할 수 없다**

이 문제를 해결하기 위해 등장한 자료구조가 **리스트**이다.

리스트는 배열처럼 데이터 집합을 저장하면서도 **유연하게 크기를 변경할 수 있는 구조**를 가진다.

또한 리스트는 스택, 큐, 트리와 같은 자료구조를 이해하는 기반이 된다.

---

# 링크드 리스트 (Linked List)

## 링크드 리스트의 개념

링크드 리스트는 리스트를 구현하는 여러 방법 중 **가장 기본적인 방법**이다.

링크드 리스트는 **노드들을 연결(link)하여 만든 리스트**이다.

각 노드는 다음과 같은 구조를 가진다.

```
[ 데이터 | 다음 노드의 주소 ]
```

즉 노드는 다음 두 가지 정보를 가진다.

* 데이터를 저장하는 필드
* 다음 노드를 가리키는 포인터

이 노드들이 연결되어 하나의 리스트를 구성한다.

---

# 링크드 리스트 구현

링크드 리스트의 주요 연산은 다음과 같다.

* 노드 생성
* 노드 삭제
* 노드 탐색
* 노드 삽입
* 노드 추가

---

## 노드 구현

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None
```

---

## 링크드 리스트 클래스 정의

```python
class LinkedList:
    def __init__(self):
        self.head = None
        self.size = 0
        self.tail = None
```

---

## 노드 추가 함수 (append)

```python
def append(self, data):
    new_node = Node(data)

    if self.head == None:
        self.head = new_node
        self.tail = new_node
    else:
        self.tail.next = new_node
        self.tail = new_node

    self.size += 1
```

---

## 노드 삭제 함수 (remove)

```python
def remove(self, data):
    prev = None

    if self.head == None:
        return False

    cur_node = self.head

    while cur_node != None and cur_node.data != data:
        prev = cur_node
        cur_node = cur_node.next

    if cur_node == None:
        return False

    if cur_node == self.head:
        if self.size == 1:
            self.__init__()
            return True
        else:
            self.head = self.head.next
            cur_node.next = None

    elif cur_node == self.tail:
        self.tail = prev
        self.tail.next = None

    else:
        prev.next = cur_node.next

    self.size -= 1
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

## 노드 삽입 함수 (insert)

```python
def insert(self, data, index):

    if self.size <= 0:
        self.append(data)
        return True

    elif index <= 0:
        new_node = Node(data)
        new_node.next = self.head
        self.head = new_node

    elif index >= self.size:
        self.append(data)
        return True

    else:
        cur_node = self.head

        for _ in range(index - 1):
            cur_node = cur_node.next

        new_node = Node(data)
        new_node.next = cur_node.next
        cur_node.next = new_node

    self.size += 1
    return True
```

---

# 전체 코드

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None


class LinkedList:
    def __init__(self):
        self.head = None
        self.size = 0
        self.tail = None


    def append(self, data):
        new_node = Node(data)

        if self.head == None:
            self.head = new_node
            self.tail = new_node
        else:
            self.tail.next = new_node
            self.tail = new_node

        self.size += 1
        return True


    def remove(self, data):
        prev = None

        if self.head == None:
            return False

        cur_node = self.head

        while cur_node != None and cur_node.data != data:
            prev = cur_node
            cur_node = cur_node.next

        if cur_node == None:
            return False

        if cur_node == self.head:
            if self.size == 1:
                self.__init__()
                return True
            else:
                self.head = self.head.next
                cur_node.next = None

        elif cur_node == self.tail:
            self.tail = prev
            self.tail.next = None

        else:
            prev.next = cur_node.next

        self.size -= 1
        return True


    def insert(self, data, index):

        if self.size <= 0:
            self.append(data)
            return True

        elif index <= 0:
            new_node = Node(data)
            new_node.next = self.head
            self.head = new_node

        elif index >= self.size:
            self.append(data)
            return True

        else:
            cur_node = self.head

            for _ in range(index - 1):
                cur_node = cur_node.next

            new_node = Node(data)
            new_node.next = cur_node.next
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
        cur_num = 0

        print("[노드 번호 | 데이터]")

        while cur_node:
            print(f"[ {cur_num} | {cur_node.data} ]")
            cur_node = cur_node.next
            cur_num += 1


if __name__ == "__main__":

    linkedList = LinkedList()

    print(linkedList.append(1))
    print(linkedList.append(2))

    linkedList.print()

    print(linkedList.insert(1.5, 1))
    linkedList.print()

    print(linkedList.insert(3, 9999))
    linkedList.print()

    print(linkedList.insert(-1, -222))
    linkedList.print()

    print(linkedList.search(2))
    print(linkedList.remove(2))
    print(linkedList.search(2))
    print(linkedList.remove(2))

    linkedList.print()

    print(linkedList.remove(1))
    linkedList.print()
```

---

## 참고 자료

* 박상현, 『이것이 자료구조+알고리즘이다』, 한빛미디어
