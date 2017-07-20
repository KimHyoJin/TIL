---
layout: post
title:  "토비의 스프링 1장"
date:   2017-07-13 04:00
category: Blog
tags: 		spring
---

스프링의 시작은 다음과 같다 : 어떻게 변화에 대응하는 코드를 짤 것인가?
(지금까지 개발해본 소규모 시스템에서 생각하는 것이 아닌 대규모 시스템을 생각해보아야한다. 변화에 대응할 수 있는 코드가 더욱 더 효율적인 것은 분명하다)
변화에 대응하는 코드를 위해서는 관심사가 분리되어야한다.

### 관심사의 분리
관심사의 분리란 관심(기능)별로 응축하고 이를 느슨하게 잇는 것이다. 즉, 관련된것끼리는 응집도를 높이고, 결합도는 낮추는 것이다. 책에서는 다음의 예시를 든다. 예를 들어 DB를 연결하는 connection 코드를 짤 때, Aconnection과 Bconnection, Cconnection이 있고 이를 상황에 따라 바꾸고자 한다면, connection에 해당하는 부분만 분리하여 변경하는 것이 편할 것이다. 이것이 관심사의 분리다

그렇다면 어떻게 분리할 수 있을 까?

### 상속
connection을 사용하는
``` java
public class UseConnection{
  public void use(){
    Aconnection aconnection = new Aconnection();
    aconnection.getConnection();
  }
}
```
이 존재한다고 하자. 이 때 위에서 말한것처럼 관심사의 분리를 하려면 어떻게 해야할까? 가장 첫번째로 상속을 생각할 수 있다. 상속을 통해 Aconnection도, Bconnection도 동일 메소드를 사용하게 하는 것이다.

``` java
public abstract Connection{
  public abstract int getConnection();
}

public class Aconnection extends Connection{
  public int getConnection(){

  }
}

public class Bconnection extends Connection{
  public int getConnection(){

  }
}
```
그러나 문제점이 있다. 우선 상속은 다중상속이 안된다. 이는 큰 제약이다


### 아예 클래스를 새로 만들자
``` java
public Class Aconnection{
  public int getConnection(){
    ...
  }
}

public class UseConnection{
  public void use(){
    Aconnection aconnection = new Aconnection();
    aconnection.getConnection();
  }
}
```
그러나 Bconnection을 쓰려면 UseConnection의 해당 부분을 수정해야한다. UseConnection이 Bconenction을 직접 사용한다.

### 인터페이스
인터페이스를 쓰면 이렇게 할 수 있다.
``` java
public interface Connection{
  public int getConnection();
}

public Class Aconnection implements Connection{
  public int getConnection(){
    ...
  }
}

public class UseConnection{
  public void use(){
    Aconnection aconnection = new Aconnection();
    aconnection.getConnection();
  }
}
```
그래도 여전히 Aconnection을 사용한다. 그 이유는 어떤 connection을 사용할지를 UseConnection에서 결정하기 때문이다(=분리가 제대로 안됨)
