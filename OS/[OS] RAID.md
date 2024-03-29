## Redundant Array Inexpensive(Independent) Disk

> (초창기에는) "저렴한 디스크를 사용해서 구성한다는 느낌으로" 'Inexepnsive' 키워드를 사용했다고 한다.
> 
> "독립적인 저장 공간 구성한다는 느낌으로(즉, 안정성, 가용성)",'Independent' 키워드를 사용한다고 한다.


<br>

### 개요

**RAID 특징**

1. 고가용성 (안정성) 향상
2. 디스크 I/O 성능 향상
3. 디스크 확장성 향상

<br>

**RAID 구성은 'HW 방식'과 'SW 방식'이 있다.**

### 1. HW 방식

HW 방식은 메인보드에 'RAID 컨트롤러 카드' 라는 것을 장착하여 구현한다고 한다.

운영체제 부팅 진입 전, RAID 구성을 확인한다고 한다.

> 'RAID 컨트롤러 카드' : RAID 를 컨트롤하기 위한 장치

<br>

### 2. 펌웨어 방식

간소화된 HW 방식이라고 볼 수 있다.

RAID 칩을 메인보드에 탑재(펌웨어)한 방식이라고 한다.

<br>

### 3. SW 방식

SW를 통해 RAID 를 구성하는 방식이다.

> 스트라이핑(striping) : 데이터를 분할하여(번갈아가며) 저장하는 방식
> = 'I/O 작업'을 디스크 수 만큼 분할하여 처리하기 때문에 성능이 향상된다고 한다.
> 
> 미러링(mirroring) : replication 과 같이 데이터를 복제하여 저장한다는 의미

|방식|설명|특징|
|-|-|-|
|RAID 0|스트라이핑 구성 <br><br>* 성능을 최우선으로 할 때 사용한다고 한다.|성능 : ★★★★★|
|RAID 1|미러링(replication 과 같이) 방식으로 데이터 저장|- 가용성(안전성) : ★★★★★ <br>- 비용 : ★ (비용 n 배)<br> - 읽기 성능 : ↑ <br>- 쓰기 성능 : ↓ (악간)|
|RAID 2|(bit 단위)스트라이핑 구성 <br> 패리티 비트(오류 검사, 수정) 기능 추가<br>즉, '실제 사용하는 디스크', '오류 핸들링을 위해 사용하는 디스크'로 분리<br><br>* RAID 3, RAID 4에 비해 이점이 없어, 현재는 사용되지 않는 추세라고 한다.||
|RAID 3|(byte 단위)스트라이핑 구성 <br>**디스크 1개 : 패리티 비트 용도 사용** <br><br> * 너무 작은 단위(byte)로 스트라이핑하기에 현재는 사용되지 않는 추세라고 한다.||
|RAID 4|(block 단위) RAID 3과 동일한 방식(스트라이핑 + 디스크 1개 : 패리티)<br> (패리티 용도의)디스크가 장애 시 '에러 핸들링 불가' <br><br> * block 단위로 저장할 경우, 파일의 사이즈가 작다면 하나의 블록(디스크)에서 데이터를 가져올 수 있기에, (비교적)성능 향상을 기대할 수 있다.<br>* RAID 4 역시 RAID 5 로 많이 대체되었다고 한다.||
|RAID 5|(block 단위) RAID 4와 동일한 방식 <br><br> **단, 패리티 디스크를 특정한 디스크로 지정하지 않고, 매 번 다른 디스크에 패리티 정보를 저장한다는 차이**<br>= (패리티 용도의) 디스크가 특정되지 않았기에, RAID 4의 '패리티 디스크' 장애 시의 문제를 (비교적)해결<br>= (어떤 디스크라도)하나의 디스크 장애 시, 복구 가능|- 성능 : (단일 디스크 사용 방식 대비) `전체 디스크 수 - 1`<br><br> - 용량 : (단일 디스크 사용 방식 대비) `전체 디스크 수 - 1`|
|RAID 6|RAID 5 + 2차 (패리티 용도 디스크) 추가<br><br>**(일반적인 경우) 2개의 디스크에 장애가 나도 핸들링 가능 (= 설계 목적)** ||
|중첩 RAID<br>- RAID01<br>- RAID10<br>- RAID50<br>- ...|||

<br>

> *" RAID의 주 사용 목적은 크게 무정지 구현(안정성)과 고성능 구현으로 구분된다.<br>
> 무정지 구현을 극도로 추구하면 RAID 1, <br>
> 고성능 구현을 극도로 추구하면 RAID 0이 되며 <br>
> RAID 5, 6은 둘 사이에서 적당히 타협한 형태 <br>
> RAID 10이나 RAID 01과 같이 두 가지 방식을 혼용하는 경우도 있다. "*
> 
> 출처: https://zetastring.tistory.com/121

<br><br>

### 참고

- https://www.stevenjlee.net/2020/03/01/이해하기-raid-구현-방식과-종류에-대하여/
- https://zetastring.tistory.com/121