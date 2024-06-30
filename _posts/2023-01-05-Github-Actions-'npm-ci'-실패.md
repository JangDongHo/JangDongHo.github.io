---
layout: post
title: "[트러블슈팅] Github Actions 'npm ci' 실패"
tags: DevOps
excerpt_separator: <!--more-->
---

## 문제 원인

Github Actions에서 빌드하는 과정에서 \`npm ci\` 명령어를 실행하지 못했다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcz8ppD%2FbtrWjkVnd23%2FxcZaaDNgluWz4RTVo1KWb0%2Fimg.png)

## 해결 방안

테스트 node-version을 14.x에서 16.x로 변경해주어야 한다.

```
strategy:
      matrix:
        node-version: [16.x]
```

## 참고 자료
