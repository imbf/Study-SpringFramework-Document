## 1.5 Bean Scopes

Bean을 정의를 생성할 때, 개발자는 Bean 정의에서 정의되어진 클래스의 실제 인스턴스의 생성을 위해 recipe를 생성한다. Bean 정의가 하나의 레시피라는 비유는 매우 중요한 아이디어이다. 왜냐하면 이러한 아이디어는 클래스와 같이(as with) 하나의 레시피로부터 많은 객체 인스턴스를 생성할 수 있기 때문입니다.

개발자는 특정한 Bean 정의로부터 생성되어진 객체와 연결된 설정 값과 다양한 의존성을 제어할 수 있고, 뿐만 아니라 특정한 Bean 정의로부터 생성되어진 객체의 범위(scope)를 제어할 수 있다. Java 클래스 레벨에서 객체의 범위를 굽지 않고 설정을 통하여 생성하려는 객체의 범위를 선택할 수 있습니다. **Bean들은 다양한 범위중의 하나로 배포되어지도록 정의되어진다.** SpringFramework는 6가지의 범위를 지원하고, 이중 4개는 web-aware ApplicationContext를 사용하는 경우에 사용할 수 있다. 또한 개발자는 사용자 정의 범위를 정의할 수 있다.

**다음은 Spring이 지원하는 범위(scope)를 나타낸다.**

- **singleton(default) :** 하나의 Bean 정의에 대해서 Spring IoC Container 내에 단 하나의 객체만 존재한다.
- **prototype :** 하나의 Bean 정의에 대해서 다수의 객체가 존재할 수 있다.
- **request :** 하나의 Bean 정의에 대해서 하나의 HTTP request의 생명주기 안에 단 하나의 객체만 존재한다. HTTP request는 하나의 Bean 정의의 뒷면에서 생성된 하나의 Bean의 인스턴스만 가지고 있습니다. request scope는 web-aware Spring ApplicationContext 내에서만 유효하다.
- **session :** 하나의 Bean 정의에 대해서 하나의 HTTP Session 라이프사이클 안에 단 하나의 객체만 존재한다. Web-aware Spring ApplicationContext 안에서만 유효하다.
- **application :** 하나의 Bean 정의에 대해서 하나의 ServletContext 생명주기 안에 단 하나의 객체만 존재한다. Web-aware Spring ApplicationContext 안에서만 유효하다.
- **websocket :** 하나의 Bean 정의에 대해서 하나의 WebSocket 생명주기 안에 단 하나의 객체만 존재한다. Web-aware Spring ApplicationContext 안에서만 유효하다.

> Spring 3.0 부터, thread scope를 이용가능하지만 기본적으로 등록이 되어있지 않습니다. 정보를 더 얻고 싶다면 SimpleThreadScope를 위한 documentation을 보십시요. 이러한 범위 또는 다른 범위를 등록하기 위한 방법에 대한 지침은, Using a Custom Scope를 참고하십시요.

### 1.5.1 The Singleton Scope

**Singleton Bean의 공유 인스턴스 하나만 관리되며, 해당 Bean 정의와 일치하는 ID를 가진 Bean에 대한 모든 요청은 Spring Container가 해당 특정 Bean 인스턴스를 리턴하도록 합니다.** 

**다시 말하면(To put it another way), Bean의 정의를 정의하고 Singleton으로써 범위를 지정할 때 Spring IoC Container는 이러한 Bean의 정의에 의해 정의되는 객체의 인스턴스 하나를 생성한다. 이 단일 인스턴스는 singleton Bean의 캐시에 저장되며, 해당 이름이 지정된 Bean에 대한 후속 요청 및 참조는 캐시된 객체를 리턴합니다.**

다음 이미지는 어떻게 singleton scope가 동작하는지 보여준다.

<img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200124173236350.png" alt="image-20200124173236350" style="zoom:50%;" />

Spring의 Singleton Bean의 개념은 Gang of Four(GOF) 패턴 책에서 정의된 Singleton pattern과는 다르다. GOF Singleton은 ClassLoader마다 생성되어지는 특정 클래스의 인스턴스가 오직 하나만 생성되도록 개체의 범위(scope)를 하드 코딩 합니다. Spring Singleton의 범위는 container마다, Bean마다 잘 묘사되어 있다. 만약 개발자가 single Spring Container의 특정 클래스에 대해 하나의 Bean을 정의했다면, Spring Container는 Bean의 정의에 정의된 클래스의 인스턴스를 오직 하나 생성할 것이다. Singleton 범위는 스프링의 기본 범위이고. XML기반의 configuration metadata에서 Bean을 Singleton으로 써 정의하기 위해서는 다음과 같이 보여지는 예제로써 Bean을 정의하면 된다.

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>
   
<!-- 다음은 비록 불필요할 지라도 동일하다. -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

### 1.5.2 The Prototype Scope

**Bean 배포의 non-singleton prototype 범위는 특정한 Bean이 만들어 지기 위해 매 요청마다 새로운 Bean의 인스턴스를 생성한다. 즉, Prototype Scope Bean은 다른 Bean 또는 컨테이너의 getBean() 메소드 호출을 통해서 개발자가 요청한 Bean에 주입되어진다.** 일반적으로, 개발자는 Stateful(이전 상태를 기록) Bean을 위해서 prototype scope를 사용하고, stateless(이전 상태를 기록 x) bean을 위해서 singleton scope를 사용한다.

다음은 Spring prototype scope를 설명하는 다이어그램이다.

<img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200124175642989.png" alt="image-20200124175642989" style="zoom:50%;" />

DAO(data access object)는 어떠한 conversational 상태를 유지하지 않음으로 prototype으로써 설정하지 않는다. singleton Bean을 사용하는 것이 더 쉽다.

다음은 XML에서 프로토 타입의 Bean을 정의하는 예제이다.

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

다른 범위(scope)와는 대조적으로, Spring은 prototype Bean의 생명주기를 완전히 관리하지 않는다. **컨테이너는 프로토 타입 객체를 인스턴스화, 구성 및 조립하고 프로토 타입 인스턴스에 대한 추가 기록없이 클라이언트에게 전달합니다**. 따라서, 비록 생명주기 초기화 콜백 메소드는 범위에 관계없이 모두 호출되어지지만, 프로토 타입의 경우 설정된 생명주기 소멸 콜백은 호출되지 않는다. 클라이언트 코드는 프로토타입 범위의 객체를 정리하고, 프로토타입 Bean이 보유한 비싼 자원을 받아서 사용해야 합니다.  Prototype범위의 Bean이 가지고 있는 자원을 얻기 위해서 Spring Container는 정리하고자 하는 Bean을 참조해주는 사용자 Bean post-processor를 사용해야 합니다.

어떤 경우에는(In some respects), 프로토 타입 범위의 Bean에 대한 Spring의 역할은 Java의 new 연산자를 대체합니다. 해당 시점 이후의 (past that point) 모든 생명주기 관리는 클라이언트에 의해서 다루어지고 있다.
(Spring Container에서의 Bean 생명주기에 대해서 더 알고 싶다면, Lifecycle Callbacks를 참고하자.)

### 1.5.3 Singleton Beans with Prototype-bean Dependencies

prototype Bean에 대한 의존성이 있는 singleton범위의 Bean을 사용할 때, 의존성은 인스턴스가 되는 시간에 해결된다는 것을 알아야 한다. 이와같이(thus), 만약 개발자가 prototype 범위의 Bean을 singleton 범위의 Bean에 의존성 주입한다면, 새로운 prototype Bean은 인스턴스화 되고 singleton범위의 Bean에게 주입되어질 것이다. 프로토 타입 인스턴스는 Singleton 범위의 Bean에게 제공되어지는 유일한 인스턴스 입니다.

그러나, 런타임 시에 singleton 범위의 Bean이 반복적으로 prototype 범위의 Bean의 새로운 인스턴스를 얻고 싶다고 가정하면, 개발자는 prototype의 범위의 인스턴스를 singleton 범위의 Bean에 의존성 주입할 수 없다. Spring Container가 singleton Bean을 인스턴스화 하고 이것의 의존성을 주입하고 해결할 때 딱 한번 의존성이 주입되기 때문이다. 만약 런타임시에 prototype Bean의 새로운 인스턴스가 한번 이상 필요하다면, Method Injection을 참고하자. 

### 1.5.4 Request, Session, Application, and WebSocket Scopes

request, session, application, websocket 범위는 web-aware Spring ApplicationContext(XmlWebApplicationContext와 같은)를 구현할 때만 사용가능한 Bean scopes이다. 만약 일반적인 Spring IoC Container(ex. ClassPathXmlApplicationContext)와 함께 이러한 Bean Scopes를 사용한다면, 잘 알려지지 않는 Bean Scope에 대한 예외를 말해주는 IllegalStateException 이 던져질 것이다.

#### Initial Web Configuration

request, session, application, websocket 레벨에서 (web 범위의 Bean) Bean의 범위지정(scoping)을 지원하기 위해, 개발자가 Bean을 정의하기 전에 몇가지의 초기 configuration이 필요하다. ( 이러한 초기 설정은 singleton과 prototype scope와 같은 표준 범위 Bean에는 포하되지 않는다.)

**이러한 초기 설정을 수행하는 방법은 특정 서블릿 환경에 의존합니다.**

**만약 Spring Web MVC 내에서 범위가 설정된 Bean에 접근한다면, 사실상 DispatcherServlet에 처리되는 요청 내에서 어떠한 특별한 설정도 필요하지 않습니다. DispatcherServlet은 이미 모든 관련된 상태를 노출합니다.**

Spring DispatcherServlet 외부에서 요청이 처리되는 Servlet 2.5 web Container를 사용하는 경우(ex. JSF, Structs를 사용하는 경우), 개발자는 org.springframework.web.context.request.RequestContextListener ServletRequestListener를 등록해야한다. Servlet 3.0 이상부터는, WebApplicationInitializer 인터페이스를 사용함으로써 이러한 초기 설정들이 프로그램적으로 행해진다. 대안으로 또는 더 오래된 Container를 위해서, 다음의 선언을 application의 web.xml 파일에 추가해야한다.

```xml
<web-app>
	<!-- ... -->
   
   <listener>
   	<listener-class>
      	org.springframework.web.context.request.RequestContextListener
      </listener-class>
   </listener>
   
	<!-- ... -->
</web-app>
```

그 대신에, 리스너 설정에 문제가 있는 경우, Spring의 RequestContextFilter를 사용하는 것을 고려해보아라. Filter mapping은 관련된 web application 설정에 의존할것이고, 개발자는 이러한 web application configuration을 적절히 수정해야한다.

다음은 web application의 filter 파트를 보여준다.

```xml
<web-app>
	...
   <filter>
   	<filter-name>requestContextFilter</filter-name>
      <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
   </filter>
   <filter-mapping>
   	<filter-name>requestContextFilter</filter-name>
      <url-patter>/*</url-patter>
   </filter-mapping>
   ...
</web-app>
```

DispatcherServlet, RequestContextListener, RequestContextFilter 모두 정확히 같은 역할을 수행한다. 즉(namely) HTTP request 객체를 해당 요청을 처리하는 Thread에 바인딩합니다. 이를 통해 요청 및 세션 범위의 Bean을 call chain에서 추가로 사용할 수 있습니다.

#### Request scope

bean의 정의를 위한 XML 기반의 configuration을 고려해보자

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

**Spring Container는 매번 각 HTTP request에 대하여 loginAction Bean definition 사용함으로써 LoginAction Bean 의 새로운 인스턴스를 생성합니다.** 즉, loginAction Bean은 HTTP request level에서 범위가 설정됩니다. 개발자는 원하는 만큼의 생성된 인스턴스의 내부 상태를 변경할 수 있습니다. 같은 loginAction Bean 정의에서 생성된 다른 인스턴스들은 이러한 상태의 변화를 볼 수 없기 때문입니다. 이러한 Bean들은 개별 요청에 따라 특별(particular)하다. request의 process가 완료될 때, request로 범위가 지정된 Bean은 소멸됩니다.(discard)

annotation 중심의 compoents 또는 Java configuration을 사용할 때, @RequestScope 애노테이션은 component를 request 범위에 할당하는데 사용됩니다.

다음은 어떻게 사용하는 지에 대한 예제 입니다.

```java
@RequestScope
@Component
public class LoginAction{
   // ...
}
```

#### Session Scope

Bean 정의를 위한 다음 XML configuration을 고려해 보자.

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

**Spring Container는 단일 HTTP Session의 수명동안 userPreferences Bean 정의를 사용함으로써 UserPreference Bean의 새로운 객체를 생성합니다.** 다른 말로, userPreferences Bean은 효율적으로 Http Session 레벨에서 범위가 지정됩니다. request 범위 Bean과 같이, 개발자는 원하는 만큼 생성된 인스턴스의 내부 상태를 바꿀 수 있다.
같은 userPreferences Bean 정의로부터 생성되는 인스턴스를 사용하고 있는 다른 HTTP Session Instance를 다른 인스턴스의 상태 변화를 보지 못한다는 것을 알아야 한다. userPreference Bean은 개개의 HTTP Session에서 유일하기 때문입니다.  HTTP Session 이 소멸되어질 때, 특정 HTTP Session의 범위가 지정된 Bean 또한 소멸된다.

annotation 중심의 components 또는 Java configuration을 사용할 때, 개발자는 @SessionScope 애노테이션을 component를 session 범위로 할당하기위해 사용할 수 있다.

```java
@SessionScope
@Component
public class UserPreferences{
   // ...
}
```

#### Application Scope

Bean 정의를 위해서 다음의 XML 기반의 configuration을 고려해야 한다.

```java
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

**Spring Container는 전체 web application 동안 오직 단 한번 appPreferences Bean 정의를 사용해서 AppPreferences Bean의 새로운 인스턴스를 생성한다.** 즉, appPreferences Bean은 ServletContext level 에서 정의가 되어있고, ServletContext 의 일반적인 속성으로써 저장되어진다. Application Scope Bean은 다소 Spring singleton Bean과 비슷하지만, 2가지의 주요 차이점이 있다. Application Scope Bean은 ApplicationContext(주어진 웹 애플리케이션에 여러개의 ApplicationContext가 존재할 수 있다.) 마다가 아니라 ServletContext마다 singleton이다. Application Scope Bean은 ServletContext 속성으로써 볼수 있다.

annotation 중심의 component 또는 Java configuration을 사용할 때, 개발자는 @ApplicationScope 애노테이션을 component가 application 범위를 할당하도록 사용할 수 있다.

다음 예는 @ApplicationScope를 어떻게 사용하는지에 관한 예제이다.

```java
@ApplicationScope
@Component
public class AppPreferences{
   // ...
}
```

#### Spring-AOP, Proxy란?

##### **AOP란 ?**

AOP는 문제를 해결하기 위한 핵심 관심 사항과 전체에 적용되는 공통 모듈 사항을 기준으로 프로그래밍 함으로써 공통 모듈을 여러 코드에 쉽게 적용할 수 있도록 도와주는 역할을 합니다.

<img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200202231600331.png" alt="image-20200202231600331" style="zoom:50%;" />

위에 그림에서 공통기능은 직접적으로 호출되지 않습니다. 핵심로직을 구현한 코드를 컴파일하거나, 컴파일된 클래스를 로딩하거나, 또는 로딩한 클래스의 객체를 생성할 때, Proxy 객체를 통해 호출할 때 AOP가 적용됩니다.

##### **AOP 용어**

- Joinpoint : Advice를 적용 가능한 지점을 의미합니다. 메소드 호출, 필드값 변경 등이 Joinpoint에 해당 합니다.
- Pointcut : Joinpoint의 부분집합으로서 실제로 Advice가 적용되는 Joinpoint를 나타냅니다. 스프링에서는 정규 표현식이나 AspectJ의 문법을 이용하여 Pointcut을 재정의 할 수 있습니다.
- Advice : 언제 공통 관심 기능을 핵심로직에 적용할 지를 정의하고 있습니다.
- Weaving : Advice를 핵심로직코드에 적용하는 것을 waving이라고 합니다. 즉, 공통코드를 핵심로직코드에 삽입하는 것을 weaving이라고 합니다.
- Aspect : 여러 객체에 공통으로 적용되는 기능을 Aspect라고 합니다. 트랜잭션이나, 보안등이 Aspect의 좋은 예입니다.

##### **Advice를 Weaving하는 3가지 weaving 방식**

1. 컴파일시에 Weaving 하기 : AspectJ라이브러리를 추가하여 구현
2. 클래스 로딩 시에 Weaving 하기 : 
3. 런타임시에 Weaving 하기 : Spring-AOP 에서 사용하는 방식 프록시를 생성하여 AOP를 적용한다.

**프록시 기반의 AOP는 핵심 로직을 구현할 객체에 직접 접근하는 것이 아니라 아래 그림과 같이 중간에 프록시를 통해 핵심 로직의 객체에 접근하는 것입니다.**

#### 프록시를 이용한 AOP 구현

스프링은 프록시를 이용하여 AOP를 구현합니다. 스프링은 Aspect의 적용대상이 되는 객체에 대한 프록시를 만들어 제공합니다. 비즈니스 로직에 접근할 때 댓아 객체로 바로 접근하는게 아니라 프록시르 통해서 간접적으로 접근하게 됩니다. 이 과정에서 프록시는 공통 기능을 실행한 뒤 대상 객체의 실제 메서드를 호출하거나 또는 대상 객체의 실제 메소드를 호출한 후에 공통기능을 실행합니다.

#### Scoped Beans as Dependencies (개어렵다 진짜 다시 공부해야한다.!!!!)

Spring IoC Container는 Bean의 인스턴스화를 관리하는 것 뿐만 아니라 의존성(collaboraotrs)을 wiring 해주는 것까지 관리한다**. 만약 개발자가 HTTP resquest-scoped Bean을 더 긴 수명을 갖는 범위의 Bean에게 주입하고 싶다면, 개발자는 AOP proxy를 범위가 지정된 Bean 대신에(in place of) 주입해야한다.** 즉, 개발자는 프록시 객체를 주입해야 합니다. 프록시 객체는 실제 target 객체를 관련된 범위로부터 검색할 수 있고 메소드 호출을 실제 객체에 위임할 수 있는 범위의 객체로서 동일한 공용 인터페이스를 노출시킨다.

> 개발자는 `singleton` 으로써 범위가 지정된 Bean간의 `\<aop:scoped-proxy/>`를 사용할 수 있으며, 참조는 직렬화 가능한 중간 프록시를 거치므로 따라서 직렬화 해제시 target singleton Bean을 재확보할 수 있습니다.
>
> `prototype` 범위의 bean에 대해서 `\<aop:scoped-proxy/>`를 선언할 때, 모든 메소드는 호출이 전달(forward)되는 새로운 target 인스턴스의 생성을 발생하는 공유 프록시를 호출한다.
>
> 또한 범위가 지정된 프록시는 라이프 사이클이 안전한 방식으로 짧은 범위의 Bean에 액세스 할 수 있는 유일한 방법은 아닙니다. 개발자는 `ObjectFactory\<MyTargetBean>`으로써 injection point(즉, 생성자 또는 setter의 인자, autowired 필드)를 선언할 수 있다. 필요할 때마다 필요에 따라 현재의 인스턴스를 검색 할 수 있도록 `getObject()` 호출을 가능하게 한다. 인스턴스를 유지하거나 또는 이것을 별도로 저장하지 않고도 getObject 호출로써 가능하다.
>
> 확장 변형으로 `getIfAvailable` 과 `getIfUnique를` 포함한 몇가지의 추가적인 접근 변형을 제공(deliver)하는 `ObjectProvider\<MyTargetBean>`를 선언할 수 있습니다.
>
> `Provider라고` 불리우는 JSR-330 변형은 `Provider\<MyTargetBean>` 선언과 모든 검색 시도를 위한 `get()` 메소드의 호출을 위해 사용되어진다. 좀 더 많은 정보를 원한다면 JSR-330 overall을 참고해 보자.

다음 예제의 configuration은 단 한줄이지만, 그 뒤에 존재하는 방법 뿐만 아니라 이유를 하는것이 매우 중요하다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           https://www.springframework.org/schema/aop/spring-aop.xsd">
   
	<!-- 프록시로써의 HTTP Session 범위의 Bean 노출 -->
   <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
   	<!-- Container가 주변 Bean을 프록시하도록 지시합니다. -->
      <aop:scoped-proxy/> <!-- 프록시를 정의하는 라인 -->
   </bean>
   
   <!-- 프록시(위에 정의된 Bean)가 주입 된 singleton 범위의 Bean-->
   <bean id="userService" class="com.something.SimpleUserService">
   	<!-- 프록시된 userPreference Bean 참조 -->
      <property name="userPreferences" ref="userPreferences"/>
   </bean>
   
</beans>
```

**이러한 Proxy를 만들기 위해서, 개발자는 범위가 결정된 Bean의 정의에 \<aop:scoped-proxy/> 요소를 삽입해야한다.**
( Choosing the Type of Proxy to Create 과 XML Schema-based configuration을 참고해서 보자 ). request, session, custom-scope level에서 범위가 결정된 Bean의 정의에서 \<aop:scoped-proxy/> 요소가 왜 필요한가? 
다음의 Singleton Bean 정의를 고려하여 앞서말한(aforementioned) 범위를 위해서 무엇을 정의해야하는가와 대조하십시요.
( 다음의 userPreferences Bean 정의는 현재 상태 그대로(as it stands) 불완전하다는 것을 명심해야 합니다.)

```xml
<bean id="userPreferences" class="com.soemthing.userPreferences" scope="session"/>
   
<bean id="userManager" class="com.something.UserManaer">
	<property name="userPreferences" ref="userPreferences"/>
</bean>
```

이전의 예제에서, Singleton Bean(userManager)는 HTTP Session scoped Bean(userPreferences) 참조가 주입되었다. 여기서의 중요한(salient) 포인트는 userManager Bean이 singleton이라는 것이다. 이것은 Container마다 확실히 한번만 인스턴스화 되고, 이것의 의존성( 이번 경우에는 userPreferences Bean ) 은 오직 한번만 의존성 주입되어진다. 이러한 것들은 userManager Bean이 정확히 동일한 userPreferences 오브젝트(즉, 원래 주입 된 오브젝트)에서만 작동함을 의미합니다.

짧은 수명의 범위를 가진 Bean을 긴 수명의 범위를 가진 Bean에 주입할 때 개발자가 원하는 동작이 아니다.
(예로들어, HTTP Session 범위의 collaborating Bean을 의존성으로 singleton Bean에 주입) 오히려, 개발자는 단일의 userManager 객체 필요로하며, HTTP Session 수명주기 동안 HTTP Session과 관련된 userPreferences 객체가 필요합니다. 따라서, Container는 범위 지정 메커니즘(HTTP request, Session, ...)으로부터 실제 UserPreferences 객체를 가져올(Fetch) 수 있는 UserPreferences 클래스와 정확히 동일한 공용 인터페이스(이상적으로 UserPreferences 인스턴스인 객체)를 노출하는 객체를 만듭니다. **Container는 proxy 객체를 userManager Bean에 주입한다. userManager Bean은 UserPreferences 참조가 프록시 라는 것을 알지 못한다.** 이러한 예제에서, UserManager 인스턴스가 UserPreferences 객체의 의존성이 주입된 method를 호출할 때, **UserManager는 실제로 프록시에서 이러한 메소드를 호출한다.** 프록시는 실제 UserPreferences 객체를 HTTP Session으로부터 가져오고, 메소드 호출을 검색된 실제 UserPreferences 객체로(onto) 위임한다.

**따라서 개발자는 request 와 session 범위의 Bean을 collaborating objects에 주입할 때 다음(올바르고 완전한)과 같은 configuration이 필요합니다.** 

```xml
<bean id="userPreferences" class="com.something.userPreferences" scope="session">
         <aop:scoped-proxy/> <!-- 프록시를 정의하는 라인 -->
</bean>

<bean id="userManger" class="com.something.UserManager">
	<property name="userPreferences" ref="userPreferences"/>
</bean>
```

#### Choosing the Type of Proxy to Create (작성할 프록시 유형 선택)

기본적으로, Spring Container가 \<aop:scoped-proxy/> 요소로 표시된 Bean을 위해서 프록시를 생성할 때, GGLIB 기반의 클래스 프록시가 생성되어진다.

> GGLIB 프록시들은 공용 메소드 호출만 인터셉트 합니다. 이러한 프록시에서 비공개 메소드를 호출하지 마십시요. GGLIB 프록시들은 실제 범위가 지정된 target 객체에게 위임되어지지 않았습니다.

그 대신에, 개발자는 범위가 지정된 Bean을 위해서 표준 JDK 인터페이스 기반의 프록시들을 생성하기 위해 Spring Container를 설정할 수 있습니다. \<aop:scoped-proxy/> 요소의 proxy-target-class 속성의 값을 false로 명시함으로써 가능하다. JDK 인터페이스 기반의 프록시를 사용하는 것은 프록싱에 영향을주기 위해 애플리케이션 클래스 경로에 추가 라이브러리가 필요하지 않다는 것을 의미한다. 그러나, JDK 인터페이스 기반의 프록시를 사용하는 것은 범위가 지정된 Bean의 클래스는 반드시 최소한 하나 이상의 인터페이스를 구현해야 한다는 것을 의미하고 범위가 지정된 Bean이 주입된 모든 협업자가 해당 인터페이스 중 하나를 통해 Bean을 참조해야 함을 의미한다.

다음 예제는 인터페이스를 기반으로하는 프록시를 보여줍니다.

```xml
<!-- UserPreferences 인터페이스를 구현하는 DefaultUserPreferences -->
<bean id="userPreferences" class="com.stuff.DefaultuserPreferences" scope="session">
	<aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
	<property name="userPreferences" ref="userPreferences"/>
</bean>
```

class 기반 또는 인터페이스 기반의 proxing 선택에 대해서 더 세세한 정보를 알고 싶다면 Proxying Mechanisms를 참고하세요.

### 1.5.5 Custom Scopes

Bean 범위지정 메커니즘은 확장가능하다. 개발자는 자신만의 범위를 정의할 수 있고 심지어는 존재하는 범위를 다시 정의할 수도 있다. 후자는 나쁜 관습으로 정의되어 있다. 개발자는 내장된 Singleton 과 Prototype의 범위를 오버라이드 할 수 없다.

#### Creating a Custom Scope

**사용자 범위를 SpringContainer에 통합하기 위해서는, 개발자는org.springframework.beans.factory.config.Scope 인터페이스를 구현해야한다.** (이것은 이번 섹션에서 설명되어진다.) 사용자 범위를 구현하는 방법에 대한 아이디어를 위해, Spring Framework 자체에서 제공해주는 Scope implementations 함께 Scope javadoc을 보아라. 이러한 문서에는 자세히 구현해야하는 메소드가 설명되어 있습니다.

Scope 인터페이스에는 범위로부터 객체를 얻고, 범위로부터 객체를 제거하고, 객체를 소멸되게하는 4가지의 메소드를 가지고 있습니다.

**session 범위의 구현은 session 범위의 Bean을 리턴한다. (존재하지 않는 경우, 메소드는 나중에 참조하기 위해 세션에 바인딩 한 후, Bean의 새 인스턴스를 리턴합니다.)**

다음 메소드는 기본 범위에서의 객체를 리턴한다.

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

예를 들어, 세션 범위 구현은 기본 세션에서 세션 범위의 Bean을 제거합니다. 객체는 리턴 되어야하지만, 만약 특정한 이름을 가진 객체가 발견되지 않으면 null이 리턴될 수 있습니다. 

다음 메소드는 기본 범위로부터 객체를 제거합니다.

```java
Object remove(String name)
```

다음은 범위가 소멸되거나 범위안의 특정한 객체가 소멸될 때 범위가 실행해야하는 콜백을 등록하는 메소드이다.

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

destruction callbacks에 대한 더 많은 정보를 위해 javadoc 또는 Spring scope implementation을 보아라.

다음은 기본 범위의 대화(conversation) 식별자를 가져오는 메소드이다.

```java
String getConversationId()
```

이 식별자는 각 범위마다 다르다. session 범위의 구현을 위해서, 이 식별자는 session 식별자가 될 수 있다.

#### Using a Custom Scope

하나 이상의 사용자 Scope 구현을 작성과 테스트 한 후, 개발자는 Spring Container가 새로운 사용자 Scope를 인지하도록 해야한다. 다음은 Spring Container에 새로운 Scope를 등록하는 주요 메소드이다.

```java
void registerScope(String scopename, Scope scope);
```

**이 매소드는 ConfigurableBeanFactory 인터페이스에 선언되어 있는데, 이는 Spring과 함께 제공되는 대부분의 구체적인 ApplicationContext 구현에서 BeanFactory 특성을 통해 사용가능합니다.**

registerScope(...) 메소드의 첫 번째 인자는 scope와 관련된 유일한 이름이다. Spring Container 자체에서 이러한 이름의 예로는 singleton 과 prototype이 있다. registerScope(...) 메소드의 두 번째 인자는 개발자가 등록 또는 사용하고 싶은 사용자 Scope 구현의 실제 인스턴스이다.

사용자 Scope 구현을 작성하고 다음 예와 같이 등록한다고 가정합시다.

> 다음 예제는 Spring에 포함되어 있지만 기본적으로 등록되지 않는 SimpleThreadScope를 사용합니다. 명령은 사용자 Scope 구현에 동일합니다.

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

개발자는 다음과 같이 사용자 정의 Scope의 scoping 규칙을 지키는 Bean 정의를 생성할 수 있다.

```xml
<bean id="..." class="..." scope="thread">
```

사용자 지정 Scope 구현에 대하여, Scope 프로그래밍방식 등록으로 제한되어 있지 않습니다. 개발자는 선언적으로 Scope 등록을 수행할 수 있습니다. CustomScopeConfigurer class를 다음 예제와 같이 사용함으로써

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread"> <!-- Scope name --> 
                    <!-- Scope class -->
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

FactoryBean 구현에서 \<aop:scoped-proxy/>를 위치시킬 때, getObject() 메소드에서 리턴된 객체가 아니라 범위가 지정된 팩토리 Bean 자체입니다.





















