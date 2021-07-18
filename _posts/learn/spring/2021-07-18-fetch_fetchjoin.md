---
title: "[Spring JPA] fetch전략과 fetch join에 관한 오해"

categories:
  - TIL
tags:
  - Spring
  - Spring Boot
  - Spring JPA
toc: true
toc_sticky: true
---

# I. 개요

`[Spring JPA]  프로젝트 중 게시판 쿼리 이슈`를 해결하던 도중 굉장히 잘못된 지식을 갖고 있다는 것을 알게 되었다.  
내용을 정리하는 편이 기억에 남을 것 같아서 글로 남기려 한다.

<br><br><br>

# II. 오해

## 1. fetch전략과 fetch join은 같다?

`DB`에 대한 기본적인 지식 없이 바로 `JPA`강의를 수강하다보니 깊게 뿌리박힌 생각이었다. 단순히 이름이 같다는 이유만으로 <`fetch`전략과 `fetch join`은 같다!> 라고 생각했다.  

### fetch전략

`fetch전략`은 특정 엔티티를 조회할 때, 연관관계에 있는 다른 엔티티를 <<__언제__>> 불러올까에 대한 옵션이다.

```java
@Entity
public class Post extends BaseTimeEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "post_id")
    private Long id;

    @OneToMany(mappedBy = "post", fetch = FetchType.EAGER)
    private List<Comment> comments;
}
```

`Post`와 `Comment`가 양방향 관계를 갖고 있다고 생각하자.  

- `FetchType`이 `EAGER`일 경우, `Post`를 조회한 <<__직후__>> `comments`까지 조회하게 된다.  
- `FetchType`이 `LAZY`일 경우, `Post`를 조회하고 <<__추후__>> `Post`를 통해 `comments`를 사용하려하면 그제서야 `comments`를 조회하는 쿼리를 날린다.

### fetch join

`fetch join`은 `JPQL`만의 최적화를 위한 `join`방식이다.  
`join`의 경우 `Post`만 조회한다면 `comments`는 조회되지 않아 접근할 수 없지만, `fetch join`은 `Post`와 `comments`를 <<__동시__>>에 조회한다.

### fetch전략의 EAGER방식과 fetch join은 같은 것?

그렇지 않다. `JPQL`을 실행하여 `Post`를 조회한다고 하자.  

- `fetch전략의 EAGER방식`: `Post`를 조회한 <<__직후__>> `comments`를 조회하므로 총 `2개`의 쿼리를 날리게 된다.
- `fetch join`: `Post`와 `comments`를 <<__동시__>>에 조회하므로 총 `1개`의 쿼리를 날리게 된다.

따라서 둘은 완전히 다른 개념이며, `fetch join`은 `fetch 전략`을 무시한다.

## 2. fetch 전략을 LAZY로 설정하면 N+1문제가 일어나지 않는다?

### N+1문제란?

```java
@Entity
public class Post extends BaseTimeEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "post_id")
    private Long id;

    @OneToMany(mappedBy = "post", fetch = FetchType.EAGER)
    private List<Comment> comments;
}
```

같은 예제에서 `Post`를 10개 조회한다고 가정하자.  
그렇다면 `Post`를 조회하는 쿼리 `1개`와 모든 `Post`에 대해 `comments`를 조회하는 쿼리 `10개`를 더해 총 `11개`의 쿼리를 날리게 된다.  
이렇듯 `N = 10`개의 객체를 조회하기 위해 `1 + N(10)`개의 쿼리를 날리게 되는 문제를 `N+1문제`라고 한다.

강의를 들으면서 `N+1문제`를 방지하려면 `LAZY`방식을 사용하면 된다고 배웠기에, `LAZY`방식은 `N+1문제`와 무관하다고 생각했다.

하지만 실은 `N+1문제`가 `EAGER`방식에 비해 덜 발생하는 것이었다.

## LAZY방식일 때

마찬가지로 `Post`를 10개 조회한다고 가정하자.

만약 `Post`를 조회했지만, `comments`에 대한 내용을 호출하지 않는다면 쿼리는 `1개`만 날리는 것으로 끝날 것이다.
하지만 만약 10개의 `Post`에 대해 각각의 `comments`를 호출해야 한다고 하자. 결국 뒤늦게 `10개`의 `comments`를 호출하게 되고, 최종적으로 `11개`의 쿼리를 날리게 되어 `N+1문제`가 발생한다.

그럼에도 최선의 경우에는 `1개`의 쿼리, 최악이어도 `EAGER`방식과 같은 `11개`의 쿼리를 날리는 것이 되기 때문에 `LAZY`를 선호하는 것이라고 생각한다.