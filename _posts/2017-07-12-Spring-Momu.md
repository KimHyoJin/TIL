---
layout: post
title:  "모모&무무와 함께하는 스프링"
date:   2017-07-13 04:00
category: Blog
tags: 		spring
published: false
---

##### 인터페이스와 상속
어떤 프로그램을 개발 했을 때, 변화에 대응할 수 있게 개발을 하려면 중복코드를 줄여야한다. 예를 들어 add()라는 메소드를 sub()로 수정하려할 때 add를 호출하는 부분을 전부 sub()로 수정해주어야한다. 최소한의 변경이 필요하다.

따라서 분리와 확장이 필요하다. 분리는 공통된 주제(관심사)들끼리 묶는 것이다. 그리고 스프링의 기본 철학은 이를 용이하게하는것이다. 그러기 위해서는 주제간의 dynamic하고 유연한 관계가 설정되어야한다.

그럼 예를 들어 고양이를 키운다고 할 때, 모모를 키우는 것과 무무를 키우는 것 두가지가 있지만, 둘에게 하는 행동의 포맷은 비슷하다. 대신 무무는 장이 안좋아서 밥에 유산균을 타먹여야한다. 모모는 털이 많이 날려서 빗질을 무무의 100배 해줘야한다. 샤워도 모모가 무무보다 자주해야한다. 대신 무무는 엉덩이를 매일 확인해줘야한다. 이렇게 세부적인 내용은 다르지만, 밥을 먹이고 씻기고 빗질해주는 것은 같다. 내가 모모를 키우다가 무무를 키우려면 어떻게 해야할까?

분리와 확장을 고려한다면, 첫번째 방법으로 상속이 있을 것이다. 구현해야할 밥먹이기, 빗질, 샤워 메소드를 정의해두고 이를 상속받아서 모모클래스, 무무클래스에서 구현하면 된다. 좋은 방법인것 같다! 대신 문제가 있다. 그럼 내가 모모를 생성할 때 어떻게 해야할까?
```java
Momo momo = new Momo();
```
이런식으로 모모를 불러와야할 것이다. 이미 내가 new Momo();를 사용한 시점에서 모모를 키우는 것을 확정했으므로, 무무를 키울려면 이를 바꾸어야한다. 또한 다중상속의 문제가 있다. 고양이 분양받는 것/고양이를 기르는 것/고양이가 죽는 것이 각각 다른 abstract로 구현되어있다면 이 3개를 동시에 상속받을 수 없다(예제 좀 억지인듯..) 고양이를 분양받고 그 고양이를 기를 수 없다!
두번째 방법은 인터페이스가 있을 것이다. 고양이 인터페이스를 만들어두면 다중상속은 해결할 수 있다. 그러나 앞서 말한 선언의 문제가 또 발생한다.

##### 그래서 DI
그래서 의존성 주입이 나왔다. 내가 키우는 고양이를 주는 것이다! 다음과 같이 구현하는 것이다. (편의상 하나의 파일에 다 쓴다..)
```java
// cat interface
public interface CatRaise{
  public void feed();
  public void brushing();
  public void shower();
}

public class Momo implements CatRaise{
  public void feed(){
    //eat dryfood;
  }
  ...(생략)
}

public class Mumu implements CatRaise{
  public void feed(){
    //eat rawfood + lactobacillus;
  }
  ...(생략)
}

public class Hyojin{
  // hyojin's cat
  private CatRaise catraise;

  public Hyojin(CatRaise catraise){
    this.catraise = catraise;
  }
}

```
이게 생성자를 사용한 방법이다. 또 setter를 사용할수도 있다.
```java
public class Hyojin{
  // hyojin's cat
  private CatRaise catraise;

  public setCatRaise(CatRaise catraise){
    this.catraise = catraise;
  }
}
```
그럼 여기에 어떻게 생성한 bean을 넣어줄 수 있을까?
첫번째로 xml을 사용할 수 있다.
```xml
<bean id="hyojincat" class="com.example.Mumu">
```
이렇게 등록 후
```java
CatRaise catraise = applicationContext.getBean("hyojincat");
```
으로 가져올 수 있다.

혹은 @을 사용할수도 있다!(Java Config)
```java

@Bean
public class Mumu implements CatRaise{
  public void feed(){
    //eat rawfood + lactobacillus;
  }
  ...(생략)
}

public class Hyojin{
  // hyojin's cat
  @Autowired
  private CatRaise catraise;
}
```
xml로만 하는 방법도 있는데 추가해서 넣을 것.
