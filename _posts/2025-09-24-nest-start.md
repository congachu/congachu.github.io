---
title: "[NestJS] NestJS 프로젝트 시작하기"
date: 2025-09-24 23:00:00 +0900
categories: [Backend, NestJS]
tags: [NestJS, Backend, Framework]
description: "NestJS 프로젝트를 시작하고 구조를 알아보자"
author: "송지민"
toc: true
---


# Nest.js 기초 가이드

Nest.js는 Node.js 위에서 동작하는 **TypeScript 기반 서버 프레임워크**입니다.  
모듈화된 구조, 의존성 주입(DI), 데코레이터 기반 선언적 코드 스타일 덕분에 대규모 서비스에 적합합니다.  

아래 내용은 Nest.js 프로젝트를 처음 시작하는 분들을 위한 **핵심 개념과 코드 예제** 정리입니다.

---

## 0) 프로젝트 생성

```bash
# Nest CLI 다운로드
npm i -g @nestjs/cli

# 새 Nest 프로젝트 생성
nest new project-name

# 생성된 프로젝트로 이동
cd project-name

# 실행 (개발 모드, hot-reload 지원)
npm run start:dev
```

### 기본 생성 파일 구조
1. `main.ts` → `NestFactory`로 애플리케이션 인스턴스 생성, 부트스트랩 지점
2. `app.module.ts` → 루트 모듈
3. `app.controller.ts` → 라우팅 담당 컨트롤러
4. `app.service.ts` → 기본 서비스
5. `app.controller.spec.ts` → 컨트롤러 유닛 테스트

---

## 1) 코드 생성 (CLI)
Nest CLI를 사용하면 모듈/컨트롤러/서비스/미들웨어 등을 빠르게 생성할 수 있습니다.

```bash
# 모듈 생성
nest g mo name

# 컨트롤러 생성
nest g co module-name

# 서비스 생성
nest g service module-name

# 미들웨어 생성
nest g middleware name
```

---

## 2) 미들웨어 (Middleware)
- 요청 전/후 공통 로직을 수행하는 컴포넌트  
- 데코레이터로는 직접 사용 불가 → `AppModule.configure()` 안에서 등록해야 함

```ts
import { Injectable, Logger, NestMiddleware } from '@nestjs/common';
import { NextFunction, Request, Response } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  private logger = new Logger('HTTP');

  use(req: Request, res: Response, next: NextFunction) {
    res.on('finish', () => {
      this.logger.log(
        `${req.method} ${req.originalUrl} ${req.ip} ${res.statusCode}`,
      );
    });
    next();
  }
}

```
### 적용 방법
```ts
import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { CatsModule } from './cats/cats.module';
import { LoggerMiddleware } from './logger/logger.middleware';

@Module({
  imports: [CatsModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('cats');
  }
}
```

---

## 3) Logger
Nest의 `Logger` 클래스를 사용하면 콘솔 로깅을 손쉽게 할 수 있습니다.

```ts
private logger = new Logger('HTTP');

res.on('finish', () => {
  this.logger.log(
    `${req.method} ${req.originalUrl} ${req.ip} ${res.statusCode}`,
  );
});
```


---

## 4) Exception Filter
예외를 잡아 통일된 응답 형식으로 반환할 수 있습니다.

```ts
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
    const error = exception.getResponse() as
      | string
      | { error: string; statusCode: number; message: string | string[] };

    if (typeof error === 'string') {
      response.status(status).json({
        success: false,
        timestamp: new Date().toISOString(),
        path: request.url,
        error,
      });
    } else {
      response.status(status).json({
        success: false,
        timestamp: new Date().toISOString(),
        ...error,
      });
    }
  }
}
```

### 적용 방법
- **핸들러 단위 적용**
```ts
@Get()
@UseFilters(HttpExceptionFilter)
getAllCats(): string {
  return 'all cats';
}
```

- **컨트롤러 전체 적용**
```ts
@Controller('cats')
@UseFilters(new HttpExceptionFilter())
export class CatsController {}
```

- **글로벌 적용 (main.ts)**
```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(process.env.PORT ?? 8000);
}
```

---

## 5) Pipe
파이프는 데이터 **유효성 검증** 및 **형변환**을 담당합니다.

### 기본 제공 파이프
```ts
@Get(':id')
getOneCat(@Param('id', ParseIntPipe) param: number): string {
  console.log(param); // 문자열이 아닌 숫자로 변환됨
  return 'a cat';
}
```

### 사용자 정의 파이프
```ts
import { HttpException, Injectable, PipeTransform } from '@nestjs/common';

@Injectable()
export class PositiveIntPipe implements PipeTransform {
  transform(value: number) {
    if (value < 0) {
      throw new HttpException('value > 0', 400);
    }
    return value;
  }
}
```

---

## 6) Interceptor
인터셉터는 요청/응답을 가로채어 가공하거나, 공통 동작(로그, 캐싱, 트랜잭션 등)을 추가하는 데 사용됩니다.

```ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class SuccessInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');

    return next.handle().pipe(map((data) => ({ success: true, data })));
  }
}
```

### 적용 예시
- **라우터 단위 적용**
```ts
@Get()
@UseInterceptors(SuccessInterceptor)
getAllCats(): string {
  return 'all cats';
}
```

- **컨트롤러 전체 적용**
```ts
@Controller('cats')
@UseInterceptors(new SuccessInterceptor())
export class CatsController {}
```

- **글로벌 적용 (main.ts)**
```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalInterceptors(new SuccessInterceptor());
  await app.listen(process.env.PORT ?? 8000);
}
```

---
## NestJS Request Lifecycle

1. Incoming request
2. Middleware
   - 2.1. Globally bound middleware
   - 2.2. Module bound middleware
3. Guards
   - 3.1. Global guards
   - 3.2. Controller guards
   - 3.3. Route guards
4. Interceptors (pre-controller)
   - 4.1. Global interceptors
   - 4.2. Controller interceptors
   - 4.3. Route interceptors
5. Pipes
   - 5.1. Global pipes
   - 5.2. Controller pipes
   - 5.3. Route pipes
   - 5.4. Route parameter pipes
6. Controller (method handler)
7. Service (if exists)
8. Interceptors (post-request)
   - 8.1. Route interceptor
   - 8.2. Controller interceptor
   - 8.3. Global interceptor
9. Exception filters
   - 9.1. Route
   - 9.2. Controller
   - 9.3. Global
10. Server response
