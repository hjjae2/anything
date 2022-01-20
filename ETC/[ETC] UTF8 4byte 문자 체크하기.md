
> [[DB] Charset & Collation 글과 연관이 있습니다.](https://github.com/hjjae2/Anything/blob/main/DB/%5BDB%5D%20Charset%20%26%20Collation.md)

Mysql UTF8의 경우 3byte로 디자인되었다. 4byte 문자를 지원하기 위해서는 아래와 같은 방법이 있다.

1. charset/collation 변경 (e.g. utf8mb4)
2. Application 단에서 (utf8)4byte 문자 확인 / 제거

이번 글에서는 2번의 방법을 살펴본다.

<br>
<br>

### (UTF8) 4byte 문자의 종류

UTF8 4byte로 된 문자는 아래 링크에서 확인할 수 있다.

https://design215.com/toolbox/utf8-4byte-characters.php

이모지, musical symbols 등의 다양한 문자가 있다.

<br><br>

### (UTF8) 4byte 문자를 체크하는 방법

아래 링크를 보면 utf8에서 4byte 문자의 코드범위를 확인할 수 있다.

https://docs.oracle.com/cd/E24693_01/server.11203/e10729/appunicode.htm#CACHBDGH


코드 범위를 정규표현식으로 표현해보면 아래와 같이 나올 수 있다. 

즉 아래 표현식을 통해 문자열을 체크하거나 제거할 수 있다.

```text
\xF0[\x90-\xBF][\x80-\xBF]{2}

[\xF1-\xF3][\x80-\xBF]{3}

\xF4[\x80-\x8F][\x80-\xBF]{2}
```
