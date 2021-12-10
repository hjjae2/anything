---
layout: post
title: "Network :: OSI 7 Layer"
author: "leehyunjae"
tags: ["network"]
---

> OSI 7 Layer 이해하기

### OSI 7 Layer

(전송 시) 7계층 -> 1계층 순서로 헤더를 붙인다. (캡슐화)

(수신 시) 1계층 -> 7계층 순서로 헤더를 떼어낸다. (디캡슐화)

<img src="https://media.vlpt.us/images/cgotjh/post/52907c8c-c149-4943-ad21-3996f44f912f/995EFF355B74179035.jpg">

<br>

**Application Layer (7 Layer)**

- 응용프로그램과 관련된 계층이다.
- 사용자/애플리케이션이 네트워크에 접근할 수 있도록 한다.
- 사용자를 위한 인터페이스를 지원한다. (사용자가 직접적으로 접촉하는 유일한 계층이다.)
- HTTP, FTP, SMTP, DNS 등이 있다.
- 데이터 Format : `Data + HTTP Header`

**Presentation Layer (6 Layer)**

- 데이터를 어떻게 표현할지 결정한다.
- (전송할, 전송된) 데이터의 인코딩, 디코딩, 암호화 등이 이뤄진다. (다음 계층이 이해할 수 있도록)
- 전송 시 : Application Layer 로부터 온 데이터(전송할 데이터)를 변환/압축한다.
- 수신 시 : Application Layer 로 보낼 데이터(수신한 데이터)를 변환/압축한다. 

**Session Layer (5 Layer)**

- 양쪽의 연결을 관리/지속시킨다.
- 세션을 만들고, 유지하고, 종료한다. (복구 기능도 있다고 한다.)
- 세션 계층에서 TCP/IP 세션을 만들고 없앤다.

**Transport Layer (4 Layer)**

- 송/수신자의 (논리적)연결을 담당한다.
- **신뢰성 있는 연결을 유지할 수 있도록 한다.**
- 연결을 생성하고, 데이터를 얼마나 보내고 받았는지, 제대로 데이터를 받았는지 확인한다.
- PORT 를 사용한다.
- TCP, UDP 가 대표적이다.
- 데이터 단위 : `Segment`
- 데이터 Format : `Data + HTTP Header + TCP Header`

**Network Layer (3 Layer)**

- IP(Internet Protocol)이 활용되는 부분이다. 즉 (Routing)경로/목적지를 찾을 수 있도록 한다.
- 장비 : 라우터, L3 스위치가 있다.
- 데이터 단위 : `Packet`
- 데이터 Format : `Data + HTTP Header + TCP Header + IP Header`

**DataLink Layer (2 Layer)**

- 같은 네트워크 내의 단말들에 대해 신뢰성 있는 전송을 보장한다.
- 'MAC Address' 를 활용하여 Endpoint, Switching 장비에 데이터를 전달한다.
- 오류를 찾아내고, 재전송할 수 있다. (**오류 검출**)
- 장비 : Bridge, Switch 가 있다.
- 데이터 단위 : `Frame`
- 데이터 Format : `Data + HTTP Header + TCP Header + IP Header + Ethernet Frame Header` 이다.

**Physical Layer (1 Layer)**

- 물리적으로 데이터가 전송되는 구간이다.
- 데이터가 전기적인 신호로 변한다.
- 데이터의 전달만 한다. 알고리즘, 오류제어 등의 기능이 없다.
- 장비 : Cable, Repeater, Hub

<br>

수신자는 아래의 과정을 거친다고 한다.

1. Ethernet Frame Header 를 보고 목적지 MAC Address 가 자신이 맞는지 확인한다.
2. IP Header 를 확인하여 목적지 IP 가 자신이 맞는지 확인한다.
3. TCP Header 를 확인하여 신뢰성 있는 연결이 필요한지 판단한다. (필요하다면 3-way Handshake 실시하여 연결을 맺는다.)
4. HTTP Header 를 확인하여 어떤 HTTP(Application) 요청인지 확인한다. (현재는 HTTP 로 가정)
5. Data 를 확인한다.

> 즉 위의 Data Format 을 거꾸로 풀어가면서 확인하는 것

<br>

### 참고

1. [OSI 7 Layer 쉽게 이해하기](https://aws-hyoh.tistory.com/50)
2. [네트워크 OSI 7 계층 기본 개념, 각 계층 설명](https://velog.io/@cgotjh/네트워크-OSI-7-계층-OSI-7-LAYER-기본-개념-각-계층-설명)