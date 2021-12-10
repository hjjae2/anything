```java
    public Card draw(){
        int size = cards.size();
        int select = (int)(Math.random()*size);
        Card selectedCard = cards.get(select);
        cards.remove(select);
        return selectedCard;
    }
```

위의 코드는 객체에서 1. 가지고 있는 카드 리스트 중 랜덤한 카드 한장을 뽑고, 2. 그 카드는 리스트에서 제거하는 코드이다.

이 코드를 한번 더 개선할 수 있다고 한다면, 무엇을 개선하면 좋을까??

---

객체를 모델링할 때, 인터페이스 레벨에서 모델링 해보자. 구체적인 구현 사항은 이후에 생각해보자.

---

모델링이 어느정도 익숙해지면, 일반화/추상화에 대해 고민해보자.


> Refrences
> 1. https://jojoldu.tistory.com/62
> 2. https://okky.kr/article/358197