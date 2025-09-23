---
title: "[Infosec] CSRF · CORS · postMessage · JSONP 정리"
date: 2025-09-23 09:00:00 +0900
categories: [보안, Web]
tags: [CSRF, CORS, postMessage, JSONP, XSS, 취약점, 대응]
description: "보호와 우회 정책 충돌"
author: "송지민"
toc: true
---

# 요약
이 문서는 CSRF(Cross-Site Request Forgery)와 관련 방어(토큰 기반), CORS(교차 출처 리소스 공유)의 오용으로 발생하는 취약점, `postMessage`와 JSONP 관련 리스크를 한데 모아 정리합니다.  
각 섹션은 **취약 원인 → 공격/우회 예시 → 실무 대응** 순으로 구성되어 있어 블로그용 학습 정리로 바로 사용할 수 있습니다.

---

# 1. CSRF (Cross‑Site Request Forgery)
## 1.1 개념 요약
CSRF는 사용자가 이미 인증한 상태(쿠키·세션을 가진 상태)에서, 공격자가 유도한 요청이 피해자 권한으로 실행되도록 만드는 공격입니다. 주로 상태 변경(송금, 설정 변경 등) 기능을 표적으로 삼습니다.

## 1.2 CSRF Token
- 서버는 세션이나 서버 저장소에 난수 토큰을 생성해 보관한다.
- 클라이언트는 폼(hidden input)이나 AJAX 요청의 페이로드에 토큰을 포함하여 전송한다.
- 서버는 전송된 토큰과 저장된 토큰을 `hash_equals` 등으로 비교하여 동일하면 요청을 신뢰함.

### 예시
```php
<?php
session_start();
// 토큰 생성
if (!isset($_SESSION['csrftoken'])) {
    $_SESSION['csrftoken'] = bin2hex(random_bytes(32));
}
$csrftoken = $_SESSION['csrftoken'];

$method = $_SERVER['REQUEST_METHOD'];
if ($method !== 'GET' && $method !== 'HEAD') {
    if (!isset($_POST['csrftoken']) || !hash_equals($csrftoken, $_POST['csrftoken'])) {
        header('HTTP/1.1 403 Forbidden');
        die('CSRF detected');
    }
    echo 'Input value: ';
    echo htmlentities($_POST['query'] ?? '', ENT_QUOTES | ENT_HTML5, 'utf-8');
}
?>
<form action="" method="POST">
    <input name="csrftoken" type="hidden" value="<?= htmlentities($csrftoken, ENT_QUOTES | ENT_HTML5, 'utf-8') ?>">
    <input name="query" type="text" />
    <input type="submit" />
</form>
```

## 1.3 흔한 실수와 우회
- **토큰 길이가 짧음**: 무차별 대입(bruteforce) 위험.
- **예측 가능한 난수**: 시간 기반 PRNG 사용 금지 — 반드시 CSPRNG 사용.
- **토큰 노출**: URL 파라미터에 넣지 않기(Referer에 유출 가능).  
- **토큰 유효기간이 지나치게 김**: 세션과 연동하거나 적당한 만료 시간을 설정.

## 1.4 실무 권장사항
- 토큰은 `random_bytes`/`CSPRNG`로 생성(길이 32바이트 이상 권장).
- 토큰은 쿠키 + 폼/헤더(또는 double submit cookie) 방식 중 서비스 요구사항에 맞게 선택.
- AJAX 요청 시 `X-CSRF-Token` 같은 커스텀 헤더 사용(서버에서 검증).
- Referer/Origin 검증을 병행하면 추가적인 방어가 된다(특히 민감한 엔드포인트).
- 중요: `SameSite` 쿠키 설정을 적극 활용 (`Lax` 또는 `Strict`).

---

# 2. CORS(교차 출처 리소스 공유)
## 2.1 개념 요약
CORS는 브라우저가 SOP로 제한한 자원 접근을 서버가 허용하도록 해주는 표준입니다. 서버는 응답 헤더(`Access-Control-Allow-*`)로 어떤 출처에 접근을 허용할지 선언합니다.

## 2.2 잘못된 설정으로 인한 취약점
- `Access-Control-Allow-Origin: *` + `Access-Control-Allow-Credentials: true` 조합은 브라우저가 막기 때문에 잘못 쓰면 정보 유출/부적절한 권한 부여로 이어질 수 있음.
- 오리진 검증 없이 모든 오리진에 대해 동적으로 `Origin` 값을 echo해서 설정하면 사실상 모든 도메인에 민감한 리소스 접근을 허용하게 됨.
- 프리플라이트/헤더 검증 누락으로 인해 민감한 헤더/메소드가 그대로 허용될 수 있음.

## 2.3 공격 예시 (정보 유출)
- 공격자 페이지가 victim.com으로 인증된 사용자의 브라우저에서 `fetch('https://victim.com/api/private', {credentials:'include'})` 를 호출. 서버가 `Access-Control-Allow-Origin: https://attacker.com` 처럼 부적절하게 응답하면 공격자가 응답을 읽을 수 있음.

## 2.4 실무 대응
- **Origin 화이트리스트**: 서버에서 `Origin` 값을 검증한 뒤 화이트리스트에 있는 경우에만 해당 Origin을 `Access-Control-Allow-Origin`에 설정. 절대 클라이언트에서 넘어온 Origin을 그대로 반영하지 말 것.
- `Access-Control-Allow-Credentials: true` 를 설정할 때는 특히 주의. 크리덴셜이 포함된 요청을 허용하려면 정확한 Origin만 허용해야 함.
- `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`를 최소 권한으로 설정.
- 민감한 API는 CORS로 공개하지 말고, 백엔드 대행(proxy) 또는 서버간 통신으로 처리.
- 로그에 Origin을 기록하고, 비정상 Origin의 요청은 차단·모니터링.

---

# 3. postMessage 취약점
## 3.1 개요
`window.postMessage`는 다른 창/프레임과 안전하게 데이터를 주고받기 위한 API입니다. 그러나 **수신 측에서 origin 검증을 하지 않거나, 메시지 내용(데이터)을 무분별하게 신뢰하면 취약점이 발생**합니다.

## 3.2 안전 사용 가이드 (메시지 수신 측)
- `window.addEventListener('message', handler)`에서 `event.origin`을 반드시 확인.
- `event.source`를 사용해 응답을 보낼 대상이 신뢰할 수 있는 창인지 확인할 수 있음.
- 메시지의 구조를 검증(스키마 체크, 타입 체크)하고 `eval` 또는 동적 코드 실행 금지.

### 수신 예시
```js
window.addEventListener('message', function(e) {
  if (e.origin !== 'https://trusted.example') return; // origin 검증
  const data = e.data;
  if (typeof data !== 'object' || !data.type) return; // 스키마 체크
  // 안전하게 처리...
});
```

## 3.3 Origin 전환(타이밍) 경합 이슈
- 부모/자식 창이 리다이렉트되어 `targetWindow`의 origin이 변경될 수 있음. 메시지 전송 시점과 수신 시점 사이 origin이 바뀔 가능성을 설계 시 고려해야 함.
- `targetOrigin` 파라미터에 `"*"`를 쓰지 말고 정확한 origin을 넣어 메시지 송신을 제한.

---

# 4. JSONP (JSON with Padding) — 위험과 대체
## 4.1 개요
JSONP는 `<script>` 태그를 이용해 다른 출처의 데이터를 로드하고, 서버가 반환하는 콜백을 실행시키는 방식입니다. XHR/CORS 이전에 사용되었지만 본질적으로 스크립트 실행이므로 보안 위험이 큽니다.

## 4.2 취약점
- 콜백 파라미터를 검증하지 않으면 공격자가 임의의 자바스크립트를 주입(또는 콜백명으로 XSS 유발)할 수 있음.
- JSONP 제공자가 침해당하면 이를 사용하는 모든 사이트가 위험에 노출됨.
- JSONP는 GET만 사용하므로 민감한 동작을 GET으로 노출시키는 설계 실수와 결합되면 크리티컬하다.

## 4.3 대응/대체
- JSONP 대신 **CORS + fetch/XHR** 사용.
- JSONP를 유지해야 할 경우: `callback` 값은 엄격한 정규표현식(예: `^[A-Za-z0-9_\.\[\]]+$`)로 검증하고, `Content-Type`을 `application/javascript`로 설정하며 `X-Content-Type-Options: nosniff` 추가.
- JSONP 응답에 민감한 데이터(인증 관련)를 절대 포함하지 말 것.

---

# 5. 실전 체크리스트
- CSRF:
  - 토큰은 CSPRNG로 생성(충분한 길이), 폼/헤더에 포함, Referer/Origin 검증 병행.
  - `SameSite` 쿠키 설정 적용.
- CORS:
  - Origin 화이트리스트만 허용, `Allow-Credentials`와 함께 `*` 사용 금지.
  - 최소 권한 원칙(Methods/Headers 최소화).
- postMessage:
  - `event.origin` 검사, 메시지 스키마 검증, `targetOrigin` 정확히 지정.
- JSONP:
  - 가능하면 사용 금지. 불가피하면 콜백명 화이트리스트 + Content-Type/Nosniff.

---

# 6. 참고용 페이로드/테스트
- CSRF (HTML 삽입을 통한 요청 예)
```html
<img src="https://victim.example/send?to=attacker&amount=1000" style="display:none">
```
- CORS (취약한 서버에서 자격증명 포함 응답 허용 시)
```js
fetch('https://vulnerable.example/private', {credentials: 'include'})
  .then(r => r.text()).then(console.log)
```
- JSONP (취약 콜백명 삽입)
```html
<script src="https://api.example.com/user?callback=alert(1)"></script>
```


