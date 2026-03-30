---
title: "[CTF] 드림핵 XSS Filtering Bypass Advanced"
date: 2025-09-19 21:45:00 +0900
categories: [CTF, Web]
tags: [CTF, XSS]
description: "드림핵 XSS Filtering Bypass Advanced"
toc: true
---

## 문제 개요

이번 문제는 XSS 필터가 더 강화된 버전이다. 서버는 입력값에 대해 1차로 `script`, `on`, `javascript` 문자열을 검사하여 포함되면 바로 차단(`filtered!!!` 반환)하고, 이어서 `window`, `self`, `this`, `document`, `location`, `(`, `)`, `&#` 같은 추가 키워드가 포함되어도 차단한다. 단순 문자열 포함 검사이므로, 입력 문자열을 분리하거나 공백/문자열 연결을 이용하면 우회가 가능하다.

* 취약 페이지: `/vuln?param=...`
* 필터링 방식: 특정 문자열 포함 시 즉시 차단
* 목표: `/flag`에 XSS 페이로드 제출 → FLAG 쿠키 탈취

---

## 문제에 제공된 원본 소스코드

```python
def xss_filter(text):
    _filter = ["script", "on", "javascript"]
    for f in _filter:
        if f in text.lower():
            return "filtered!!!"

    advanced_filter = ["window", "self", "this", "document", "location", "(", ")", "&#"]
    for f in advanced_filter:
        if f in text.lower():
            return "filtered!!!"

    return text

@app.route("/vuln")
def vuln():
    param = request.args.get("param", "")
    param = xss_filter(param)
    return param
```

---

## 내가 실제로 푼 방법

강화된 필터는 문자열이 포함되면 즉시 차단하는 방식이다. 하지만 필터는 **연속된 키워드 검사**이므로 키워드 내부에 공백을 삽입하거나 문자열을 분할·연결하면 필터에 걸리지 않는다. 이를 이용해 브라우저에서 실행 가능한 컨텍스트를 만들면 XSS를 달성할 수 있다.

### 핵심 아이디어

* `javascript` 및 `document`, `location` 등의 키워드에 공백을 섞어 `javascript`나 `document` 라는 완전한 문자열이 보이지 않게 한다.
* HTML 속성(예: `src`)에 `javascript:` 스킴을 넣을 때도 공백을 끼워넣으면 필터를 우회할 수 있다.
* iframe을 이용해 `src`에 `javascript:` 스킴을 넣어 스크립트를 실행시켰다.

### 사용한 최종 페이로드

(공백은 실제로는 ASCII 공백 한 칸을 넣어 사용함)

```html
<iframe src="javas cript: locatio n.href='/memo?memo='+docu ment.cookie;"></iframe>
```

* `javascript` → `javas[space]cript` 로 분할하면 `javascript` 키워드가 텍스트에 직접 포함되지 않음
* `location` → `locatio[space]n`, `document` → `docu[space]ment` 등으로 분리하여 advanced\_filter의 검사도 우회
* iframe의 `src`에 스킴 형태로 넣으면 브라우저가 이를 자바스크립트로 해석하여 즉시 실행함

> 주의: 브라우저/버전별로 `iframe src="javascript:..."` 동작이 다를 수 있으나, 이 문제 환경(내부 headless Chrome)에서는 동작하여 우회에 성공했다.

---

## 재현 흐름

1. 공격자: `/flag` 페이지에서 POST로 `param`에 위 페이로드 제출
2. 서버: 내부 Selenium 기반 봇이 `/vuln?param=...`에 접속하여 필터 검증
3. 봇 브라우저: 필터를 통과한 페이로드를 실행 → `document.cookie`를 `/memo?memo=`로 전송
4. 공격자: `/memo`에서 기록된 쿠키를 확인하여 FLAG 획득

---

## 왜 이게 가능했나

* 필터는 단순 문자열 포함 검사만 수행하며, 공백이나 문자열 분리·연결 같은 문법적 우회를 고려하지 않았다.
* 자바스크립트 문법에는 식별자를 대체할 수 있는 여러 표현(대괄호 표기, 문자열 연결, 공백 삽입 등)이 있어 필터를 우회할 여지가 많다.

---

## 마무리

이번 문제는 단순 문자열 포함 검사 방식의 필터가 얼마나 쉽게 우회될 수 있는지를 보여준다. 실무에서는 입력 필터링만으로 XSS를 막으려 하기보다는, 출력 시 컨텍스트에 맞는 이스케이프(HTML 엔티티 인코딩)를 적용하거나 CSP 같은 방어를 병행하는 것이 안전하다.
