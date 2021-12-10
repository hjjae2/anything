---
layout: post
title: "OpenJDK"
author: "leehyunjae"
tags: ["java"]
---

> [OpenJDK 공식 사이트](https://openjdk.java.net/)
> 
> _"썬 마이크로시스템즈는 라이선스에 문제가 있는 몇몇 모듈을 제외하고 openjdk.org에 JDK 코드 제공하였다. 이 코드는 공개되었다."_

OpenJDK 에는 여러 배급처(Vendor)가 존재한다.

1. [JCP(Java Community Process)](https://www.jcp.org/en/home/index) 를 통해 [JSR(Java Specification Request)](https://jcp.org/en/jsr/overview) 표준이 정의된다.

2. JSR 내용을 기반으로 각 배급처(Vendor)가 구현한다.

3. [TCK(Technology Compatibility Kit)](https://jcp.org/en/resources/tdk) 를 이용해, JSR 표준에 잘 맞춰져 개발이 되었는지 검증/테스트 한다.

<br>

### OpenJDK 종류

|Community / Vendor|Product name|OSS / Commercial|Architecture|Description|
|------------------|------------|----------------|------------|-----------|
|Oracle|Oracle JDK|Commercial|Hotspot|-|
|OpenJDK.org|OpenJDK|OSS|Hotspot|-|
|RedHat/CentOS|OpenJDK|OSS|Hotspot|-|
|AdoptOpenJDK.net|OpenJDK|OSS|Hotspot/Openj9 (?)|-|
|Eclipse.org|OpenJ9|OSS|OpenJ9(IBM)|-|
|AZUL|Zing|Commercial|Zing|-|

<br>

### 참고

- [LINE의 OpenJDK 적용기: 호환성 확인부터 주의 사항까지](https://engineering.linecorp.com/ko/blog/line-open-jdk/)
- [Java 유료 논쟁(Oracle JDK 와 OpenJDK의 차이 정리)](https://mine-it-record.tistory.com/7)
- [Difference between OpenJDK and Adoptium/AdoptOpenJDK](https://stackoverflow.com/questions/52431764/difference-between-openjdk-and-adoptium-adoptopenjdk)