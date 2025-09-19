---

title: "[CTF] filestorage"
date: 2025-09-19 21:50:00 +0900
categories: [CTF, Web]
tags: [CTF, Prototype Pollution]
description: "드림핵 filestorage 문제 풀이 정리"
author: "송지민"
toc: true
---------

## 문제 개요

이번 문제는 간단한 파일 저장/읽기 서비스로, 파일 생성 시 내부 객체(`file`)와 읽기 시 사용하는 객체(`read`)를 활용한다. 파일 생성 시 파일명이 해시되어 저장되므로 직접 이름으로 접근하기 어렵지만, 서비스에는 객체 조작을 허용하는 `/test` 엔드포인트(`rename` 기능)가 있어 프로토타입 오염을 이용하면 `read` 객체의 기본값을 바꿔 서버 루트의 파일을 읽을 수 있다.

* 업로드 엔드포인트: `/mkfile` (POST: `filename`, `content`) — 실제 파일명은 SHA256 해시로 저장
* 읽기 엔드포인트: `/readfile?filename=...` — 내부 맵 `file`에서 조회 후 `read[filename]`을 파일 경로로 사용
* 관리 엔드포인트: `/test?func=rename&filename=...&rename=...` — `setValue()`를 통해 `file` 객체에 키를 설정
* FLAG 위치: `../../../flag` (상대패스 사용 예시)

---

## 문제에 제공된 원본 소스코드

아래는 문제에서 제공된 `app.js` 전체 소스코드(핵심부 포함)이다.

```js
const express=require('express');
const bodyParser=require('body-parser');
const ejs=require('ejs');
const hash=require('crypto-js/sha256');
const fs = require('fs');
const app=express();


var file={};
var read={};
function isObject(obj) {
  return obj !== null && typeof obj === 'object';
}
function setValue(obj, key, value) {
  const keylist = key.split('.');
  const e = keylist.shift();
  if (keylist.length > 0) {
    if (!isObject(obj[e])) obj[e] = {};
    setValue(obj[e], keylist.join('.'), value);
  } else {
    obj[key] = value;
    return obj;
  }
}

app.use(bodyParser.urlencoded({ extended: false }));
app.set('view engine','ejs');


app.get('/',function(req,resp){
	read['filename']='fake';
	resp.render(__dirname+"/ejs/index.ejs");

})

app.post('/mkfile',function(req,resp){
	let {filename,content}=req.body;
	filename=hash(filename).toString();
	fs.writeFile(__dirname+"/storage/"+filename,content,function(err){
		if(err==null){
			file[filename]=filename;
			resp.send('your file name is '+filename);
		}else{
			resp.write("<script>alert('error')</script>");
			resp.write("<script>window.location='/'</script>");
		}
	})

})

app.get('/readfile',function(req,resp){
	let filename=file[req.query.filename];
	if(filename==null){
		fs.readFile(__dirname+'/storage/'+read['filename'],'UTF-8',function(err,data){
			resp.send(data);
		})
	}else{
		read[filename]=filename.replaceAll('.','');
		fs.readFile(__dirname+'/storage/'+read[filename],'UTF-8',function(err,data){
			if(err==null){
				resp.send(data);
			}else{
				resp.send('file is not existed');
			}
		})
	}

})

app.get('/test',function(req,resp){
	let {func,filename,rename}=req.query;
	if(func==null){
		resp.send("this page hasn't been made yet");
	}else if(func=='rename'){
		setValue(file,filename,rename)
		resp.send('rename');
	}else if(func=='reset'){
		read={};
		resp.send("file reset");
	}
})


app.listen(8000);
```

---

## 내가 실제로 푼 방법

핵심은 \*\*프로토타입 오염(protoype pollution)\*\*을 이용해 `read` 객체의 기본 프로퍼티를 바꾸는 것이다. `setValue()`는 dotted key를 분해하여 `file` 객체 내부에 중첩 속성을 생성한다. 이 동작을 이용해 `file` 객체의 프로토타입 체인에 값을 설정하면, 이후 `read['filename']` 조회 시 프로토타입에서 값을 가져오도록 유도할 수 있다.

### 공격 아이디어 요약

1. `/test?func=rename&filename=proto.filename&rename=../../../flag` 로 호출해 `file` 객체의 프로토타입에 `filename` 키를 설정한다.
2. `/test?func=reset`로 `read` 객체를 초기화(로컬 오브젝트 상태 정리).
3. `GET /readfile?filename=asdasdasdasasd` 처럼 존재하지 않는 파일명을 조회하면 `file[req.query.filename]`는 `null`이 되고, 코드 경로는 `fs.readFile(__dirname+'/storage/'+read['filename'],...)`를 실행한다. 이때 `read['filename']`이 오브젝트의 own property로는 없지만 프로토타입에 설정된 값이 있으면 그 값이 사용되어 상대 경로를 통해 루트의 `flag` 파일을 읽을 수 있다.

### 요청 예시

* 프로토타입 오염 수행:

  ```
  /test?func=rename&filename=proto.filename&rename=../../../flag
  ```
* read 객체 초기화:

  ```
  /test?func=reset
  ```
* 취약점 트리거:

  ```
  /readfile?filename=asdasdasdasasd
  ```

---

## 재현 흐름

1. 공격자: `/test?func=rename&filename=proto.filename&rename=../../../flag` 호출 → `file`의 프로토타입에 `filename` 값 설정
2. 공격자: `/test?func=reset` 호출 → `read` 객체 초기화
3. 공격자: `/readfile?filename=<nonexistent>` 호출 → 서버가 `read['filename']`에서 프로토타입 값을 읽어 `fs.readFile(__dirname+'/storage/'+ '../../../flag', ...)`를 시도
4. 서버: 상대경로를 통해 루트의 flag 파일을 읽어 응답으로 반환 → FLAG 획득

---

## 왜 이게 가능했나

* `setValue()`는 사용자 입력으로부터 dotted key를 받아 객체 트리를 조작하며, 프로토타입 체인을 통해 전역 객체의 속성을 오염시킬 수 있다.
* `read` 객체를 직접 조작하는 경로가 없더라도 프로토타입에 값을 설정하면 조회 시 프로토타입에서 값이 반환될 수 있다.
* 파일명 해싱으로 직접 파일명 접근을 어렵게 했지만, 프로토타입 오염은 해싱을 거치지 않는 경로를 제공한다.

---

## 마무리

이번 문제는 프로토타입 오염을 통해 애플리케이션의 내부 상태를 전역적으로 변경하고, 이를 통해 민감 파일을 읽어내는 전형적인 Node.js 환경 취약점 사례다. 객체 키의 무분별한 설정과 프로토타입 체인의 오염을 방지하려면 입력 검증과 `Object.prototype` 직접 접근을 차단하는 방어가 필요하다.