---

title: "[CTF] Dream Gallery"
date: 2025-09-19 22:00:00 +0900
categories: [CTF, Web]
tags: [CTF, SSRF, Filter Bypass]
description: "드림핵 Dream Gallery"
author: "송지민"
toc: true
---------

## 문제 개요

이미지 URL을 받아 서버가 직접 가져와(base64로 인코딩) 갤러리로 보여주는 웹앱이다. `file://` 스킴과 `flag` 문자열을 **문자열 포함 여부**로만 필터링하고, 입력 URL을 `.lower()`로 소문자화한다. 이때 **`file:` 스킴 변형**과 **퍼센트 인코딩**을 이용하면 필터를 우회해 로컬 파일(`flag.txt`)을 읽어올 수 있다.

* 요청 엔드포인트: `/request?url=<URL>&title=<TITLE>`
* 필터 조건: `url == ''` **또는** `url.startswith('file://')` **또는** `'flag' in url` **또는** `title == ''`
* 처리 흐름: 차단 조건이 아니면 `urlopen(url).read()` → base64 인코딩 → `mini_database`에 `{title: base64}` 추가 → `/view`에서 이미지로 렌더링

---

## 문제에 제공된 원본 소스코드 전체

```python
from flask import Flask, request, render_template, url_for, redirect
from urllib.request import urlopen
import base64, os

app = Flask(__name__)
app.secret_key = os.urandom(32)

mini_database = []


@app.route('/')
def index():
    return redirect(url_for('view'))


@app.route('/request')
def url_request():
    url = request.args.get('url', '').lower()
    title = request.args.get('title', '')
    if url == '' or url.startswith("file://") or "flag" in url or title == '':
        return render_template('request.html')

    try:
        data = urlopen(url).read()
        mini_database.append({title: base64.b64encode(data).decode('utf-8')})
        return redirect(url_for('view'))
    except:
        return render_template("request.html")


@app.route('/view')
def view():
    return render_template('view.html', img_list=mini_database)


@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if request.method == 'POST':
        f = request.files['file']
        title = request.form.get('title', '')
        if not f or title == '':
            return render_template('upload.html')

        en_data = base64.b64encode(f.read()).decode('utf-8')
        mini_database.append({title: en_data})
        return redirect(url_for('view'))
    else:
        return render_template('upload.html')


if __name__ == "__main__":
    img_list = [
        {'초록색 선글라스': "static/assetA#03.jpg"}, 
        {'분홍색 선글라스': "static/assetB#03.jpg"},
        {'보라색 선글라스': "static/assetC#03.jpg"}, 
        {'파란색 선글라스': "static/assetD#03.jpg"}
    ]
    for img in img_list:
        for k, v in img.items():
            data = open(v, 'rb').read()
            mini_database.append({k: base64.b64encode(data).decode('utf-8')})
    
    app.run(host="0.0.0.0", port=80, debug=False)
```

---

## 내가 실제로 푼 방법

핵심은 **두 가지 우회 포인트**다.

1. `file://`만 차단 → `file:/...`은 허용

* 필터는 `url.startswith('file://')`만 검사한다. `file:/fl%61g.txt`처럼 슬래시 하나(`file:/`)를 쓰면 차단에 걸리지 않는다.

2. `'flag' in url` 검사 → 퍼센트 인코딩으로 우회

* `'flag'` 부분을 `fl%61g`로 쓰면(\`%61`= 'a'),`.lower()`이후에도 여전히`fl%61g`이므로 `'flag'\` 문자열 검색에 걸리지 않는다.

이 두 가지를 결합해 **로컬 파일 스킴 + 퍼센트 인코딩**으로 필터를 통과시킨다.

### 사용한 최종 입력

* 요청 URL (예시):

  ```
  /request?url=file:/fl%61g.txt&title=flag
  ```

  * `file:/...` : 슬래시 하나로 시작하는 file 스킴(차단 회피)
  * `fl%61g.txt` : 'flag.txt'에서 'a'만 퍼센트 인코딩하여 문자열 필터 회피

요청이 성공하면 서버는 `flag.txt` 내용을 읽어 base64로 인코딩하여 `mini_database`에 추가하고, `/view` 화면에서 **이미지로 렌더링**을 시도한다. 텍스트 파일을 이미지로 렌더링하므로 화면에는 **깨진 이미지**가 보이지만, 실제로는 base64 데이터에 FLAG가 들어있다.

---

## 재현 흐름

1. 공격자: `/request?url=file:/fl%61g.txt&title=flag` 요청
2. 서버: 차단 로직 통과 → `urlopen()`으로 로컬 파일 읽기 → base64 인코딩 → `mini_database`에 저장
3. 브라우저: `/view`에서 추가된 항목을 이미지로 표시(텍스트라 깨진 이미지)
4. 공격자: 개발자도구 요소 선택 → 해당 `<img>`의 `src`에서 base64 문자열 복사 → base64 디코딩 → FLAG 확인

> **팁(디코딩 예시)**: 터미널에서
>
> ```bash
> echo "<복사한_base64>" | base64 -d
> ```
>
> 또는 파이썬
>
> ```python
> import base64
> print(base64.b64decode("<복사한_base64>").decode())
> ```

---

## 왜 이게 가능했나

* `file://`만 정확히 일치로 차단하고 `file:/`, `file:///` 등 변형을 고려하지 않았다.
* `'flag' in url` 같은 **순수 문자열 포함 검사**는 퍼센트 인코딩 하나로 쉽게 회피된다.
* 서버가 URL의 **스킴/경로를 엄격히 검증하지 않고** 그대로 `urlopen()`에 넘겨 로컬 파일 시스템을 읽을 수 있었다.

---

## 마무리

Dream Gallery는 `file:` 스킴과 문자열 필터를 허술하게 처리해 **로컬 파일 읽기**가 가능한 전형적인 케이스였다. `file:/fl%61g.txt`로 요청해 `/view`에 올라온 base64를 디코딩하면 FLAG를 얻을 수 있다.