---
layout: post
title: "Trigger vs Application(Business) Logic"
author: "leehyunjae"
tags: ["db"]
---

> 트리거 vs Application Logic 비교해보기


<br>

### Trigger

1. 개발자 입장에서 유지보수가 쉽다.
> 개인적으로 Trigger 가 유지보수가 더 쉬운 것인지 잘 모르겠다.

2. 빌드/배포 시간에 제약이 없다.

3. Application -> DB call 횟수가 줄어든다.

4. 성능(처리율, 속도) 우수하다.

5. (한 액션 시 여러 개의 DB Table 에 삽입/수정 등이 이뤄질 때) Application Logic 으로 관리하면 Exception, Transaction 등을 신경써줘야 한다.

<br>

### Application Logic

1. 개발자 입장에서 유지보수가 어렵다.

2. Trigger 의 경우 위험이 크다.

3. 비즈니스 로직이 크다면 Application 이 더 나을 수 있다.

4. 꼭 필요한 경우에는 Trigger 에 로직이 들어가는 것은 찬성, 그러나 기본적인 로직들은 Application 단에서 이뤄지는 것이 맞다. (단순한 로직, Performance 의 비약적인 향상 등도 Trigger 찬성)

5. H/W 사양 변경, 유지보수 등의 문제가 많이 있다.


<br>

### 결론

여러 글을 읽어보니 개개인의 취향에 따라 의견이 다양한 것 같다. 나의 경우에는 아직은 트리거와 익숙하지 않아서 그런지 Application 단에서 로직을 작성하는 것이 관리, 유지보수 측면에서 더욱 용이하지 않을까라고 생각이 드는 것 같다. 조금 더 직접 경험해보고 비교해보면 더 잘 알 수 있을 것 같다.

<br>

### 참고

- http://gurubee.net/article/68704 (2016년 글)