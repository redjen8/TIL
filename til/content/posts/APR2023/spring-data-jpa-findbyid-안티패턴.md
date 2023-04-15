---
title: "Spring Data JPA findById 안티패턴"
date: 2023-04-16T20:11:58+09:00
draft: false
author: redjen
---

Spring Data JPA는 가장 편리하게 자바 객체와 데이터베이스 간 ORM을 구성할 수 있게 해주는 편리한 방법 중 하나이다.

그런데, Spring Data JPA의 `findById`를 사용하는 것은 라지 스케일 어플리케이션에서 성능 이슈를 일으킬 수 있다.

https://medium.com/javarevisited/spring-data-jpa-findbyid-anti-pattern-aefbf045fc44

## 1. 불필요한 데이터베이스 쿼리

데이터베이스에서 여러 개의 엔티티를 PK 값으로 가져와야 하는 상황을 가정해보자.

이 때 각 엔티티에 대해 `findById`를 사용한다면 하나의 엔티티를 가져오기 위해 하나의 쿼리를 사용하게 된다.
그리고 이는 성능에 있어서 빠른 병목 지점이 될수 있다.

### 해결 방법

`findAllById` 메서드를 대신 사용하는 방법이 있다. 이 메서드는 하나의 데이터 베이스에 대해 PK의 컬렉션을 전달함으로써 다수의 엔티티를 가져올 수 있게 한다.

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public List<User> getUsersByIds(List<Long> ids) {
        return userRepository.findAllById(ids);
    }
}
```

상기 코드는 아래와 같은 쿼리를 수행한다.

```sql
SELECT * FROM User user WHERE user.id IN :ids
```

`findById`를 사용하면 단일 SQL 쿼리를 사용해서 다수의 사용자를 가져오게 된다. 이는 데이터베이스로의 접근 횟수를 줄이고, 따라서 어플리케이션의 성능을 향상시킨다.

## 2. 불필요한 객체 생성

`findById`를 수행할 때 마다 Spring Data JPA는 이미 해당 엔티티가 퍼시스턴스 컨텍스트에 존재해도 새로운 엔티티 객체를 생성한다. 이는 메모리 사용을 꽤 증가시키고, GC에 따른 오버헤드를 증가시킨다.

```java
@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
    private List<Employee> employees = new ArrayList<>();

    // getters and setters
}

@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;

    // getters and setters
}
```

위의 예시에서는 엔티티 간 일대다 관계를 설정해놓았다. 엔티티를 id로 조회한다면 `Department` 객체와 관련된 `Employee` 객체를 fetch 하기 위해 하이버네이트는 SQL 쿼리를 생성할 것이다.

```sql
SELECT * FROM departments WHERE id = ?

SELECT * FROM employees WHERE department_id = ?
```

### 해결 방법

대신 `getOne` 메서드를 사용해야 한다. `getOne` 메서드는 실제로 데이터베이스로부터 가져오는 것이 아니라 엔티티에 대한 레퍼런스를 리턴한다.

이 동작은 엔티티의 PK나 엔티티 속성의 하위 집합에만 접근이 필요할 때 유용하다.

```java
@Service
public class DepartmentService {

    @Autowired
    private DepartmentRepository departmentRepository;

    public Department getDepartmentReferenceById(Long id) {
        return departmentRepository.getOne(id);
    }
}
```

`id` 값으로 상기 메서드를 호출할 때, 하이버네이트는 `Department` 엔티티를 표현하는 프록시 객체를 fetch하기 위한 HQL 쿼리를 생성할 것이다.

```sql
SELECT department FROM Department department WHERE department.id = ?
```

관계된 `Employee` 객체를 fetch하기 위한 실제 SQL 쿼리는 `Department` 객체의 `employees` 속성에 접근하기 전까지 실행되지 않는다.

JPA를 사용할 때에는 lazy하게 쿼리를 수행하는 전략을 적절히 사용해야 성능을 향상시킬 수 있을 것 같다.

