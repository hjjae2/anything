
**평면(Plain)**

- 0 ~ 2^16 범위를 표현하는 코드표(세트)
- 총 17개(0 ~ 16개)의 평면(plaon) 존재
- **BMP(Basic multilingual plane) : 0번째 평면(plain)**
  - 거의 대부분의 문자는 BMP 에 속함
  - 다만 이모지와 같은 최근에 사용되기 시작한 문자들은 속하지 않음

<br>

### UCS-2 (Universal Character Set)

- BMP(기본 다국어 평면)만 인코딩
- (고정 사이즈) 문자당 2byte 표현
  - '평면' 을 구분하는데 2byte가 사용되는데, UCS-2는 BMP 고정이기 때문에 평면 구분 byte가 없어도 됨
  - BMP 외의 문자는 표현할 수 없음


<br>

### UTF-32 (Unicode Transformation Format)

- (고정 사이즈) 4byte 인코딩 방식 : 평면 구분(2byte) + 문자(2byte)
  - 모든 유니코드 문자 표현할 수 있음
- (단점) 용량 차지
  - 1byte로 표현할 수 있는 것도 4byte 고정
  - 거의 사용되고 있지 않음

<br>

### UTF-16

- 가변 사이즈(2byte, 4byte) 인코딩
  - 2byte : BMP
  - 4byte : BMP 외의 유니코드 문자 (**UTF32 와 같은 형태가 아님에 주의!!**)
  - Surrogate(서러게이트)영역 문자를 구분하기 위해 (조금 복잡한?) 인코딩 규칙이 있음
- Java 의 인코딩 방식
- 단점 : 아스키 코드(1byte) 호환되지 않음

> " *Surrogate pair는 UTF-16 인코딩을 위해서만 사용하도록 지정된 유니코드 영역으로, U+D800 ~ U+DBFF(high surrogates)와 U+DC00 ~ U+DFFF(low surrogates)가 존재합니다. BMP 영역을 벗어나는 문자(supplementary plane 영역)는 UTF-16에서 high surrogate와 low surrogate의 조합으로 표현됩니다. (두 surrogate가 짝이 맞지 않는다면 인코딩이 잘못된 것입니다.)* " 
> 
> http://klutzy.github.io/blog/2014/06/20/unicode/

> " *그렇기 때문에 UTF-16의 앞의 6비트를 확인했을 때 110110이나 110111로 시작하지 않는다면 거기서부터 2바이트는 무조건 기본 다국어 평면 상에 있는 문자라고 확신할 수 있게 됩니다. 만약 반대로 110110으로 시작한다면 기본 다국어 평면 외의 문자라고 단언할 수 있게 됩니다. 위의 110110으로 시작하는 문자나, 110111로 시작하는 문자를 서러게이트(Surrogate) 영역 문자라고 합니다.* "
> 
> https://dingue.tistory.com/16

> **BMP 는 2byte 로 표현되는데, BMP 가 아닌 것을 인지하기 위해 사용되는 부분인 것 같다. 때문에 BMP에서 사용되지 않는 영역(110110, 110111)이 사용되는 것으로 이해** 


<br>

### UTF-8

- 가변 사이즈(1byte ~ 4byte)
  - 1byte : ASCII
  - 2byte : 아랍, 히브리, 유럽계 문자
  - 3byte : 1,2 byte 외의 BMP 문자
  - 4byte : BMP 외 문자 (e.g. 한글)
  - 인코딩 시 헤더값으로 붙는 것들이 있기 때문에 BMP 3byte 표현됨
- 아스키 코드 호환 OK
- 인코딩 시 아래와 같은 규칙을 가진다.

|Byte1|Byte2|Byte3|Byte4|
|-|-|-|-|
|0xxxxxxx|-|-|-|
|110xxxxx|10xxxxxx|-|-|
|1110xxxx|10xxxxxx|10xxxxxx
|11110xxx|10xxxxxx|10xxxxxx|10xxxxxx|

<br>


### 기타
- 엔디안 방식에 따라, 문자가 다르게 표현될 수 있음 -> BOM(Byte Order Mark)를 파일 첫 부분에 명시(엔디안 방식 명시)

<br><br>

### 참고

1. [아스키 코드, 유니코드 그리고 UTF-8, UTF-16](https://dingue.tistory.com/16)
2. http://klutzy.github.io/blog/2014/06/20/unicode/