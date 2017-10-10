---
layout: post
title: Spring Bean Scope
---

# Spring Bean Scope
spring bean은 보통 singleton으로 동작한다고 생각할 수 있는 데, 사실 scope를 줄 수 있다.

### 시작하기에 앞서
요즘 나는.. 방명록 API들을 구현하고 있다. 글을 수정/삭제 등등의 작업시, 요청한 사람이 실제 존재하는 user인지를 확인하기 위해 Interceptor에서 해당 요청자의 userID로 데이터베이스에서 userNo를 구한다. userNo가 존재하면 존재하는 user인 것이다. 그리고 controller로 넘어가는데 ...
그런데 이 userNo가  repository나 service에서도 사용이 되면 또 데이터베이스에 접속해야한다. 굳이 이런 일들을 하고 싶지않아서 해당 userNo를 저장하기로 하였다.

#### 처음 시작은 thread-local
Tomcat의 경우 request당 thread가 하나씩 생기므로 thread-local을 사용해서 데이터값을 저장해도 좋다!

#### 그런데 ...
애석하게도 thread-local은 인턴 과제 때 써본적이 있다. 뭔가 새로운 것을 이용해서 구현을 하고 싶었다. 전에 책에서 읽었을 때, spring에서 scope를 통해 singleTon이 아닌 객체를 생성할 수도 있다고 하였다. 그래서 scope를 써보자! 가 된것이다.

즉 이글은 내가 scope를 쓰면서 삽질한 이야기를 담았다.

### Scope

#### SingleTon
그냥 default 값은 singleton이다. singleTon은 아는 것처럼 한번 생성하고 이걸 계속 쓰는 것이다. thread가 함께 공유한다. 그래서 thread마다 가지고 있어야하는 값은 thread-local을 써야하거나, 뭐 thread id를 이용해서 저장하거나 그런식으로 구분해서 해야한다.

#### Prototype
prototype은 object에 접근할 때마다 새로운 인스턴스를 생성하는 것이다.

이 아래부터는 spring MVC에서만 사용이 가능하다.   

#### Request
request는 http request별로 인스턴스를 생성한다. 내가 요번에 쓴 것이다.

#### Session
session은 http session 별로 인스턴스를 생성한다.


이제 설계에 맞게 scope를 설정해줄 것이다. 아래처럼
~~~java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST)
public class RequestUser {
    //private ThreadLocal<Integer> xUserNo;
    private Integer xUserNo;
}
~~~

### 그런데 문제가 있다!
그런데 이렇게 하면 에러가 발생한다. 왜일까?
나는 이걸 이런식으로 주입하여 사용하였다.

~~~java
@RestController
@RequestMapping("/api/v1/user/")
public class UserAccountController {

    @Autowired
    private UserAccountService userAccountService;

    @Autowired
    private RequestUser requestUser;

    @RequestMapping(value = "create.json", method = RequestMethod.POST)
    public ResultContainer addUserAccount(@RequestBody UserAccountParam userAccountParam) {

        checkParam(userAccountParam.getUserId());
        checkParam(userAccountParam.getUserName());

        userAccountService.addUserAccount(userAccountParam.getUserId(), userAccountParam.getUserName());
        return new ResultContainer();
  }
}
~~~

controller도 component이다. 그것도 singleTon component이다. 생각해보면, singleTon은 처음 한번 만들어지는 데, 이 때 requestUser를 주입해주고 그다음에는 주입하지 않는다. 즉 scope설정이 무의미해진다.
한번 돌려보면 알수 있듯, 에러가 뜬다. 문구는 못긁어놨었는데, 해석해보자면
"주입받는 객체도 같은 scope를 가지고 있어야한다" 였다.
그럼 이를 어째. 이거 쓰자고 controller scope를 변경할까?

### Proxy Pattern
인턴 때 책을 살수있는 기회가 있었는데, pattern 책을 주문해달라고 부탁드렸었다. 인턴 때, 수학으로 비유하자면 정석인 풀이방법이 있는데 자꾸 난잡하게 푸는 기분, 즉 코드짜는 정석인 방법이 있는 데 자꾸 돌아돌아 짜는 기분이 들었었다. 그래서 책을 선물 받았었는데 아직 안펴봤다... 헤헤
그 후회를 지금한다.
Proxy Pattern이라는 게 있다. 네트워크 proxy처럼, proxy를 두고 해당하는 인스턴스로 연결해주는 것이다!
(사실 아직 100% 아는 건 아니고, 40% 정도 이해한 것 같다. 찾아보니 AOP이야기도 나오는 데 왜 나오는 지 모르겠다. 집가서 책 읽고 이해안되면 물어봐야지)
그러면 singleTon이 생성될 때 이것은 proxy bean을 주입받고, 나중에 dynamic하게 연결해줄 수있는 것같다(정확한지 모름).
따라서 다음과 같이 해주면 에러가 안난다.

~~~java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestUser {
    //private ThreadLocal<Integer> xUserNo;
    private Integer xUserNo;
}
~~~

#### ProxyMode?
그런데 한가지 고민되는 게 있었다. ProxyMode로 다음의 것들을 설정할 수 있다.
~~~java
public enum ScopedProxyMode {

	/**
	 * Default typically equals {@link #NO}, unless a different default
	 * has been configured at the component-scan instruction level.
	 */
	DEFAULT,

	/**
	 * Do not create a scoped proxy.
	 * <p>This proxy-mode is not typically useful when used with a
	 * non-singleton scoped instance, which should favor the use of the
	 * {@link #INTERFACES} or {@link #TARGET_CLASS} proxy-modes instead if it
	 * is to be used as a dependency.
	 */
	NO,

	/**
	 * Create a JDK dynamic proxy implementing <i>all</i> interfaces exposed by
	 * the class of the target object.
	 */
	INTERFACES,

	/**
	 * Create a class-based proxy (uses CGLIB).
	 */
	TARGET_CLASS;

}
~~~

(그냥 긁어옴...)
INTERFACES와 TARGET_CLASS는 무슨 차이일까? 아무리 생각해봐도 모르겠는 것이다. 읽어봐도 차이를 잘 모르겠었다.. 나의 경우는 둘다 써도 되는 거 아닐까?
여쭈어보니
 https://stackoverflow.com/questions/10664182/what-is-the-difference-between-jdk-dynamic-proxy-and-cglib
이런 좋은 답변을 알려주셨다!
그런데 아직 정확이 이해한건 아니라서 나중에 관련 글을 쓰던가 수정해야겠다.
맨날 내 글은 나중에 수정해야겠다로 끝나는 것같다.. 더 써야대는데 배고파서 집에 가야될것같다.. 집에가서 시간되면 써야지.. 그런데 내 맥북은 너무 구져서 느리구.. 회사컴을 들고가자니 무겁다...
