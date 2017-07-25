---
layout: trouble
title:  "IntelliJ에서 jar파일 만들고 실행시키기"
date:   2017-07-12 04:00
category: TIL
tags: 		spring intellij
---
jar파일 만들려면 terminal에서 command를 쳐서 만들거나 ide를 활용해서 만드는 두가지 방법이 있다.    
우선 IntelliJ에서 하는 방법은 버그가 있다고 한다..    
첫번째 문제는 META-INF 폴더가 java아래 생기는 것이다. 그래서 artifact를 추가해줄 때 manifest 위치를 따로 설정해줘야한다. 두번째 문제는 [**No auto configuration classes found in META-INF/spring.factories**](https://stackoverflow.com/questions/38792031/springboot-making-jar-files-no-auto-configuration-classes-found-in-meta-inf) 문제인데 이거 때문에 삽질 엄청함. 보면 해결법이 나와있긴 한대 나는 해결이 안되었다.. 왜일까..? 따라서 생성자체를 command를 활용해서 했음..  
maven도 따로 설치해야하는 데 [설치방법](http://javakorean.com/맥-메이븐-설치하기)을 따라서 설치하면 된다. source를 해줘야 profile이 적용됨.
```shell
$java -jar [jar file name]
```
하면 실행된다.
