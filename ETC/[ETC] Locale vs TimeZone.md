|Locale|TimeZone|
|-|-|
|The place where something happens. <br><br> The set of settings related to the language and region in which a computer program executes. Examples are language, paper format, currency and time formats, character encoding etc. <br><br> 프로그램이 실행되는 언어(language), 지역(region)과 관련된 설정<br>(예시) 언어, 통화, 시간 형식, 문자 인코딩 등|A vertical region of the globe that somewhat corresponds to longitude, that uses the same time.|

> 출처 : https://wikidiff.com/timezone/locale

<br><br>

## LOCALE

프로그램이 실행되는 언어(language), 지역(region)과 관련된 설정<br>
(예시) 언어, 통화, 시간 형식, 문자 인코딩 등

### Format

```
  언어       지역   문자셋(인코딩)
language_territory.codeset

예시 : ko_KR.UTF-8
```

<br><br>

## TimeZone

그리니치 천문대(본초 자오선, 경도 0도, 0시) 기준으로 지역에 따른 시간의 차이

> 지구의 지역 사이에 생기는 낮과 밤의 차이를 인위적으로 조정하기 위해 고안된 시간의 구분선을 의미한다.

### Format

```
Z 또는 +/- 기호 사용

UTC+0 시간대 (2022년 1월 1일 00시)
2022-01-01T00:00Z
20220101T0000Z

UTC+9 시간대 (2022년 1월 1일 00시)
2022-01-01T00:00+09:00
2022-01-01T00:00+0900
```

> \+ : 2022-01-01T0900+09:00 == 2022-01-01T0000Z 
> \- : 2022-01-01T0000-09:00 == 2022-01-01T0900Z

<br><br>

## ISO8601

날짜, 시간 표기에 대한 국제 표준 규격

나라마다 날짜, 시간 표기가 다른 문제를 해결하기 위한 공통 규격

> https://en.wikipedia.org/wiki/ISO_8601

<br><br>

## RFC3339 (Date and TIme on the Internet : Timestamps)

인터넷 프로토콜 상에서 ISO8601의 프로파일(내용?)을 사용하여 날짜, 시간 형식을 정의한 문서

> ISO8601 은 ISO에서 정한 공통 규격일 뿐, 인터넷과 아무 관계가 없다고 한다. 
>
> 개념적으로 거의 비슷해서, 혼용되어 사용되기도 한다고 한다.

> https://datatracker.ietf.org/doc/html/rfc3339