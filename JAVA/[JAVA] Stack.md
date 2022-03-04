**Thread-safe**
- Vector 클래스 상속 
  - push, size => Vector 클래스의 메서드 사용
- synchronized 키워드 사용
  - method level


<br>

- push() : (Vector) synchronized
- pop() : synchronized
- peek() : synchronized
- empty() : (Vector) synchronized
- search() : synchronized

```java
public class Stack<E> extends Vector<E> {
    ...

    public E push(E item) {
        addElement(item); // (Vector) synchronized

        return item;
    }

    // synchronized
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
    }

    // synchronized
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }

    public boolean empty() {
        return size() == 0; // (Vector) synchronized
    }

    // synchronized
    public synchronized int search(Object o) {
        int i = lastIndexOf(o); // (Vector) synchronized

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }
}
```