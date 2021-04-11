---
title: "자바 ORM 표준 JPA 프로그래밍 기본편 - 2. JPA 시작하기"

categories:
  - Study
tags:
  - JAVA
  - JPA
toc: true
toc_sticky: true
---

[김영한님의 인프런 강의 - 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://inf.run/VP3b)  
위 강의를 정리한 내용입니다.

- `EntityManagerFactory` : 하나만 생성해서 애플리케이션 전체에서 공유
- `EntityManager` : 쓰레드간에 공유 X, 사용하고 버려야 함
- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

## EntityManagerFactory

### 선언
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
```

## EntityManager

### 선언
```java
EntityManager em = emf.createEntityManager();
```

### Create
```java
Member member = new Member();
member.setId(1L);
member.setName("HelloA");
em.persist(emeber);
```

### Read
```java
Member findMember = em.find(Member.class, 1L);
```

### Update
```java
Member findMember = em.find(Member.class, 1L);
findMember.setName("HelloJPA");
```
- 다시 `persist`하지 않아도 된다.

### Delete
```java
Member findMember = em.find(Member.class, 1L);
em.remove(findMember);
```

### 그 외의 것들
```java
em.createQuery
```
- `JPQL`문을 사용하여 얻어내자.