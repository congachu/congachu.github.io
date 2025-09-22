---
title: "[CTF] 드림핵 file-download-1"
date: 2025-09-19 21:10:00 +0900
categories: [CTF, Web]
tags: [CTF, File Vulnerable]
description: "드림핵 file-download-1"
author: "송지민"
toc: true
---

## 문제 개요

이번 문제는 간단한 메모(파일) 작성·읽기 웹 앱이다. 글을 작성하면 `uploads/` 디렉토리에 파일로 저장하고, `/read?name=제목`으로 해당 파일을 읽어온다. 제목에 `..`을 포함하는 것을 금지했지만, 읽기 로직에서 파일 경로 조합을 그대로 사용하기 때문에 디렉터리 탈출을 통해 서버 내 다른 파일을 읽을 수 있다.

* 업로드 엔드포인트: `/upload` (POST 폼으로 `filename`, `content` 전송)
* 읽기 엔드포인트: `/read?name=...` (업로드된 파일을 `uploads/{name}`에서 읽음)
* FLAG 위치: `flag.py` (애플리케이션 루트, 소스와 동일한 경로)

---

## 문제에 제공된 원본 소스코드

아래는 문제에서 제공된 `add.py` 전체 소스코드이다.

```python
#!/usr/bin/env python3
import os
import shutil

from flask import Flask, request, render_template, redirect

from flag import FLAG

APP = Flask(__name__)

UPLOAD_DIR = 'uploads'


@APP.route('/')
def index():
    files = os.listdir(UPLOAD_DIR)
    return render_template('index.html', files=files)


@APP.route('/upload', methods=['GET', 'POST'])
def upload_memo():
    if request.method == 'POST':
        filename = request.form.get('filename')
        content = request.form.get('content').encode('utf-8')

        if filename.find('..') != -1:
            return render_template('upload_result.html', data='bad characters,,')

        with open(f'{UPLOAD_DIR}/{filename}', 'wb') as f:
            f.write(content)

        return redirect('/')

    return render_template('upload.html')


@APP.route('/read')
def read_memo():
    error = False
    data = b''

    filename = request.args.get('name', '')

    try:
        with open(f'{UPLOAD_DIR}/{filename}', 'rb') as f:
            data = f.read()
    except (IsADirectoryError, FileNotFoundError):
        error = True


    return render_template('read.html',
                           filename=filename,
                           content=data.decode('utf-8'),
                           error=error)


if __name__ == '__main__':
    if os.path.exists(UPLOAD_DIR):
        shutil.rmtree(UPLOAD_DIR)

    os.mkdir(UPLOAD_DIR)

    APP.run(host='0.0.0.0', port=8000)
```

---

## 내가 실제로 푼 방법

문제 핵심은 **업로드 시에는 `..`을 검사하지만, 읽을 때는 파라미터를 그대로 `uploads/{name}`에 결합해서 파일을 연다는 점**이다. 따라서 읽기 파라미터를 이용해 상대경로(`..`)를 포함시키면 `uploads/../<파일>` 형태가 되어 애플리케이션 루트의 파일을 읽을 수 있다.

이 서비스에서는 FLAG가 `flag.py`에 정의되어 있으므로, 다음과 같이 요청하면 flag.py의 내용을 읽어올 수 있다.

### 내가 사용한 요청

* URL:

  ```
  /read?name=../flag.py
  ```

### 결과

* 서버가 `uploads/../flag.py` 파일을 열어 그 내용을 `read.html` 템플릿에 그대로 표시했다.
* `flag.py` 내부에 정의된 FLAG 문자열을 확인하여 문제를 풀었다.

---

## 재현 흐름

1. 공격자: 브라우저에서 `/read?name=../flag.py` 요청
2. 서버: `open(f'uploads/{filename}', 'rb')` → `open('uploads/../flag.py', 'rb')`가 되어 상위 디렉토리의 `flag.py`를 읽음
3. 서버: 파일 내용을 템플릿에 출력 → FLAG 노출

---

## 왜 이게 가능했나

* 업로드 시에는 `filename`에 `..`을 금지했지만, 읽기 파라미터(`name`)에는 동일한 검증이 적용되지 않았다.
* 읽기에서는 입력값을 검증하거나 정규화하지 않고 `uploads/{name}`으로 바로 결합해 파일을 열었다. 이로 인해 상대경로를 포함한 입력이 그대로 경로로 해석되어 디렉터리 탈출이 발생했다.

---

## 마무리

간단한 파일 읽기 기능에서도 입력값 검증을 한 곳에서만 적용하고 다른 경로에 적용하지 않으면 이런 취약점이 발생한다. 이번 문제는 `read` 파라미터를 통해 `../flag.py`를 읽어 FLAG를 획득하는 전형적인 directory traversal / file-download 문제였다.
