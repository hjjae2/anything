> Mysql 기준

### Mysql에서 제공하는 Charset 종류

아래의 명령어로 확인할 수 있다.

```sql
show character set ;
```

<br>

대표적으로 아래와 같다.
- utf8
- utf8mb4
- ascii
- euckr

<br>

**특징 (중요!!)**

- Mysql `utf8`은 3byte까지 지원한다.
- 4byte 문자를 사용하기 위해서는 `utf8mb4`를 사용한다. 
  - utf16, utf32도 4byte를 지원한다.

<br><br>

### Mysql에서 제공하는 Collation 종류

아래의 명령어로 확인할 수 있다.

```sql
show collation ;
```

<br>

대표적으로 아래와 같다.
- `utf8_bin`, `utf8mb4_bin` (utf8, utf8mb4)
  - binary 값(hex)으로 비교한다.
- `utf8_general_ci`, `utf8mb4_general_ci` (utf8, utf8mb4)
  - a, b, ... 순서 (즉 휴머니스틱하게 정렬)
- `utf8_unicode_ci`, `utf8mb4_unicode_ci` (utf8, utf8mb4)
  - utf8_general_ci 보다 조금 더 휴머니스틱하게 정렬한다고 한다.
  - (우리나라에서 잘 사용되지 않는)특수 문자들의 순서가 다르다고 한다.