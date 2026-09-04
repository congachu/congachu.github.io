---
title: "[Infosec] Port Scanning & Service Scanning"
date: 2026-09-04 12:00:00 +0900
categories: [보안, 정보수집]
tags: [Nmap, Port Scanning, Service Scanning]
description: "Nmap을 이용한 포트 및 서비스 정보 수집"
toc: true
---

# 모의해킹 정보수집 - 포트 스캐닝과 서비스 스캐닝

모의해킹의 정보수집 단계에서 **포트 스캐닝(Port Scanning)**과 **서비스 스캐닝(Service Scanning)**을 학습했다.

정보수집은 단순히 대상의 IP를 알아내는 것에서 끝나는 것이 아니라,

```text
호스트 확인
   ↓
포트 확인
   ↓
서비스 확인
   ↓
서비스 버전 및 정보 확인
   ↓
취약점 분석
```

과 같이 대상 시스템에 대한 정보를 점점 구체적으로 수집해 나가는 과정이다.

이번에는 `Nmap`을 이용하여 포트와 서비스 정보를 수집하는 방법을 학습했다.

---

# 1. 포트 스캐닝

## 포트 스캐닝이란?

**포트 스캐닝(Port Scanning)**은 대상 호스트의 어떤 포트가 열려 있는지 확인하는 과정이다.

네트워크 통신에서 포트는 특정 서비스와 연결되어 있기 때문에, 열린 포트를 확인하면 대상 시스템에서 어떤 서비스가 동작하고 있을 가능성이 있는지 파악할 수 있다.

예를 들어 다음과 같은 결과를 얻었다고 하자.

```text
PORT
22/tcp   OPEN
80/tcp   OPEN
443/tcp  OPEN
```

이를 통해 대상 호스트에서 22, 80, 443 포트가 열려 있다는 사실을 확인할 수 있다.

---

# 2. Nmap

포트 스캐닝에는 `Nmap(Network Mapper)`을 사용했다.

Nmap은 네트워크상의 호스트와 포트, 서비스 등의 정보를 탐색할 수 있는 대표적인 네트워크 스캐닝 도구이다.

---

## 2.1 기본 스캔

```bash
nmap <address>
```

대상 호스트에 대해 기본적인 TCP 포트 스캔을 수행한다.

예:

```bash
nmap 192.168.0.10
```

---

# 3. Host Discovery

포트 스캐닝을 수행하기 전에 대상 호스트가 실제로 존재하고 동작하고 있는지 확인할 수 있다.

## `-sn`

```bash
nmap -sn <address>
```

`-sn` 옵션은 **포트 스캔을 수행하지 않고 Host Discovery를 수행**한다.

예:

```bash
nmap -sn 192.168.0.10
```

이를 통해 대상 호스트가 살아 있는지 확인할 수 있다.

```text
대상 주소
   ↓
Host Discovery
   ↓
호스트가 존재하는가?
```

---

# 4. TCP Connect Scan

```bash
nmap <address>
```

Nmap의 기본적인 TCP 스캔 방식 중 하나이다.

TCP 연결을 실제로 성립시키는 방식으로 동작한다.

일반적인 TCP 연결 과정은 다음과 같다.

```text
Scanner                Target
   |                     |
   |------ SYN --------->|
   |<--- SYN/ACK --------|
   |------ ACK --------->|
   |                     |
   |    TCP 연결 완료     |
```

TCP 연결이 정상적으로 이루어지면 해당 포트가 열려 있다고 판단할 수 있다.

---

# 5. TCP SYN Scan

## `-sS`

```bash
nmap -sS <address>
```

**TCP SYN Scan**을 수행한다.

TCP 연결 과정에서 SYN 패킷을 보내고 응답을 확인하지만, 일반적인 TCP 연결을 끝까지 완료하지 않는 방식이다.

```text
Scanner                Target
   |                     |
   |------ SYN --------->|
   |<--- SYN/ACK --------|
   |------ RST --------->|
   |                     |
   |   연결을 완료하지 않음
```

따라서 TCP Connect Scan과 달리 실제 TCP 연결을 완전히 성립시키지 않는다.

---

# 6. ICMP Scan

Host Discovery 과정에서는 ICMP를 이용하여 대상 호스트의 응답 여부를 확인할 수도 있다.

Nmap의 `-sn` 옵션을 이용한 Host Discovery에서는 ICMP Echo Request 등의 여러 탐지 방법이 상황에 따라 사용될 수 있다.

```bash
nmap -sn <address>
```

즉, `-sn`은 특정 하나의 ICMP 스캔만 의미하는 것이 아니라 **포트 스캔을 생략하고 호스트 탐지를 수행하는 옵션**이라고 이해하는 것이 좋다.

---

# 7. UDP Scan

## `-sU`

```bash
nmap -sU <address>
```

**UDP 포트 스캔**을 수행한다.

TCP와 달리 UDP는 연결 지향적인 프로토콜이 아니기 때문에 TCP 스캔과 다른 방식으로 열린 포트를 판단한다.

```text
TCP
 ↓
연결 과정 존재
 ↓
SYN / SYN-ACK 등의 응답 확인

UDP
 ↓
연결 과정 없음
 ↓
UDP 응답 또는 ICMP 오류 등을 이용하여 상태 판단
```

UDP는 응답이 없는 경우도 있기 때문에 일반적으로 TCP 포트 스캔보다 스캔에 시간이 오래 걸릴 수 있다.

---

# 8. 포트 지정

Nmap에서는 `-p` 옵션을 사용하여 스캔할 포트를 지정할 수 있다.

## 특정 포트

```bash
nmap -p <port> <address>
```

예:

```bash
nmap -p 80 192.168.0.10
```

80번 포트만 스캔한다.

여러 포트를 지정할 수도 있다.

```bash
nmap -p 22,80,443 192.168.0.10
```

---

## 포트 범위 지정

```bash
nmap -p <start-port>-<end-port> <address>
```

예:

```bash
nmap -p 1-1000 192.168.0.10
```

1번부터 1000번까지의 포트를 스캔한다.

---

## 모든 포트 스캔

```bash
nmap -p- <address>
```

전체 TCP 포트 범위인 **1~65535번 포트**를 스캔한다.

예:

```bash
nmap -p- 192.168.0.10
```

---

## 주요 포트 스캔

```bash
nmap --top-ports <number> <address>
```

Nmap에서 정의한 주요 포트들을 대상으로 스캔한다.

예:

```bash
nmap --top-ports 100 192.168.0.10
```

전체 포트를 스캔하는 것보다 빠르게 주요 서비스의 존재 여부를 확인할 수 있다.

---

# 9. 열린 포트만 확인

## `--open`

```bash
nmap --open <address>
```

스캔 결과에서 **열린 포트가 있는 결과를 중심으로 출력**한다.

예:

```bash
nmap --open 192.168.0.10
```

많은 포트를 스캔했을 때 열린 포트를 중심으로 결과를 확인하기 편리하다.

---

# 10. Host Discovery 비활성화

## `-Pn`

```bash
nmap -Pn <address>
```

Nmap의 **Host Discovery 과정을 생략**하고 포트 스캔을 수행한다.

일반적인 경우 Nmap은 먼저 대상 호스트가 살아 있는지 확인한 후 포트 스캔을 수행한다.

```text
Host Discovery
      ↓
호스트 존재 확인
      ↓
Port Scanning
```

하지만 방화벽 등의 이유로 Host Discovery에 응답하지 않는 경우 실제로 호스트가 존재하더라도 스캔이 정상적으로 이루어지지 않을 수 있다.

이때 `-Pn`을 사용하면 호스트가 살아 있다고 간주하고 포트 스캔을 진행한다.

```bash
nmap -Pn 192.168.0.10
```

---

# 11. 스캔 결과 저장

## `-oA`

```bash
nmap -oA <filename> <address>
```

`-oA` 옵션을 사용하면 Nmap 스캔 결과를 여러 형식으로 저장할 수 있다.

예:

```bash
nmap -oA scan_result 192.168.0.10
```

다음과 같은 파일들이 생성된다.

```text
scan_result.nmap
scan_result.xml
scan_result.gnmap
```

스캔 결과를 파일로 저장하면 이후 분석하거나 모의해킹 보고서를 작성할 때 활용할 수 있다.

---

# 12. 서비스 스캐닝

포트 스캐닝을 통해 열린 포트를 확인했다면, 다음으로 **해당 포트에서 어떤 서비스가 동작하고 있는지** 확인할 수 있다.

예를 들어 포트 스캐닝 결과가 다음과 같다고 하자.

```text
22/tcp   OPEN
80/tcp   OPEN
443/tcp  OPEN
```

포트가 열려 있다는 사실만으로는 정확히 어떤 서비스가 동작하고 있는지 알기 어렵다.

따라서 서비스 스캐닝을 통해 다음과 같은 정보를 추가로 수집한다.

```text
열린 포트
   ↓
어떤 서비스가 동작하는가?
   ↓
어떤 버전인가?
   ↓
서비스의 특징 및 추가 정보
```

서비스 스캐닝에서는 **배너 그래빙(Banner Grabbing)**과 **프로빙(Probing)** 등을 활용할 수 있다.

---

# 13. 배너 그래빙 (Banner Grabbing)

## `-sV`

```bash
nmap -sV <address>
```

`-sV` 옵션을 사용하면 대상 포트에서 실행 중인 **서비스와 버전 정보를 탐지**할 수 있다.

서비스가 연결될 때 자신의 서비스 종류나 버전 등의 정보를 제공하는 경우가 있는데, 이러한 정보를 확인하는 것을 **배너 그래빙(Banner Grabbing)**이라고 한다.

예:

```bash
nmap -sV 192.168.0.10
```

결과:

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1
80/tcp open  http    Apache httpd 2.4.57
```

이를 통해 다음과 같은 정보를 얻을 수 있다.

```text
22/tcp → SSH → OpenSSH 9.2p1
80/tcp → HTTP → Apache 2.4.57
```

즉, 단순히 포트가 열려 있다는 것뿐만 아니라 **어떤 서비스가 어떤 버전으로 실행되고 있는지** 파악할 수 있다.

이러한 정보는 이후 취약점 분석 단계에서 중요한 정보가 될 수 있다.

> 배너 그래빙의 핵심은 서비스가 제공하는 정보를 이용하여 **"무슨 서비스가 어떤 버전으로 실행되고 있는가?"**를 알아내는 것이다.

---

# 14. 프로빙 (Probing)

서비스에 직접 요청을 보내고 그 응답을 분석하여 서비스의 종류나 특성을 확인하는 방법을 **프로빙(Probing)**이라고 한다.

Nmap에서는 `-sC` 옵션을 사용하여 **기본 NSE(Nmap Scripting Engine) 스크립트**를 실행할 수 있다.

```bash
nmap -sC <address>
```

예:

```bash
nmap -sC 192.168.0.10
```

NSE 스크립트는 대상 서비스에 요청을 보내고 응답을 분석하여 추가적인 정보를 수집한다.

예를 들어 HTTP 서비스라면 웹 서버에 요청을 보내고 응답을 분석하여 웹 서버에 대한 추가 정보를 얻을 수 있다.

```text
요청
 ↓
대상 서비스
 ↓
응답
 ↓
응답 내용 분석
 ↓
서비스에 대한 추가 정보 획득
```

`-sV`와 `-sC`는 서로 다른 목적을 가진다.

```text
-sV
 ↓
서비스 및 버전 탐지
 ↓
"무슨 서비스인가?"

-sC
 ↓
기본 NSE 스크립트 실행
 ↓
"이 서비스의 특성이나 추가 정보는 무엇인가?"
```

---

# 15. Nmap으로 확인하기 어려운 경우

Nmap의 `-sV`나 `-sC`를 사용해도 원하는 정보를 충분히 얻지 못하는 경우가 있다.

이때는 **Netcat(nc)을 이용하여 서비스에 직접 연결**해 보는 방법도 있다.

```bash
nc <address> <port>
```

예를 들어 80번 포트의 HTTP 서비스에 직접 연결할 수 있다.

```bash
nc 192.168.0.10 80
```

연결 후 직접 HTTP 요청을 보내볼 수도 있다.

```http
GET / HTTP/1.1
Host: 192.168.0.10

```

서비스가 응답한다면 해당 응답을 직접 확인하면서 서비스의 특성을 파악할 수 있다.

Nmap이 자동화된 방식으로 정보를 수집한다면, `nc`는 서비스에 직접 연결하여 **직접 요청을 보내고 응답을 확인**할 수 있다는 차이가 있다.

```text
Nmap
 ↓
자동화된 탐지
 ↓
서비스 정보 수집

nc
 ↓
서비스에 직접 연결
 ↓
직접 요청
 ↓
응답 확인
```

따라서 Nmap으로 충분한 정보를 얻지 못했을 때 `nc`를 이용한 직접적인 프로빙도 유용한 방법이 될 수 있다.

---

# 16. 포트 스캐닝과 서비스 스캐닝의 차이

두 과정의 가장 큰 차이는 **확인하려는 정보의 수준**이다.

```text
Port Scanning
      ↓
"어떤 포트가 열려 있는가?"
      ↓
22/tcp
80/tcp
443/tcp

          ↓

Service Scanning
      ↓
"그 포트에서 무엇이 동작하는가?"
      ↓
22 → SSH → OpenSSH
80 → HTTP → Apache
443 → HTTPS → nginx
```

즉,

| 구분 | 목적 |
|---|---|
| Port Scanning | 어떤 포트가 열려 있는지 확인 |
| Service Scanning | 열린 포트에서 어떤 서비스가 동작하는지 확인 |
| Banner Grabbing | 서비스 및 버전 등의 정보 확인 |
| Probing | 서비스에 요청을 보내 추가적인 정보 확인 |
| `nc` | 서비스에 직접 연결하여 직접 요청 및 응답 확인 |

---

# 17. 전체 정보수집 흐름

이번에 학습한 내용을 전체적으로 정리하면 다음과 같다.

```text
"어떤 호스트가 있는가?"
        ↓
   Host Discovery
        ↓
"어떤 포트가 열려 있는가?"
        ↓
    Port Scanning
        ↓
"그 포트에서 무엇이 실행되는가?"
        ↓
   Service Scanning
        ↓
"어떤 버전이고 어떤 특성을 가지고 있는가?"
        ↓
Banner Grabbing / Probing
        ↓
"더 자세한 정보가 필요한가?"
        ↓
       nc
        ↓
"취약점이 존재할 가능성이 있는가?"
        ↓
 Vulnerability Analysis
```

결국 정보수집은 **점점 더 구체적인 정보를 수집해 나가는 과정**이라고 볼 수 있다.

```text
Host
 ↓
IP
 ↓
Open Port
 ↓
Service
 ↓
Version
 ↓
Service Information
 ↓
Potential Vulnerability
```

---

# 18. 주요 Nmap 명령어 정리

| 명령어 | 설명 |
|---|---|
| `nmap <address>` | 기본 TCP 포트 스캔 |
| `nmap -sn <address>` | Host Discovery |
| `nmap -sS <address>` | TCP SYN Scan |
| `nmap -sU <address>` | UDP Scan |
| `nmap -sV <address>` | 서비스 및 버전 탐지 |
| `nmap -sC <address>` | 기본 NSE 스크립트 실행 |
| `-p <port>` | 특정 포트 지정 |
| `-p <start-port>-<end-port>` | 포트 범위 지정 |
| `-p-` | 전체 포트 스캔 |
| `--top-ports <number>` | 주요 포트 지정 |
| `--open` | 열린 포트 중심으로 출력 |
| `-Pn` | Host Discovery 비활성화 |
| `-oA <filename>` | 여러 형식으로 결과 저장 |

---

# 19. 마무리

이번 학습을 통해 모의해킹의 정보수집 과정에서 **Host Discovery → Port Scanning → Service Scanning**으로 이어지는 흐름을 이해할 수 있었다.

특히 포트 스캐닝과 서비스 스캐닝은 비슷해 보이지만 목적이 다르다.

```text
Host Discovery
→ 어떤 호스트가 존재하는가?

Port Scanning
→ 어떤 포트가 열려 있는가?

Service Scanning
→ 어떤 서비스가 동작하는가?

Banner Grabbing
→ 어떤 서비스와 버전인가?

Probing
→ 서비스의 추가적인 정보는 무엇인가?
```

그리고 Nmap으로 충분한 정보를 얻지 못하는 경우에는 `nc`를 이용하여 서비스에 직접 연결하고 요청과 응답을 확인할 수도 있다.

이렇게 수집한 정보는 이후 **취약점 분석(Vulnerability Analysis)** 단계에서 대상 시스템을 분석하기 위한 기초 자료로 활용된다.