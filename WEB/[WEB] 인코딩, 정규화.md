### [WEB] 인코딩, 정규화.md

컴퓨터는 데이터를 바이트 혹은 간단히 숫자로 처리한다. 그러므로 글자를 나타내려면 바이트(숫자) <-> 글자를 대응시켜야 한다. 어떻게 대응(매핑)할지 정한 규칙을 **"인코딩 규약"** 이라고 한다.

<br>

### **ASCII**

알파벳, 숫자, 간단한 문장부호를 포함하여 128개의 글자를 나타낼 수 있다. (7 Bytes, 2^7)

프린트할 수 없는(non-printable) 글자들도 포함된다. `newline(\n)`, `tab`, `carriage return` 등이 있다.

<br>

### **Unicode**

전세계의 언어, 다양한 문구/부호를 표현할 수 있다.

다양한 방식으로 구현이 가능하다. 대표적인 방식으로 `UTF-8`, `UTF-16`의 방법이 있다. (유니코드를 실제 파일 등에 어떻게 기록할 것인지를 표준화한 것이다. 유니코드는 문자를 각 숫자에 대응시킨 것에 불과하고 이를 실제 비트로 표현하는 방식은 다양하다)

`UTF-8` 1\~4 Bytes 를 사용한다. `UTF-16` 2\~4 Bytes 를 사용한다. 두 가지 방식이 존재하는 이유는 **'효율성'** 이다. 서구권 언어(알파벳 등)에서는 대부분의 문자를 1 Byte 만으로 표현할 수 있어서 `UTF-8` 을 사용하는 것이 효율적이지만, 아시아권 언어에서는 `UTF-16` 을 사용할 때 가장 효율적이라고 한다.

각각의 글자(문자)는 구별 가능한 숫자/값(code point)와 대응된다.

```
// 예를 들어 강아지 이모티콘 (🐶)은 U+1F436의 값을 가지고 있다.
Unicode: U+1F436
UTF-8: 4 bytes, 0xF0 0x9F 0x90 0xB6
UTF-16: 4 bytes, 0xD83D 0xDC36
```

아무튼 '합자(combining characters)' 에 대한 처리를 할 때 종종 문제가 발생한다.

```
console.log('\u00e9') // => é
console.log('\u0065\u0301') // => é
console.log('\u00e9' == '\u0065\u0301') // => false
console.log('\u00e9'.length) // => 1
console.log('\u0065\u0301'.length) // => 2
```

(이러한 문제에 대한 해결책, `유니코드 정규화`) 유니코드는 텍스트를 한 가지 규칙을 이용하여 정규화하여 저장하는 것을 권장한다.

정규화 규칙에는 아래 4개의 방식이 존재한다.

- NFC: Normalization Form Canonical Composition : 모든 글자를 분해한 후에, (표준에 명시된 순서에 따라) 다시 합치는 방식이다. 다만, 옛 한글 자모의 결합은 결합하지 못한다. NFC는 많은 GNU/Linux 시스템, Windows에서 주로 사용한다.
- NFD: Normalization Form Canonical Decomposition : 모든 글자를 분해하여 저장한다. NFD는 macOS 시스템에서 주로 사용한다.
- NFKC: Normalization Form Compatibility Composition
- NFKD: Normalization Form Compatibility Decomposition
 
NFKC와 NFKD는 한글자모/한글음절 영역 이외의 한글 유니코드 영역을 처리할 때 유용하게 사용할 수 있다.

<br>

> Refrences
> [번역] 유니코드 문자열을 정규화 해야하는 이유, https://velog.io/@leejh3224/%EB%B2%88%EC%97%AD-%EC%9C%A0%EB%8B%88%EC%BD%94%EB%93%9C-%EC%8A%A4%ED%8A%B8%EB%A7%81%EC%9D%84-%EB%85%B8%EB%A9%80%EB%9D%BC%EC%9D%B4%EC%A7%95-%ED%95%B4%EC%95%BC%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0
> 한글과 유니코드, https://gist.github.com/Pusnow/aa865fa21f9557fa58d691a8b79f8a6d
