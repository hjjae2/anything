---
layout: post
title: "JVM에 사설 인증서 적용시키기"
author: "leehyunjae"
tags: [jvm, ssl]
---

> JVM 에 신뢰할 수 있는 인증서 추가하기

<br>

### 개요

['사내(사설) SSL을 사용하는 이유'](https://github.com/hjjae2/Anything/blob/main/ETC/%5BETC%5D%20사내에서%20사설%20인증서를%20사용하는%20이유.md) 글에서 잠깐 적었는데, (로컬 환경)개발을 하면서 이 사내 인증서 때문에 통신 문제가 발생했다.

<br>

### 원인

테스트하고자 하는 작업의 흐름은 간단하게 다음과 같다.

1. Exception 이 터진다.
2. Exception 에 대한 내용을 (사설 인증서가 적용된)Sentry 쪽으로 보낸다.
3. Sentry 에 내용이 쌓이고, 사용자(개발자)에게 알림을 보내준다.

위의 2번의 과정에서 오류가 발생했고, 내가 이해한 원인은 다음과 같다.

(외부에서 받아온)JVM 은 당연히도 사내에서 사용하고 있는 사설 인증서 정보를 가지고 있지 않다. <u>JVM은 공식적으로 인정된 인증서의 리스트만 갖고 있을 것이기 때문이다.</u> 때문에 이 JVM을 통해 (사설 인증서가 적용된)Sentry 와 통신을 하면 '유효하지 않은 인증서'로 취급되어 오류가 발생한다.

<br>

### 해결

다양한 방법이 있겠지만, 이번에 작성할 내용은 JVM에 인증서를 등록하는 방법이다.

**1. 먼저 JDK 를 설치한 폴더에 접근한다.**

`cd $JAVA_HOME`

그러면 `lib/security` 폴더가 보일텐데, 이 폴더에 들어가보면 다음과 같은 파일들이 있다.

```sh
blacklisted.certs
cacerts
default.policy
...
```

**2. 여기서 cacerts 파일을 수정해준다.**

['여기'](https://docs.microfocus.com/SM/9.50/Hybrid/Content/security/concepts/what_is_a_cacerts_file.htm)에서도 알 수 있듯이, cacerts 는 신뢰할 수 있는 인증서 리스트 파일이다. 여기에 (사내에서 사용 중인) 사설 인증서를 등록해주면 신뢰할 수 있다고 보는 것이다.

cacerts 파일을 변경하는 방법이나 구체적인 해결 방법은 ['여기'](https://www.lesstif.com/system-admin/java-validatorexception-keystore-ssl-tls-import-12451848.html)를 참고하면 된다.
