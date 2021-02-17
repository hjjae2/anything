### unordered_map vs map

**결론**

1. 데이터가 많다면 unordered_map 을 사용하자.
2. 대게 unordered_map 이 우수하다.

<br>

**개념**

map
1. Red-Black Tree (RB Tree) 기반 -> key 정렬
2. 키 값이 고르지 못할 경우, balancing 비용 커짐
3. O(logN)

unordered_map
1. Hash Table 기반
2. O(1)

<br>

**비교**

Key 타입: Int

성능: unordered_map > map

<br>

Key 타입: 문자열

성능: unordered_map ?? map

1. 문자열의 대소 비교가 쉽다면, map 의 정렬 비용이 적다. (map 이 우수할 수 있다.)
2. 문자열의 길이가 짧다면 unordered_map 의 해싱 비용이 적다. (unordered_map 이 우수할 수 있다.)

<br>

> 출처: https://gracefulprograming.tistory.com/3
