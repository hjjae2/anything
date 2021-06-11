## Map

가장 흔히 사용되는 구현체는 **HashMap**, **TreeMap**, **LinkedHashMap**, **HashTable** 이 있다.

<br>

###  TreeMap
1. Red-Block Tree 로 구현되었다.
2. C++ 에서의 map 과 유사하다.
3. 대소 비교가 가능해야 한다. (Compariable 인터페이스를 구현해야 한다. compareTo())

<br>

### HashMap

1. Hash Table 로 구현되었다.
2. C++ 에서의 unordered_map 과 유사하다.
3. equals(), hashCode() 를 사용한다.

<br>

### HashTable

1. HashMap + 동기화(synchrosized)
2. Thread-Safe 프로그래밍에서 사용해야 한다.

<br>

### LinkedHashMap

1. 연결리스트로 구현되었다.
2. 순서를 유지한다. (Insert 된 순서를 유지한다.)
3. HashMap 의 subclass 이다.
4. Key 값이 중복된다면, 뒤에 삽입된 것이 유지된다.

Key 값이 중복될 때, 노드가 새로 생기는 것이 아니라 기존 노드에 값이 덮어쓰이는 것에 유의한다.

```
    public void test() {
        LinkedHashMap<String, Integer> linkedHashMap = new LinkedHashMap<>();
        linkedHashMap.put("1", 1);
        linkedHashMap.put("2", 2);
        linkedHashMap.put("3", 3);
        linkedHashMap.put("1", 4);

        for(Map.Entry<String, Integer> entry : linkedHashMap.entrySet()) {
            System.out.println(entry);
            
            // 출력은 아래와 같다.
            // 1=4
            // 2=2
            // 3=3
        }
    }
```

<br>

\* Entry 클래스?
- Map 에 사용되는 Key, Value 형태를 다루기 위한 인터페이스
- Map 인터페이스의 내부 인터페이스

<br>

> Reference
> 1. https://bestalign.github.io/2015/09/20/Java-Map-types-comparison/
> 2. https://m.blog.naver.com/PostView.nhn?blogId=sthwin&logNo=220825616965&proxyReferer=https:%2F%2Fwww.google.com%2F
