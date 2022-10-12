## 압축 (Compression)

[HTTP 압축 (1) : 성능 향상을 위한 다른 접근 기법](http://www.simpleisbest.net/archive/2005/07/14/184.aspx) 글을 읽어보면, 왜 압축이 필요한지? 어떤 상황에서 적절한지? 등에 대해 이해할 수 있다.

간단하게 요약하면, 

- 압축을 적용하는 것이 모든 상황에서 이점을 주지 않을 수 있다.
  - (최근에는 옛날에 비해 네트워크 환경이 좋아지면서 압축의 중요성이 떨어질 수도 있으려나..? 싶다.)
- 다만, (성능 향상을 위해) 충분히 고려해볼만한 요소이다.

<br>


[HTTP 압축 (2) : HTTP 압축 작동 원리](http://www.simpleisbest.net/archive/2005/07/18/185.aspx) 글을 읽어보면, (서버-클라이언트 사이에서의) '압축'에 대한 협상 과정을 살펴볼 수 있다. 크게 어려운 부분은 없다. 

**Accept-Encoding** 헤더를 통해, 클라이언트(주로 디바이스, 브라우저일 것이다.)는 자기가 처리할 수 있는 압축 알고리즘을 서버에게 전달한다.

**Content-Encoding** 헤더를 통해, 서버가 선택한 알고리즘을 확인할 수 있다.


> [fiddler](https://www.telerik.com/fiddler)를 통해 직접 확인해볼 수 있다고 한다. (사용해보자.)

<br><br>

## RLE (Run Length Encoding)

예전에 사용되었던 인코딩 방식이라고 한다. 
- 예전에 그래픽을 처리하기 위해 사용된 인코딩 방식이라고 한다.
- 이전 그래픽(이미지)의 경우에는 주변에 비슷한 색(값)들이 활발히 사용되었다.

연속된 동일 문자를 횟수로 축약하는 방식이다. (따라서, 연속되지 않은 글자가 많다면 효율이 떨어진다.)

```sh
AAAAABBCCD
5A2B2C1D
```

<br><br>

## deflate

> deflate : 압축 알고리즘
> inflate : 압축 해제 알고리즘

```sh
# deflate

Raw Data ---> (LZ Encoder) ---> LLD ---> (Huffman Encoder) ---> Compressed Data
```

```sh
# inflate

Raw Data <--- (LZ Encoder) <--- LLD <--- (Huffman Encoder) <--- Compressed Data
```

deflate(inflate) 는 **'허프만 코딩'** 과 **'LZ 인코딩(LZ77, LZSS)'** 알고리즘이 핵심인 압축 방식이라고 한다.

LZ77으로 압축하면 LLD(literal, length, distance)가 나오고, 이것을 허프만 코딩으로 압축하는 것이다.

<br><br>

## gzip

<br><br>

## br

<br><br>

## Huffman Code

> 심볼의 '빈도 수'와 관련된 인코딩 기법

심볼이 나타나는 빈도(확률)에 따라, 사용하는 비트를 결정하고 압축한다. (= 엔트로피 부호화)

예를 들어, `AAAAABBCCD` 문자가 있을 때 다음과 같이 심볼/확률을 표기할 수 있다.

|심볼|확률|
|-|-|
|A|0.5 (5/10)|
|B|0.2 (2/10)|
|C|0.2 (2/10)|
|D|0.1 (1/10)|

위 확률을 오름차순으로 정렬하고, 앞에서부터(낮은 확률부터)(확률 값을 합산해나가면서)트리를 생성해나간다. 

트리를 만든 후엔, 위에서부터 0, 1 비트를 주어 표현한다. 즉, 확률이 높은, 빈도수가 높은 심볼부터 작은 비트로 표현할 수 있게 된다.

<img src="../images/[ETC]%20압축(Compression)_25.png" width="60%">

<img src="../images/[ETC]%20압축(Compression)_12.png" width="60%">

> 자세한 내용은 이후에 더 찾아보자.


<br><br>

## LZ77

> 심볼의 '반복'과 관련된 인코딩 기법

LZ77 은 1977년에 만들어져 LZ77이라고 한다. (LZ1 으로 표현되기도 한다.)

- (반복되는)중복된 문자열을 제거한다.
- 슬라이딩 윈도우가 특징이다.
- 반복되는 문자열이 슬라이딩 윈도우보다 길 경우 압축할 수 없다.
- 다만, 일반적으로는 거의 다 압축된다고 한다. (즉, LZ77로도 거의 충분하다고 한다.)

**LLD(Literal, Length, Distance)**

|단어|설명|
|-|-|
|Literal|일치하지 않는 첫 번째 글자|
|Length|일치하는 문자열의 길이|
|Distance|View 에서부터의 거리|

<img src="../images/[ETC]%20압축(Compression)_18.png" width="60%">

<br><br>

### 참고

- https://youtu.be/Yc_orrKXn1I