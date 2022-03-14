## Map

흔히 사용되는 구현체

- HashMap
- LinkedHashMap
- HashTable
- ConcurrentHashMap
- TreeMap

<br><br>

## HashMap

```java
// jdk 11 기준
public class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {
    private static final long serialVersionUID = 362498820763181265L;
    static final int DEFAULT_INITIAL_CAPACITY = 16; // <-- 초기 사이즈

    ...

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        HashMap.Node[] tab;
        int n;
        if ((tab = this.table) == null || (n = tab.length) == 0) {
            n = (tab = this.resize()).length;
        }

        Object p;
        int i;
        if ((p = tab[i = n - 1 & hash]) == null) {
            // 대부분의 정상적인 경우, key (해시) 충돌 없을 때
            tab[i] = this.newNode(hash, key, value, (HashMap.Node)null);
        } else {
            Object e;
            Object k;
            if (((HashMap.Node)p).hash == hash && ((k = ((HashMap.Node)p).key) == key || key != null && key.equals(k))) {
                // 대부분의 정상적인 경우, key (해시) 값이 같고, 동일성(==) 혹은 동등성(equals)일 때 
                // value 를 overwrite 한다.
                // * 동일/동등하기 때문에 해시 충돌이라고 판단하지 않고 그냥 overwrite 하는 것 같다.
                e = p;
            } else if (p instanceof HashMap.TreeNode) {
                e = ((HashMap.TreeNode)p).putTreeVal(this, tab, hash, key, value);
            } else {
                // 해시 충돌 (동일 X , 동등 X) : 동일, 동등하지 않은데 해시는 같음 => 해시 충돌
                // e.g. 
                // 1. key 값은 다른데, (해시 func로 계산된) 해시가 같다.
                // 2. key 값은 같은데, 해시가 다르다. (<-- 일반적인 경우는 아님. 중간에 해시 함수가 변경되어야 나타날수 있는 케이스)
                // --> OpenAddressing : 선형 탐색
                int binCount = 0;

                while(true) {
                    // Loop
                    // 다음 노드가 null 이면, 새 노드 삽입
                    // -> 즉 키가 같은데 , 데이터는 두 개가 됨 (overwrite 안됨)
                    if ((e = ((HashMap.Node)p).next) == null) {
                        ((HashMap.Node)p).next = this.newNode(hash, key, value, (HashMap.Node)null);
                        if (binCount >= 7) {
                            this.treeifyBin(tab, hash);
                        }
                        break;
                    }

                    // 해시 충돌난 것들 중에서도 해시 같은 것(overwrite) 체크
                    if (((HashMap.Node)e).hash == hash && ((k = ((HashMap.Node)e).key) == key || key != null && key.equals(k))) {
                        break;
                    }

                    p = e;
                    ++binCount;
                }
            }

            if (e != null) {
                V oldValue = ((HashMap.Node)e).value;
                if (!onlyIfAbsent || oldValue == null) {
                    ((HashMap.Node)e).value = value;
                }

                this.afterNodeAccess((HashMap.Node)e);
                return oldValue;
            }
        }

        ...
    }
```


```java
// hashcode : 충돌
// equals : 구현
public class MyTest {

    class Node {
        public String key;

        public Node(String key) {
            this.key = key;
        }

        @Override
        public int hashCode() {
            return 1;
        }

        @Override
        public boolean equals(Object obj) {
            Node node = (Node) obj;
            return this.key == node.key;
        }

        @Override
        public String toString() {
            return "Node{" +
                   "key='" + key + '\'' +
                   '}';
        }
    }

    @Test
    public void test() {
        HashMap<Node, String> map = new HashMap<>();
        map.put(new Node("1"), "value1");
        map.put(new Node("2"), "value2"); // 1번 데이터와 해시 같고, 동등(equals)하지 않음 => 해시 충돌 (새로운 데이터로 삽입)
        map.put(new Node("2"), "value3"); // 2번 데이터와 해시 같고, 동등함 => overwrite

        for (Map.Entry<Node, String> m: map.entrySet()) {
            System.out.println(m.getKey());
            System.out.println(m.getValue());
        }

        /**
         * Node{key='1'}
         * value1
         * Node{key='2'}
         * value3
         */
    }
}
```

```java
// hashcode : 충돌
// equals : 구현 X
public class MyTest {

    class Node {
        public String key;

        public Node(String key) {
            this.key = key;
        }

        @Override
        public int hashCode() {
            return 1;
        }

        @Override
        public String toString() {
            return "Node{" +
                   "key='" + key + '\'' +
                   '}';
        }
    }

    @Test
    public void test() {
        HashMap<Node, String> map = new HashMap<>();
        map.put(new Node("1"), "value1");
        map.put(new Node("2"), "value2"); // 1번 데이터와 해시 충돌 -> 새로운 값 삽입
        map.put(new Node("2"), "value3"); // 2번 데이터와 해시 충돌 -> 새로운 값 삽입

        for (Map.Entry<Node, String> m: map.entrySet()) {
            System.out.println(m.getKey());
            System.out.println(m.getValue());
        }

        /**
         * Node{key='1'}
         * value1
         * Node{key='2'}
         * value2
         * Node{key='2'}
         * value3
         */
    }
}
```

\* **HashMap 에서는 `hashcode()`, `equals()` 둘 다 사용**
- 어떻게, 어떤 것을 구현했냐에 따라 결과 값이 달라짐

\* **충돌 해결 방법 : OpenAddressing 선형 탐색**
- Node 가 next 속성을 가지고 있음

<br>
<br>


## HashTable

1. HashMap + 동기화(synchrosized)
2. Thread-Safe (synchronized)
3. value 에 null 값 불가 : `NullPointerException`


\* **아래와 같이 대부분의 메소드에 synchronized 키워드가 적용**

\* **value == null 이면 NullPointerException**
```java
public class Hashtable<K, V> extends Dictionary<K, V> implements Map<K, V>, Cloneable, Serializable {

    ...

    public synchronized V put(K key, V value) { // method (인스턴스) 동기화 
        if (value == null) {
            throw new NullPointerException(); // NPE
        } else {
            Hashtable.Entry<?, ?>[] tab = this.table;
            int hash = key.hashCode();
            int index = (hash & 2147483647) % tab.length;

            for(Hashtable.Entry entry = tab[index]; entry != null; entry = entry.next) {
                if (entry.hash == hash && entry.key.equals(key)) {
                    V old = entry.value;
                    entry.value = value;
                    return old;
                }
            }

            this.addEntry(hash, key, value, index);
            return null;
        }
    }

    public synchronized V remove(Object key) {
        Hashtable.Entry<?, ?>[] tab = this.table;
        int hash = key.hashCode();
        int index = (hash & 2147483647) % tab.length;
        Hashtable.Entry<K, V> e = tab[index];

        for(Hashtable.Entry prev = null; e != null; e = e.next) {
            if (e.hash == hash && e.key.equals(key)) {
                if (prev != null) {
                    prev.next = e.next;
                } else {
                    tab[index] = e.next;
                }

                ++this.modCount;
                --this.count;
                V oldValue = e.value;
                e.value = null;
                return oldValue;
            }

            prev = e;
        }

        return null;
    }

    public synchronized void putAll(Map<? extends K, ? extends V> t) {
        ...
    }
```


<br><br>

## LinkedHashMap

1. (double)연결리스트 (순서 유지)
2. HashMap 의 subclass

> HashMap 의 예시/출력 동일하다.

```java
public class LinkedHashMap<K, V> extends HashMap<K, V> implements Map<K, V> {
    ...
}
```

```java
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

<br><br>

##  TreeMap
1. Red-Black Tree 로 구현되었다.
2. C++ 에서의 map 과 유사하다.
3. 대소 비교가 가능해야 한다. (Compariable 인터페이스를 구현해야 한다. compareTo())

> Red-Black Tree 란? Balacned Binary Search Tree 라고 볼 수 있다.

<br>
<br>

## \* Entry 클래스?
- Map 에 사용되는 Key, Value 형태를 다루기 위한 인터페이스
- Map 인터페이스의 내부 인터페이스

```java
// 예시 : HashMap.Node 구현체
static class Node<K, V> implements Entry<K, V> {
        final int hash;
        final K key;
        V value;
        HashMap.Node<K, V> next;

        ...
}
```

<br>

> Reference
> 1. https://bestalign.github.io/2015/09/20/Java-Map-types-comparison/
> 2. https://m.blog.naver.com/PostView.nhn?blogId=sthwin&logNo=220825616965&proxyReferer=https:%2F%2Fwww.google.com%2F
