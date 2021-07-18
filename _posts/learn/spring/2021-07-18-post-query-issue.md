---
title: "[Spring JPA] 프로젝트 중 게시판 쿼리 이슈"

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

![01](https://user-images.githubusercontent.com/68958979/126056497-e0709e12-622d-49ed-90b8-9d15847ecabd.png)

현재 구현되어있는 게시판의 모습이다.  
이 화면에서는 크게 세가지 기능이 있다.

1. `POST`에 대한 간략한 정보를 제공한다. 게시글 제목이나 번호, 추천수 등
2. 글을 작성한 `USER`의 닉네임을 표기하고 해당 `USER`의 링크를 제공한다.
3. 해당 `POST`에 달린 `COMMENT`의 갯수를 보여준다.

헌데 개발 도중 이상한 점을 발견했다.

![02](https://user-images.githubusercontent.com/68958979/126056498-c7409e45-ec3b-4348-94df-a932c1a9efc6.png)

게시판을 한번 조회하는데 저많은 쿼리가 날아가는 것이다.  
물론 아직 DB라던가 이해가 많이 부족하지만, 명백하게 비효율적인 부분이라고 생각했다.  
이 부분에 대해 해결해보고자 한다.

<br><br><br>

# II. 원인


```
(기존 작성하던 잘못된 생각)
원인은 명백하게도 `POST`의 필드에서 `USER`와 `COMMENTS`의 `fetch join`옵션이 지연로딩이었기 때문이다.  

즉시로딩의 경우 엔티티를 호출할 때 연관관계를 맺고 있는 다른 엔티티들까지 일괄 호출하여 가져오게 되고, 
지연로딩의 경우 엔티티를 호출해도 연관관계를 맺고 있는 다른 엔티티를 사용하려하기 전까지는 호출하지 않는다.

`OneToMany`등으로 연관관계를 맺고 있는 엔티티를 즉시로딩으로 호출할 경우 컬렉션의 수만큼 추가로 쿼리가 날라갈 수 있기 때문에 N+1문제를 야기할 수 있어 일반적으로 지연로딩을 사용한다.

이번 경우에는 되려 지연로딩으로 설정되어있기 때문에 발생한 문제였지만, 그렇다고 이를 위해 지연로딩을 즉시로딩으로 변경하면 추후 다른 곳에서 `POST`엔티티를 가져올 때 문제가 될 수 있을 것이다.
```

위 내용에 대해 깊이 고민해보다보니, 내가 굉장히 크게 잘못된 지식을 갖고 있다는 것을 알게 됐다. 이에 대해서는 다른 포스트로 별도로 작성할 예정이다.  

결국 원인은 `join`방식이 `fetch join`방식이 아니기때문에 일어난 `N+1`문제이다.

<br><br><br>

# III. 해결과정

## 1. @LazyCollection

```java
@LazyCollection(LazyCollectionOption.EXTRA)
@OneToMany(mappedBy = "post")
private List<Comment> comments;
```

검색을 통해 `@LazyCollection` 어노테이션에 대해 알게되어, 직접 적용해봤다.  
`Collection`에 대해 세가지 옵션을 줄 수 있다.

1. `LazyCollectionOption.TRUE`: `fetch 전략`의 지연로딩과 마찬가지로, 직접 접근해야 할 때 `query`를 날린다.
2. `LazyCollectionOption.FALSE`: `fetch 전략`의 즉시로딩과 마찬가지로, 사용여부에 관계없이 바로 `query`를 날린다.
3. `LazyCollectionOption.EXTRA`: `.size()` 또는 `.contains()`로 접근할 때, 컬렉션 전체를 초기화하지 않고 그 값만 가져온다.

3번의 `EXTRA`옵션이 사용할 수 있을 것 같다고 판단해 그대로 적용해봤다.

![03](https://user-images.githubusercontent.com/68958979/126056500-a2ebcf64-d60d-4c8f-8888-fac051a680a2.png)

하지만, 결과는 쿼리의 길이가 줄어들었을 뿐 쿼리의 횟수 자체에는 변화가 없었다.

```java
public static PostListResponse of(Post post) {
    return PostListResponse.builder()
            .id(post.getId())
            .title(post.getTitle())
            .userId(post.getUser().getId())
            .userName(post.getUser().getNickname())
            .created(post.getCreatedDate())
            .commentCount(post.getComments().size())
            .viewCount(post.getViewCount())
            .reLike(post.getReLike())
            .build();
}
```

그 이유를 생각해보면, 게시판 리스트를 보여주는 `PostListResponse`를 구성할 때, 페이징된 모든 `post` 각각에 대해 `post.getComments().size()`를 사용하기 때문에 결과적으로 `post`의 수만큼 쿼리를 날려야하는 것은 변함이 없는 것이다.

## 2. @EntityGraph

이번엔 `Spring Data JPA`쪽에서 해결방법을 찾을 수 있지 않을까 생각했다.

검색 중 `@EntityGraph`라는 어노테이션을 찾게 됐다.

```java
@EntityGraph(attributePaths = {"user", "comments"}, type = EntityGraph.EntityGraphType.LOAD)
Page<Post> findAll(@Nullable Specification<Post> spec, Pageable pageable);
```

`@EntityGraph`는 적용한 메소드에 `attributesPaths`로 지정한 연관관계 엔티티에 대해서는 한번에 조회할 수 있게 된다. (`fetch join`처럼)


![04](https://user-images.githubusercontent.com/68958979/126057071-4f2240a8-73ca-4685-90bb-243e5e643f79.png)

결과적으로 쿼리가 단 한건으로 줄어들었다!  
그런데 딱봐도 좋지 않은 경고메세지가 보인다.

```
HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
```

검색을 통해 확인해보니, 페이징 API와 (컬렉션과의)`fetch join`을 함께 사용할 경우 `limit`쿼리를 없애버리고 모든 내용을 메모리로 가져와서 올린 후에 그제서야 페이징을 시행한다고 한다.  
즉, 모든 `POST`를 싹다 가져와서 해당페이지에 맞는 30개만을 잘라 보여주는 것이다.

굉장히 비효율적이거니와 `POST`의 수가 늘어난다면 메모리에 큰 부담이 될 것이다.

## 3. BatchSize 

`BatchSize`를 이용하기로 했다.

```java
@BatchSize(size = 100)
@OneToMany(mappedBy = "post", fetch = FetchType.EAGER)
private List<Comment> comments;
```

`@BatchSize`를 설정하면, 해당 요소를 읽어올 때 정해진 수만큼 합산하여 한번에 쿼리를 날리게 된다. 원래 `@OneToMany`의 `fetch`전략 기본값은 지연로딩이지만 `@BatchSize`사용을 위해 즉시로딩으로 바꿔줬다. 대부분의 `POST`를 불러오는 부분에 있어서 `comments`를 필요로 하기 때문에 성능상의 문제는 없을 것이라고 생각한다.

```java
root.fetch("user", JoinType.LEFT);
```

`User`의 경우에는 `Specification`을 정의하는 부분에서 `fetch join`할 수 있도록 했다. 컬렉션이 아닌 경우는 `fetch join`을 해도 페이징에 있어서 문제가 생기지 않기 때문이다.

![05](https://user-images.githubusercontent.com/68958979/126060653-1d6e0c48-d189-446c-9cb9-1d929c6ae6c5.png)

그 결과 `POST`와 `USER`는 하나의 쿼리로 합쳐지고, `COMMENT`의 경우에도 쿼리가 현저히 줄어들었다. 코멘트 쿼리가 두개로 나가는 것은 하이버네이트에 의해 최적화 된 것이라고 한다.

<br><br><br>

# IV. 마치며

당장 마주친 문제를 해결하고자 굉장히 많이 검색하고 공부했다.  
그런데, 기존에 알고있던 지식중에 크게 잘못 알고 있던 지식들이 있어 특히 혼란이 왔다. 검색하며 여러 포스트를 읽다 보니 나만 잘못 생각하고 있는 것이 아닌가보다.  
이 부분에 대해서 정리하면 JPA에 대해 더 잘 알게 될 것 같다.

처음으로 단순 기능이 아니라 성능, 효율성에 대해 깊게 생각해볼 수 있던 시간이었다.