
> [[DB] Charset & Collation 글과 연관이 있습니다.](https://github.com/hjjae2/Anything/blob/main/DB/%5BDB%5D%20Charset%20%26%20Collation.md)

Mysql UTF8의 경우 3byte로 디자인되었다. 4byte 문자를 처리하기 위해서는 아래와 같은 방법이 있다.

1. DB 스키마 변경 : charset/collation (e.g. utf8mb4)
2. Application 처리 :  (utf8)4byte 문자 확인 / 제거 등

<br>

**이번 글에서는 2번의 방법을 살펴본다.**

<br>

### (UTF8) 4byte 문자의 종류

UTF8 4byte로 된 문자는 아래 링크에서 확인할 수 있다.

https://design215.com/toolbox/utf8-4byte-characters.php

이모지, musical symbols 등의 다양한 문자가 있다.

<br><br>

### (UTF8) 4byte 문자를 체크하는 방법

> 간단하게는, BMP 영역에 있는지를 체크하면 된다.

아래 링크를 보면 utf8에서 4byte 문자의 코드범위를 확인할 수 있다.

https://docs.oracle.com/cd/E24693_01/server.11203/e10729/appunicode.htm#CACHBDGH

이것을 대략적으로 UTF8 인코딩 규칙과 비교해보면,

|Byte1|Byte2|Byte3|Byte4|
|-|-|-|-|
|0xxxxxxx|-|-|-|
|110xxxxx|10xxxxxx|-|-|
|1110xxxx|10xxxxxx|10xxxxxx
|11110xxx|10xxxxxx|10xxxxxx|10xxxxxx|

```text
1 Byte       : F0 = 11110000 ~ F4 = 11110100

2, 3, 4 Byte : 80 = 10000000 ~ BF = 10111111
```


<br>

또, 코드 범위를 정규표현식으로 표현해보면, 

```text
\xF0[\x90-\xBF][\x80-\xBF]{2}

[\xF1-\xF3][\x80-\xBF]{3}

\xF4[\x80-\x8F][\x80-\xBF]{2}
```


<br><br>

### 참고

JAVA(UTF16) 에서는 아래와 같이 체크할 수 있다.

```java
Character.isSurrogate(ch) // true : BMP 외, false : BMP
```


```java
public static boolean isSurrogate(char ch) {
    return ch >= MIN_SURROGATE && ch < (MAX_SURROGATE + 1);
}

// MIN_SURROGATE = '\uD800';
// MAX_SURROGATE = '\uDFFF';
```


> " *Surrogate pair는 UTF-16 인코딩을 위해서만 사용하도록 지정된 유니코드 영역으로, U+D800 ~ U+DBFF(high surrogates)와 U+DC00 ~ U+DFFF(low surrogates)가 존재합니다. BMP 영역을 벗어나는 문자(supplementary plane 영역)는 UTF-16에서 high surrogate와 low surrogate의 조합으로 표현됩니다. (두 surrogate가 짝이 맞지 않는다면 인코딩이 잘못된 것입니다.)* " 
> 
> http://klutzy.github.io/blog/2014/06/20/unicode/
