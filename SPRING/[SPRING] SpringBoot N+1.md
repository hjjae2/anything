## N+1 문제

(쿼리를 통해 데이터를 가져올때) (연관관계에 있는)데이터를 얻기 위해 추가적인 쿼리가 발생하는 문제

<br>

### 예시 : 단건 조회

```java
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class Orders {
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    private Long id;

    @Column
    private String name;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "customer_id")
    private Customer customer;

    @OneToMany(fetch = FetchType.EAGER, mappedBy = "orders")
    private List<OrderItem> orderItems;
}

```

**EAGER 방식**

left join , inner join 으로 한번에 가져온다.  <br>
> \* optional = false/true 에 따라 left, inner join 결정


```sql
select orders.id          
       orders.customer_id 
       orders.name        
       customer.id        
       customer.name      
       order_item.order_id
       order_item.id      
       order_item.id      
       order_item.name   
       order_item.order_id
from orders
         left outer join customer on orders.customer_id = customer.id
         left outer join order_item on orders.id = order_item.order_id
where orders.id = ?
```

**LAZY 방식**

```sql
select orders.id, orders.customer_id, orders.name
from orders
where orders.id = ?
```

<br>

### 사용자 정의 메서드 단건 조회

```java
Optional<Orders> findByName(String name);
```

**EAGER 방식**

```sql
select orders.id, orders.customer_id, orders.name
from orders orders
where orders.name = ?

select customer.id, customer.name
from customer
where customer.id = ?

select order_item.order_id
       order_item.id      
       order_item.id      
       order_item.name    
       order_item.order_id
from order_item
where order_item.order_id = ?
```


**LAZY 방식**

```sql
select orders.id, orders.customer_id, orders.name
from orders orders
where orders.name = ?

-- (단, 해당 객체 필요/사용 시 발생)
select customer.id, customer.name
from customer
where customer.id = ?

-- (단, 해당 객체 필요/사용 시 발생)
select order_item.order_id
       order_item.id      
       order_item.id     
       order_item.name   
       order_item.order_id
from order_item
where order_item.order_id = ?
```


<br>

### 예시 : 리스트 조회

**EAGER 방식**

```sql
-- orders 전체 조회 쿼리
select orders.id, orders.customer_id, orders.name
from orders

-- customer 조회 쿼리
select customer.id, customer.name
from customer customer
where customer.id = ?

-- order_item 조회 쿼리
select order_item.order_id
       order_item.id      
       order_item.id      
       order_item.name    
       order_item.order_id
from order_item
where order_item.order_id = ?
```

**LAZY 방식**

```sql
select orders.id, orders.customer_id, orders.name
from orders

-- (단, 해당 객체 필요/사용 시 발생)
select customer.id, customer.name
from customer customer
where customer.id = ?

-- (단, 해당 객체 필요/사용 시 발생)
select order_item.order_id
       order_item.id      
       order_item.id      
       order_item.name    
       order_item.order_id
from order_item
where order_item.order_id = ?
```

<br>

### 사용자 정의 메서드 리스트 조회

```java
List<Orders> findAllByName(String name);
```

**EAGER 방식**

```sql
select orders.id, orders.customer_id, orders.name
from orders
where orders.name = ?

select customer.id, customer.name
from customer customer
where customer.id = ?

select order_item.order_id
       order_item.id      
       order_item.id      
       order_item.name    
       order_item.order_id
from order_item
where order_item.order_id = ?
```


**LAZY 방식**

```sql
select orders.id, orders.customer_id, orders.name
from orders
where orders.name = ?

-- (단, 해당 객체 필요/사용 시 발생)
select customer.id, customer.name
from customer customer
where customer.id = ?

-- (단, 해당 객체 필요/사용 시 발생)
select order_item.order_id
       order_item.id      
       order_item.id      
       order_item.name    
       order_item.order_id
from order_item
where order_item.order_id = ?
```


<br>

--- 
## 이전 작성 글



```text
Person person * --- 1 Company compnay
```

<br>

### 첫 번째 예시

1. Person 객체의 리스트를 조회하고
2. Person 객체의 엮인 Company 객체를 조회한다고 가정<br>
> \* Person 객체에 엮인 Company 는 모두 다르다고 가정

```java
class Test {
    @Autowired TestEntityManager entityManager;
    @Autowired PersonRepository personRepository;
    @Autowired CompanyRepository companyRepository;

    @BeforeEach
    void setUp() {
        System.out.println("------------------------ insert ------------------------");
        Company company1 = Common.getCompany("c1");
        Company company2 = Common.getCompany("c2");
        Company company3 = Common.getCompany("c3");
        companyRepository.save(company1);
        companyRepository.save(company2);
        companyRepository.save(company3);

        personRepository.save(Common.getPerson("p1", company1));
        personRepository.save(Common.getPerson("p2", company2));
        personRepository.save(Common.getPerson("p3", company3));

        entityManager.flush();
        entityManager.clear();
    }

    @Test
    void test() {
        System.out.println("------------------------ test ------------------------");
        List<Person> people = personRepository.findAll();
        
        // FetchType 이 Eager 방식이라면 아래 로직은 생략한다고 가정한다.
        for (Person person : people) {
            System.out.println(person.getCompany().getName());
        }
    }
```

그러면 아래와 같은 쿼리가 발생할 것이다. 

```sql
-- 예상 로그
select * from person;   -- 1 번


select * from company where company_id = ?  -- N 번
select * from company where company_id = ?
select * from company where company_id = ?
...

-- 실제 로그
------------------------ insert ------------------------
Hibernate: insert into company (id, created_date, updated_date, address, email, person_id, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into company (id, created_date, updated_date, address, email, person_id, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into company (id, created_date, updated_date, address, email, person_id, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into person (id, created_date, updated_date, address, company_id, email, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into person (id, created_date, updated_date, address, company_id, email, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into person (id, created_date, updated_date, address, company_id, email, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)

------------------------ test ------------------------
Hibernate: select person0_.id as id1_4_, person0_.created_date as created_2_4_, person0_.updated_date as updated_3_4_, person0_.address as address4_4_, person0_.company_id as company_9_4_, person0_.email as email5_4_, person0_.name as name6_4_, person0_.phone as phone7_4_, person0_.remark as remark8_4_ from person person0_
Hibernate: select company0_.id as id1_1_0_, company0_.created_date as created_2_1_0_, company0_.updated_date as updated_3_1_0_, company0_.address as address4_1_0_, company0_.email as email5_1_0_, company0_.person_id as person_i9_1_0_, company0_.name as name6_1_0_, company0_.phone as phone7_1_0_, company0_.remark as remark8_1_0_ from company company0_ where company0_.id=?
Hibernate: select company0_.id as id1_1_0_, company0_.created_date as created_2_1_0_, company0_.updated_date as updated_3_1_0_, company0_.address as address4_1_0_, company0_.email as email5_1_0_, company0_.person_id as person_i9_1_0_, company0_.name as name6_1_0_, company0_.phone as phone7_1_0_, company0_.remark as remark8_1_0_ from company company0_ where company0_.id=?
Hibernate: select company0_.id as id1_1_0_, company0_.created_date as created_2_1_0_, company0_.updated_date as updated_3_1_0_, company0_.address as address4_1_0_, company0_.email as email5_1_0_, company0_.person_id as person_i9_1_0_, company0_.name as name6_1_0_, company0_.phone as phone7_1_0_, company0_.remark as remark8_1_0_ from company company0_ where company0_.id=?
```

<br>

### 두 번째 예시

1. Company 객체의 리스트를 조회하고
2. Company 객체에 엮인 Person List 를 조회한다고 해보자!

```java
@DataJpaTest
class Test {
    @Autowired TestEntityManager entityManager;
    @Autowired PersonRepository personRepository;
    @Autowired CompanyRepository companyRepository;

    @BeforeEach
    void setUp() {
        Company company1 = Common.getCompany("c1");
        Company company2 = Common.getCompany("c2");
        Company company3 = Common.getCompany("c3");
        companyRepository.save(company1);
        companyRepository.save(company2);
        companyRepository.save(company3);

        personRepository.save(Common.getPerson("p1", company1));
        personRepository.save(Common.getPerson("p2", company2));
        personRepository.save(Common.getPerson("p3", company3));
        
        entityManager.flush();
        entityManager.clear();
    }

    @Test
    void test() {
        System.out.println("------------------------ test ------------------------");
        List<Company> companies = companyRepository.findAll();

        for (Company company : companies) {
            System.out.println(company.getPeople().size());
        }
    }
```

그러면 아래와 같은 쿼리가 발생할 것이다. 

```sql
-- 예상 로그
select * from company;  -- 1 번

select * from person where company_id = ?   -- N 번
select * from person where company_id = ?
select * from person where company_id = ?
...

-- 실제 로그
------------------------ insert ------------------------
Hibernate: insert into company (id, created_date, updated_date, address, email, person_id, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into company (id, created_date, updated_date, address, email, person_id, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into company (id, created_date, updated_date, address, email, person_id, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into person (id, created_date, updated_date, address, company_id, email, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into person (id, created_date, updated_date, address, company_id, email, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: insert into person (id, created_date, updated_date, address, company_id, email, name, phone, remark) values (null, ?, ?, ?, ?, ?, ?, ?, ?)

------------------------ test ------------------------
Hibernate: select company0_.id as id1_1_, company0_.created_date as created_2_1_, company0_.updated_date as updated_3_1_, company0_.address as address4_1_, company0_.email as email5_1_, company0_.person_id as person_i9_1_, company0_.name as name6_1_, company0_.phone as phone7_1_, company0_.remark as remark8_1_ from company company0_
Hibernate: select people0_.company_id as company_9_4_0_, people0_.id as id1_4_0_, people0_.id as id1_4_1_, people0_.created_date as created_2_4_1_, people0_.updated_date as updated_3_4_1_, people0_.address as address4_4_1_, people0_.company_id as company_9_4_1_, people0_.email as email5_4_1_, people0_.name as name6_4_1_, people0_.phone as phone7_4_1_, people0_.remark as remark8_4_1_ from person people0_ where people0_.company_id=?
Hibernate: select people0_.company_id as company_9_4_0_, people0_.id as id1_4_0_, people0_.id as id1_4_1_, people0_.created_date as created_2_4_1_, people0_.updated_date as updated_3_4_1_, people0_.address as address4_4_1_, people0_.company_id as company_9_4_1_, people0_.email as email5_4_1_, people0_.name as name6_4_1_, people0_.phone as phone7_4_1_, people0_.remark as remark8_4_1_ from person people0_ where people0_.company_id=?
Hibernate: select people0_.company_id as company_9_4_0_, people0_.id as id1_4_0_, people0_.id as id1_4_1_, people0_.created_date as created_2_4_1_, people0_.updated_date as updated_3_4_1_, people0_.address as address4_4_1_, people0_.company_id as company_9_4_1_, people0_.email as email5_4_1_, people0_.name as name6_4_1_, people0_.phone as phone7_4_1_, people0_.remark as remark8_4_1_ from person people0_ where people0_.company_id=?
```

<br>

**즉, 프록시 객체로 되어있는, 연관관계가 있는 엔티티라면 언제든지 발생할 수 있는 것으로 이해했다.**

<br>

**더불어 EAGER 방식이 권장되지 않는 이유도 분명히 알 수 있다.**

- EAGER 방식의 경우: Person List 를 조회했을 뿐인데 연관된 모든 객체(엔티티, Company)를 조회하여 예측하지 못한 쿼리가 발생한다.

- LAZY 방식의 경우: 위의 로직에서 `company.getPeople().isEmpty()` 와 같은 실질적으로 객체(엔티티)가 사용되는 로직이 없다면 쿼리가 발생하지 않는 것이다.

<br>

### Lazy vs Eager 예시

**Lazy 일 때**

```java
List<Person> people = personRepository.findAll();   // select * from person;

for (Person person : people) {
    System.out.println(person.getCompany().getName());  // select * from company where company_id = ? (N번)
}
```

**Eager 일 때**

```java
List<Person> people = personRepository.findAll();
// select * from person;
// select * from company where company_id = ? (N번)
```

<br>

## 해결 방법

다른 글들에서 소개된 해결 방법은 FetchJoin, BatchSize, EntityGraph 설정 등의 방법이 소개되는 것 같다.

> \+ 조금 이해/정리가 된 상태에서 https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1 여기의 글을 읽어보니 조금 더 확실히 이해가 되는 것 같다.

<br><br>

- https://jojoldu.tistory.com/165
- https://velog.io/@woo00oo/N-1-문제
- https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1