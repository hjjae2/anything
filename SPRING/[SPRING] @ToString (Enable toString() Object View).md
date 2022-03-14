
> Intellij 디버깅 시, 
> 
> Build, Execution, Deployment > Debugger > Data Views > Java > `Enable toString() Object View` 설정이 활성화되어 있다는 가정


**Lazy 연관관계를 가진 Entity 조회 시, Lazy 객체의 초기화(Fetch) 시점이 예상과 다를 수 있다.**


<br>

### 예시

```java
...
@ToString
@Entity
public class Poster {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "poster_type_id")
    private PosterType posterType;

    ...
```

```java
public void test() {
    List<Poster> allPosters = posterRepository.findAll();

    Poster poster = allPosters.get(0); // <-- (1) 이때 쿼리 발생

    System.out.println(poster.getPosterType()); // <-- (2) 이때 Lazy 객체가 사용되니까, 이때 쿼리가 발생하는 것 예상
}
```

<br>

`(1)` 시점에 쿼리 발생

<br>

```java
...

Hibernate: select user0_.user_id as user_id1_3_0_, user0_.email as email2_3_0_, user0_.is_activated as is_activ3_3_0_, user0_.is_authenticated as is_authe4_3_0_, user0_.name as name5_3_0_, user0_.oauth_type as oauth_ty6_3_0_, user0_.phone as phone7_3_0_, user0_.role as role8_3_0_ from user user0_ where user0_.user_id=?

...

Hibernate: select postertype0_.poster_type_id as poster_t1_2_0_, postertype0_.height as height2_2_0_, postertype0_.type as type3_2_0_, postertype0_.width as width4_2_0_ from poster_type postertype0_ where postertype0_.poster_type_id=?
```

<br>

### 이유

(Intellij 에서) 디버깅 시 Object 의 상태를 보여주기 위해 데이터를 조회 <br>
(* Lazy 초기화 조건과 마찬가지로, 실제 데이터가 필요하니까 데이터를 조회)


**toString 있을 때**

<img src="/images/toString%20ON.png">

<br><br>

**toString 없을 때**

<img src="/images/toString%20OFF.png">

<br><br>

**아래 설정을 해제하거나 `@ToString` 을 제거하면 예상한것과 같이 동작하는 것을 볼 수 있다.**

<img src="/images/toString%20Option.png">


<br><br>

> Resolve https://github.com/hjjae2/Anything/issues/15
