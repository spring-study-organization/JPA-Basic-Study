# JPA가 나오게 된 배경

지금 대다수의 애플리케이션은 데이터를 관계형 DB에 저장한다.

따라서 필연적으로 데이터를 저장하고 불러오는데 있어 `SQL`이 많이 사용될 수 밖에 없는데

여기서 문제가 발생하게 된다.

## 객체와 관계형 데이터베이스의 불일치 

객체와 관계형 데이터베이스의 불일치로 인해 다음 4가지 문제점이 생긴다.

1. 상속
2. 연관관계
3. 데이터 타입
4. 데이터 식별 방법


### 1. 상속  

<img src="../src/1주차/data1.png">

가령 위와 같은 상속을 받는 객체 구조가 있다고 하면

`Album` 객체를 디비에 저장한다고 하자. 그러면,

테이블은 상속이라는 개념이 없으므로

`Album` 테이블 안에 있는 맴버 중에 `Item` 테이블에 있는 항목과 `Album` 

테이블에 있는 항목을 나눠서 각각 `SQL`을 작성해서 넣어주는 과정이 필요하다.

디비에서 꺼내거나 수정해야 할 때 역시 테이블 마다 항목을 나눠서 `SQL` 을  따로 작성해야 하는 문제점이 생긴다.

### 2. 연관 관계

<img src="../src/1주차/data2.png">

`Member` 와 `Team` 객체를 위와 같이 설계 했다고 하면 

객체와 테이블 사이에 불일치가 다시 발생하는데 바로

>객체는 `Member` 에서 `Team`에 접근할때는 참조를 이용하고 , `Member` -> `Team` 단방향 접근만 가능하다.

반면에

>테이블은 `Member` 에서 `Team`에 접근할때 `PK`, `FK` 를  이용하고,  `Member` <-> `Team` 양방향 접근이 가능하다.

#### 객체를 테이블에 맞추기

이 불일치를 해결하기 위해 객체를 테이블 구조에 맞춰서 설계했다고 하면 다음과 같은 형태가 될 것이다.

```java
    class Member{
        String id;          //Member_ID 컬럼 사용
        Long teamId;        //TEAM_ID FK 사용
        String username;    //USERNAME 컬럼 사용
    }

    class Team{
        Long Id;        //TEAM_ID PK 사용
        String name;    //NAME 컬럼 사용
    }
```

위와 같이 설계를 하게 되면 `Member` 에서 `Team` 으로 직접 참조를 하지 못하고  
`Member` 에서 `Team` 으로 접근할 때 `Member` 의 `teamId` 값을 가지고 
```sql
    SELECT * 
    FROM TEAM 
    WHERE ID =  #{TeamId}
```
 이런 식의 SQL을 보내줘야 할 것이다.

하지만 이는 객체 지향의 장점을 포기해야 하므로 그다지 좋은 방법이 아니다.

#### 객체 모델링 대로 테이블에 저장

그러면 이제 남은 방법은 객체 그대로 내비두고 개발자인 우리가 테이블에 맞게 매핑해주는 수 밖에 없다.

```java
    class Member{
        String id;          //Member_ID 컬럼 사용
        Team team;        //참조로 연관관게를 맺는다.
        String username;    //USERNAME 컬럼 사용

        Team getTeam() {
            return team;
        }
    }

    class Team{
        Long Id;        //TEAM_ID PK 사용
        String name;    //NAME 컬럼 사용
    }
```

>즉, 위와 같이 설계하고 `CRUD` 를 할때 따로 매핑해주는 작업이 필요하다 => 복잡하고 귀찮다.

### 3. 데이터 타입


