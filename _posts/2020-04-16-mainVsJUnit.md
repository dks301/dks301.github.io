---
title:  "main method vs JUnit"
excerpt: "우아한 테크코스 - 1주차"
permalink: /programming/
categories:
    - Programming

tags:
    - 우아한 테크코스
    - JUnit5
    - Test
    - assertj
    - main method

last-modified_at: 2020-04-16T17:00:00-05:00
toc: true
toc_sticky: true
---

## main method vs JUnit
레벨 1 첫번째 미션에서 배운 `main method`를 테스트 용도로 사용하는 것의 문제점과 해결 방법인 `JUnit` 정리입니다.

### 1. main method 테스트의 문제점  
main method 테스트(E.g.`System.out.println("foo")`) 는 JUnit을 배우기 전 항상 테스트를 위해 하던 것인데요.  
여러 문제점이 있습니다.  
> * Test 코드가 실 서비스에 같이 배포된다.
> * main method가 한 번에 여러개의 기능을 테스트한다.
> * 어떤 기능 테스트인지 의도를 드러내기 힘들다.
> * 테스트 결과를 사람이 수동으로 확인해야 한다.

1-1. Test 코드가 실 서비스에 같이 배포된다.
실제로 내가 진행했던 프로젝트에는 이런 실수가 없나 찾아보았습니다.
```java
@Override
public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
    for (DataSnapshot snapshot: dataSnapshot.getChildren()) {
        Log.e("제발000", tempUser.getEmail() + snapshot.child(tempUser.getUsername()).getValue()
        + tempUser.getBetting() + tempUser.getUsername());
    }
}
``` 
~~그 때의 간절함이 다시 느껴지네요.~~  
이렇게 실 서비스에 그대로 배포하는 실수를 할 수 있습니다.  
  
1-2. main method가 한 번에 여러개의 기능을 테스트한다.
다시 위의 코드를 자세히 살펴볼까요.
```java
@Override
public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
    for (DataSnapshot snapshot: dataSnapshot.getChildren()) {
        Log.e("제발000", tempUser.getEmail() + snapshot.child(tempUser.getUsername()).getValue()
        + tempUser.getBetting() + tempUser.getUsername());
    }
}
```
제발000이 눈에 띄는데요.  
아마 여러 기능을 동시에 테스트하기 위해서 더 찾아보면 제발001, 제발002, 제발003 등이 있겠죠?  
  
1-3. 어떤 기능 테스트인지 의도를 드러내기 힘들다.
역시 위의 코드를 다시 소환 해보겠습니다.
```java
@Override
public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
    for (DataSnapshot snapshot: dataSnapshot.getChildren()) {
        Log.e("제발000", tempUser.getEmail() + snapshot.child(tempUser.getUsername()).getValue()
        + tempUser.getBetting() + tempUser.getUsername());
    }
}
```
제가 짠 코드인데도 잘 모르겠습니다.  
dataSnapshot이 제대로 저장되는지 확인하려는 것 같기는 한데...  
무언가 테스트를 해서 돌아가는 코드를 만드려는 간절함만 느껴집니다.
거기에 수많은 get의 향연...  
이렇게 제가 짠 코드임에도 어떤 의도를 가진 테스트인지 모르는 문제점이 있습니다.  
  
1-4. 테스트 결과를 사람이 직접 확인해야 한다.
이쯤이면 이 포스팅을 하기 위해서 일부로 저렇게 배포를 했나 싶을 정도입니다.  
다시 위의 코드를 한번 보겠습니다.  
```java
@Override
public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
    for (DataSnapshot snapshot: dataSnapshot.getChildren()) {
        Log.e("제발000", tempUser.getEmail() + snapshot.child(tempUser.getUsername()).getValue()
        + tempUser.getBetting() + tempUser.getUsername());
    }
}
```
이렇게 제발000이 출력되는 위치에서 어떠한 기능이 제대로 동작하는지 확인해야합니다.  
따라서, tempUser의 이메일, 유저네임 등을 눈으로 보고 내가 원하던 기능이 맞는지 직접 판단을 해야하는 문제가 있습니다.  

### JUnit을 이용한 문제점 해결
이렇게 main method의 문제점을 해결하기 위해서 등장한 도구가 바로 `JUnit`입니다.
특히, JUnit5와 asssertj를 활용한 메소드 체이닝으로 좀 더 깔끔하고 읽기 좋은 assert코드로 테스트가 가능합니다.  
이제 위 예제를 JUnit기반 테스트로 바꾸어 보겠습니다.  
정확히 기억은 나지 않지만 아마  1.DB데이터 변화가 생기면, 2.dataSnapshot을 통해, 3.해당 메서드가 동작  
위와 같이 가정하고 테스트 코드를 간단하게 작성해보겠습니다.
```java
@DisplayName("DB에 유저를 추가했을 때 dataSnapshot이 정상적으로 변환되는지 확인")
@Test
public void onDataChangeTest() {
    DataSnapshot beforeDataSnapshot = Database.getDataSnapshot(); // 기존 dataSnapshot 로드라고 가정

    Database.addUser(new User());// db에 유저가 추가되었다고 가정
    DataSanpshot afterDataSnapshot = Database.getDataSnapshot();

    assertThat(beforeDataSnapshot.size() + 1).isEqualTo(afterDataSnapshot.size());
}
```
  
위와 같이 dataSnapshot에 정상적으로 유저가 추가되었는지 크기 비교를 통해 간단하게 확인할 수 있습니다.  
조금 더 정확하게 내가 추가한 유저가 맞는지 확인하고 싶다면 아래와 같이 테스트를 작성할 수도 있습니다.
```java
@DisplayName("DB에 유저를 추가했을 때 dataSnapshot에 해당 유저가 추가되었는지 확인")
@Test
public void onDataChangeTest() {
    User user = new User();

    Database.addUser(user);// db에 유저가 추가되었다고 가정
    DataSanpshot dataSnapshot = Database.getDataSnapshot();

    assertThat(dataSnapshot.getLastUser()).isEqualTo(user);
}
```

이제 두 가지 테스트를 보면서 main method test의 문제점이 해결되었는지 알아보겠습니다.
 * ~~Test 코드가 실 서비스에 같이 배포된다.~~(해결)
    * 위 테스트는 프로덕션 코드 위치인 src 디렉토리가 아니라 test 디렉토리에 위치하므로 따로 배포할 수 있습니다.
 * ~~main method가 한 번에 여러개의 기능을 테스트한다.~~(해결)
    * 정확히 하나의 테스트 메서드가 하나의 기능을 테스트합니다. 
 * ~~어떤 기능 테스트인지 의도를 드러내기 힘들다.~~(해결)
    * DisplayName을 보면 정확히 어떤 의도를 가진 테스트인지 알 수 있습니다.
 * ~~테스트 결과를 사람이 수동으로 확인해야 한다.~~(해결)
    * 테스트 실행하면 메서드 마다 pass/fail 결과가 바로 나옵니다.

### 결론
main method는 프로그램 시작 용도로만 사용하고 테스트는 JUnit을 사용하자.