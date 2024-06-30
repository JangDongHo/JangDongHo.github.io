---
layout: post
title: "[트러블슈팅] NestJS 'an unknown value was passed to the validate function' 오류"
tags: Server
excerpt_separator: <!--more-->
---

## 문제 원인

NestJS에서 데이터 전송 객체(DTO)를 사용해서 POST를 요청하니 다음과 같은 오류가 발생했다.<!--more-->

```
{
    "statusCode": 400,
    "message": [
        "an unknown value was passed to the validate function"
    ],
    "error": "Bad Request"
}
```

createUserDto의 코드는 다음과 같다.

```
export class CreateUserDto {
  name: string;
  email: string;
}
```

## 해결 방안

### 1\. @nestjs/common을 9.3.9으로 업데이트

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcsdOA3%2Fbtr2NwWs7cT%2FMsuboZZFxzyiWIX7GtY40k%2Fimg.png)

@nestjs/common 버전을 업그레이드 하고 다시 테스트를 진행하니 다음과 같이 떴다.

```
{
    "statusCode": 400,
    "message": [
        "property name should not exist",
        "property email should not exist"
    ],
    "error": "Bad Request"
}
```

에러 message는 달라졌으나, 여전히 해결되지는 않았다.

### 2\. forbidNonWhitelisted 비활성화

```
// main.ts

import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: false,
      transform: true,
    }),
  );
  await app.listen(3000);
}
bootstrap();
```

main.ts 파일에 ValidationPipe()에 forbidNonWhitelisted 옵션을 꺼주었다.

forbidNonWhitelisted는 엔티티 데코레이터에 없는 값을 넣을 시 그 값에 대한 에러메세지 알려주는 옵션인데,

즉 dto에 정의되지 않은 프로퍼티를 차단하기 위한 용도로 사용된다.

확인해보니 실제로 string형이 아닌 undefined가 저장되었다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FshC19%2Fbtr2C8JHFQH%2FLgAjDiKiWo1Ea9AZOAbM6k%2Fimg.png)

### 3\. 클래스 유효성 검사기 추가

결론적으로 DTO 필드에 유효성 검사기가 없으면 오류가 발생했다.

그래서 다음과 같이 코드를 수정하고 테스트를 진행해보았다.

```
import { IsString } from 'class-validator';

export class CreateUserDto {
  @IsString()
  name: string;

  @IsString()
  email: string;
}
```

## Reference

[https://github.com/typestack/class-validator/issues/1873](https://github.com/typestack/class-validator/issues/1873)

[https://stackoverflow.com/questions/63838993/nodejs-an-unknown-value-was-passed-to-the-validate-function-but-i-cant-find-wh](https://stackoverflow.com/questions/63838993/nodejs-an-unknown-value-was-passed-to-the-validate-function-but-i-cant-find-wh)
