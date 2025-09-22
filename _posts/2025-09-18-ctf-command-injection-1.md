---
title: "[CTF] 드림핵 command-injection-1"
date: 2025-09-18 19:00:00 +0900
categories: [CTF, Web]
tags: [CTF, command-injection]
description: "드림핵 command-injection-1"
author: "송지민"
toc: true
---

## 문제 개요
웹앱이 사용자 입력 값 `host`를 받아 쉘에서 `ping -c 3 "{host}"` 명령을 실행한다. 입력값을 따옴표로 감싸서 사용하지만, 서버 측에서 추가적인 필터링이 없어 따옴표/세미콜론을 이용한 커맨드 인젝션이 가능하다.

문제 코드:

```python
#!/usr/bin/env python3
import subprocess

from flask import Flask, request, render_template, redirect

from flag import FLAG

APP = Flask(__name__)


@APP.route('/')
def index():
    return render_template('index.html')


@APP.route('/ping', methods=['GET', 'POST'])
def ping():
    if request.method == 'POST':
        host = request.form.get('host')
        cmd = f'ping -c 3 "{host}"'
        try:
            output = subprocess.check_output(['/bin/sh', '-c', cmd], timeout=5)
            return render_template('ping_result.html', data=output.decode('utf-8'))
        except subprocess.TimeoutExpired:
            return render_template('ping_result.html', data='Timeout !')
        except subprocess.CalledProcessError:
            return render_template('ping_result.html', data=f'an error occurred while executing the command. -> {cmd}')

    return render_template('ping.html')


if __name__ == '__main__':
    APP.run(host='0.0.0.0', port=8000)
```

## 내가 실제로 푼 방법
폼의 `host` 입력값이 클라이언트 측 `pattern`으로 제한되어 있었지만, 개발자도구로 해당 필드를 지우거나 직접 POST 요청을 보내면 우회 가능했다. 서버는 입력을 따옴표로 감싸서 실행하지만, 쉘에서 세미콜론(`;`)으로 명령 연결과 `#`으로 주석을 만들어 주입할 수 있다.

### 내가 사용한 페이로드
```
8.8.8.8" ; flag.py #
```

설명:
- 서버에서 실행되는 명령은 `ping -c 3 "8.8.8.8" ; flag.py #"`
- `;`로 이어진 `flag.py` 명령이 실행되고 `#` 이후는 주석 처리되어 남은 따옴표 문제가 무시된다.
- 개발자 도구로 input `pattern` 제거 후 폼 제출하거나 curl로 직접 POST하면 된다.

### PoC
```bash
curl -X POST -F 'host=8.8.8.8" ; python3 flag.py #' http://TARGET:8000/ping
```

설명:
- 클라이언트 측 검사만 있고 서버 측에서 입력을 안전하게 처리하지 않으면 이런 방식으로 임의 명령을 실행할 수 있다.
- 내가 폼에 위 값을 넣고 제출하니 flag.py가 실행되어 FLAG를 화면에 출력했다.

## 재현 흐름
1. 개발자도구로 input `pattern` 제거(또는 curl 사용)  
2. POST /ping with host=`8.8.8.8" ; python3 flag.py #`  
3. 서버가 쉘로 명령을 실행 → flag 내용이 결과에 포함  
4. 화면에서 FLAG 확인

---
