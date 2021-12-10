---
layout: post
title: "Java :: CLASSPATH"
author: "leehyunjae"
tags: ["java"]
---

> classpath 에 대해 정리해보기

### CLASSPATH

- (Windows의 시스템 환경 변수 처럼) Java 실행 시 클래스 파일들(.class)의 위치/경로
- classpath에 실행하고자 하는 패키지(클래스)의 루트를 등록

<br>

### 예시

`/home/user/me/com/example/Test.class` 를 실행하고자 할 때, 이 클래스가 위치한 경로로 찾아가지 않고 (`java Teset` 명령어를 통해)아무곳에서나 사용하고 싶다.

```sh
> vim .zshrc

CLASSPATH="/home/user/me/com/example"

> java Test
Hello, World!
```



