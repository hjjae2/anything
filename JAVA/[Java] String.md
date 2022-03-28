### contains()

```java
public boolean contains(CharSequence s) {
    return this.indexOf(s.toString()) >= 0;
}
```

```java
public static int indexOf(byte[] value, int valueCount, byte[] str, int strCount, int fromIndex) {
        byte first = str[0];
        int max = valueCount - strCount;

        for(int i = fromIndex; i <= max; ++i) {
            if (value[i] != first) {
                do {
                    ++i;
                } while(i <= max && value[i] != first);
            }

            if (i <= max) {
                int j = i + 1;
                int end = j + strCount - 1;

                for(int k = 1; j < end && value[j] == str[k]; ++k) {
                    ++j;
                }

                if (j == end) {
                    return i;
                }
            }
        }

        return -1;
    }
```

1. 시작 인덱스부터 for-loop 탐색 시작
2. 첫 글자 같다면, 검사 시작
3. 중간에 틀리면 다시 2번부터 시작
4. 같다면 시작 인덱스 반환