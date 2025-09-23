---
title: "[Infosec] CSP 우회 정리"
date: 2025-09-18 18:00:00 +0900
categories: [보안, Web]
tags: [웹해킹, CSP, XSS, Bypass]
description: "CSP(Content Security Policy) 설정에서 흔히 발생하는 취약점과 우회 기법, 그리고 실무 대응을 정리한 문서입니다."
author: "송지민"
toc: true
---

# 개요
이 문서는 **CSP(Content Security Policy)** 를 올바르게 적용하지 않았을 때 발생하는 대표적인 우회 기법을 정리하고, 각 우회에 대한 실전적인 대응책과 설정 권장안을 제공합니다. 학습·방어 목적이며, 실제 서비스를 보호하는 관점에서 적용하세요.

> 요약: CSP는 훌륭한 완화 수단이나, **신뢰한 출처의 취약점(파일 업로드, JSONP 등), nonce/캐시·base-uri 설정 오류, 그리고 브라우저 정규화 차이**를 통해 우회될 수 있습니다. 우회 시나리오와 그에 대한 방어를 함께 제시합니다.

---

# 1. 신뢰하는 도메인에 업로드(Trusted domain upload)
## 개념
CSP는 특정 도메인(출처)을 신뢰하도록 허용할 수 있습니다. 그러나 허용한 출처가 파일 업로드/호스팅 기능을 제공하면 공격자가 해당 출처에 스크립트나 HTML을 업로드한 뒤, 페이지에서 해당 리소스를 로드하게 하여 XSS를 수행할 수 있습니다.

## 우회 예시
```html
<meta http-equiv="Content-Security-Policy" content="script-src 'self' https://uploads.example.com">
...
<h1>검색 결과: <script src="/download_file.php?id=177742"></script></h1>
```
- `/download_file.php?id=177742`가 외부 업로드 파일을 반환하면, 악성 스크립트가 실행될 수 있음.

## 대응
- 신뢰 도메인에도 **업로드된 콘텐츠가 스크립트로 실행되지 않도록** 서버 측에서 Content-Type을 강제(`text/plain`)하거나 `Content-Disposition: attachment`로 다운로드 처리.
- 업로드된 파일은 별도의 서브도메인(예: `uploads.example.com`)에서 서비스하되, 그 서브도메인에 대해 `script-src`를 허용하지 않음(리소스는 static-only).
- 업로드 스토리지에 업로드 가능한 확장자·MIME을 엄격히 검증. SVG 등 XML 계열은 특히 주의.
- 가능하다면 업로드 영역은 별도의 오리진에서 호스팅하고, 해당 호스트의 CSP는 매우 엄격하게 설정.

---

# 2. JSONP API 악용
## 개념
JSONP는 서버가 `callback` 파라미터를 받아서 JS 콜백을 반환하는 기법입니다. CSP가 `https://trusted.example` 같은 출처를 허용하면, 그 출처의 JSONP API를 통해 임의 코드를 실행할 수 있습니다.

## 우회 예시
```
<script src="https://trusted.example/api?callback=alert(1)"></script>
```
- trusted.example이 콜백을 그대로 반영하면 `alert(1)`이 실행됨.

## 대응
- 가능한 경우 JSONP 사용을 금지하고 CORS로 전환.
- JSONP를 제공해야 한다면 `callback` 파라미터는 엄격히 화이트리스트(정규식: 알파넘·언더스코어·점·대괄호 등)로 검증.
- CSP 정책에서 서드파티 API가 허용되어야만 사용하도록 최소 권한 설정.

---

# 3. nonce 예측·캐시 문제
## 개념
`nonce` 기반 허용은 인라인 스크립트를 안전하게 허용할 수 있게 해주지만, **nonce가 예측 가능하거나 캐싱되어 재사용되면** 보호 효과가 무력화됩니다.

## 위험 사례
- 서버가 **고정된 nonce**를 쓰거나, 비충분한 랜덤 소스(CSPRNG가 아닌 rand/time 기반) 사용.
- Nginx+php-fpm 설정에서 `PATH_INFO`나 잘못된 location 처리로 인해 `.php`가 포함된 경로에서 HTML이 정적 파일처럼 캐시되어 nonce가 CDN/프록시에 의해 캐시되는 경우.

## 공격 시나리오
- CDN이 `.css`나 기타 정적 리소트만 캐시하도록 구성되어 있더라도, 잘못된 설정으로 동적 페이지가 캐시되어 nonce가 동일하게 재배포될 수 있음. 공격자는 캐시된 페이지에서 nonce를 획득해 악성 인라인 스크립트를 주입 가능.

## 대응
- **매 요청마다 고유한 nonce**를 CSPRNG로 생성하고, 절대 재사용 금지.
- CDN/프록시에서 HTML 응답이 캐시되지 않도록 `Cache-Control: no-store, no-cache, must-revalidate` 등을 설정.
- Nginx 설정 시 `location ~ \.php$` 형태로 끝맺는 블록을 사용하고, `fastcgi_split_path_info` 이용 시 `try_files`로 스크립트 존재 여부를 확인하여 PATH_INFO 취급을 주의.
- nonce는 HTTP 헤더로 전달하거나 templating에서 동적으로 삽입하고, CDN 캐싱 정책과 연계해 테스트.

---

# 4. base-uri 미지정 (Base tag 재작성 / Nonce Retargeting)
## 개념
`<base>` 태그는 문서 내 상대 URL 해석의 기준을 바꿉니다. 공격자가 DOM 삽입 지점을 통해 `<base href="https://attacker/">`를 추가하면, 이후의 상대 URL들은 공격자 도메인을 가리키게 되어 리소스(스크립트 등)를 우회 로드할 수 있습니다. 이를 **Nonce Retargeting** 공격이라고도 부릅니다.

## 우회 예시
```html
<base href="https://malice.test/">
<script src="/jquery.js" nonce="NONCE"></script>
<!-- 실제 로드: https://malice.test/jquery.js -->
```

## 대응
- CSP에 `base-uri 'none'` 또는 `base-uri 'self'` 를 명시해서 `<base>` 태그가 문서에 의해 바뀌지 않도록 제한.
- 가능한 한 페이지에서 `<base>`를 사용하지 않거나, 사용 시 출처를 엄격히 제한.

---

# 5. 기타 우회/정규화 이슈 (브라우저 차이, 인코딩)
## 개념
브라우저별 URL 정규화 방식, 인코딩(퍼센트 인코딩) 처리 차이, 제어 문자(`%09`, `%0A`, `%00` 등)에 의해 `javascript:` 스킴이나 인라인 실행이 복원될 수 있습니다.

## 대응
- 사용자 입력을 이용해 URL을 만들거나 속성에 삽입할 때는 `new URL()`로 정규화 후 `protocol`/`hostname`을 검사.
- 서버와 클라이언트 모두에서 디코딩 → 검사 → 인코딩(또는 거부) 순서를 통일.
- 제어문자(ASCII 0x00–0x1F, 0x7F) 포함 여부를 검사하여 거부.
- CSP 외 추가적인 출력 컨텍스트별 이스케이프(HTML attr, JS, URL 등)를 적용.

---

# 6. 요약: 우회 벡터와 핵심 방어 체크리스트
- **업로드 가능한 신뢰 도메인**: 업로드 스토리지에서 스크립트 실행 금지 / 다운로드를 attachment로 제공 / 별도 오리진에서 서빙 및 제한.
- **JSONP**: 사용 금지 권장 / 불가 시 callback 화이트리스트 적용.
- **Nonce 관련**: CSPRNG, per-request, CDN 캐시 정책 확인.
- **base-uri**: `base-uri 'none'` 또는 엄격 설정.
- **정규화/인코딩**: 디코드 후 제어문자 검사, URL 객체로 프로토콜 검증.
- **로깅/모니터링**: CSP report (report-uri/report-to) 활성화, 위반 로그 주기적으로 검토.

---

# 7. 부록: 실전 설정 예시 
- 강력 권장(최소 권한, 해시/nonce 병행):
```
Content-Security-Policy:
  default-src 'none';
  base-uri 'self';
  script-src 'self' 'nonce-<RUNTIME>' 'sha256-<BUILD_HASH>';
  style-src 'self' 'sha256-<BUILD_HASH>';
  img-src 'self' data:;
  object-src 'none';
  frame-ancestors 'none';
  report-to csp-endpoint;
```

- report-only로 먼저 운영:
```
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report;
```

---