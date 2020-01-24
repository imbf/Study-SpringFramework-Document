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















































