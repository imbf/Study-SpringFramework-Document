## 1.12 Java-based Container Configuration

Spring Container를 설정하기 위해서 자바 코드에서 어떻게 애노테이션을 사용할 수 있는지에 대해서 이번 섹션에서 다룰 것이다. 이번 섹션은 다음의 예제를 포함합니다.

- Basic Concepts : `@Bean` and `@Configuration`
- Instantiating the Spring Container by Using `AnnotationConfigApplicationContext`
- Using the `@Bean` Annotation
- Using the `@Configuration` annotation
- Composing Java-based Configurations
- Bean Definition Profiles
- `PropertySource` Abstraction
- Using `@PropertySource`
- Placeholder Resolution in Statements

---

### 1.12.1 Basic Concepts : `@Bean` and `@Configuration`

**Spring의 새로운 Java-Configuration 지원의 핵심 아티팩트 `@Configuration` 애노테이션이 붙여진 클래스와 `@Bean` 애노테이션이 붙여진 메소드이다.**

**`@Bean` 애노테이션은 메소드가 Spring IoC Container가 관리 할 새로운 객체를 인스턴스화, 설정, 초기화 함을 나타내는데 사용한다.** **Spring `<beans/>` XML 설정과 비슷하게, `@Bean` 애노테이션은 `<bean/>` 요소와 같은 역할을 수행한다.** 개발자는 `@Bean` 애노테이션이 붙여진 메소드들을 Spring `@Component`와 함께 사용할 수 있다. 하지만, 이러한 메소드들은 `@Configuration` Bean에서 가장 많이 사용된다.

**`@Configuration` 애노테이션이 붙여진 클래스는 Bean 정의가 최우선의 목적임을 나타낸다.** 게다가, `@Configuration` 클래스는 같은 클래스의 다른 `@Bean` 메소드를 호출함으로써 내부 Bean 의존성을 규정할 수도 있습니다. 다음의 예제에서 보여준다.

```java
@Configuration
public class ApplicationConfig {
   
   @Bean
   public ArrayList<Integer> array(){
      return new ArrayList<Integer>();
   }
   
   @Bean
   public Student student() {
      return new Student(array()); // array() 메소드 호출
   }
}
```

@Configuration의 가장 간단한 예제는 다음과 같다.

```java
@Configuration
public class AppConfig {
   
   @Bean
   public MyService myService() {
      return new MyServiceImpl();
   }
}
```

위의 `AppConfig` 클래스는 다음의 Spring `<beans/>` XML과 동일하다.

```xml
<beans>
	<bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

> #### Full @Configuration vs "lite" @Bean mode?
>
> **`@Configuration` 애노테이션이 붙여지지 않는 클래스에서 `@Bean` 메소드를 선언했을 때, 이러한 것들은 "lite" 모드에서 처리되는 것으로 언급되어집니다.** `@Component` 또는 심지어 평범한 class에서 선언되어진 Bean 메소드들은 "lite"로 고려되어지고 포함하는 클래스의 주요목적이 다른 주요 목적과 `@Bean` 메소드는 일종의 보너스입니다. 예로들어, Service Component는 이용가능한 각 component 클래스에서 추가적인 `@Bean` 메소드를 통하여 Container에게 관리 view 노출 할 수 있습니다. 이러한 시나리오에서, `@Bean` 메소드는 범용 팩토리 메소드 매커니즘입니다.
>
> **전체 `@Configuration`과 달리, lite `@Bean` 메소드들은 내부 Bean 의존성을 선언할 수 없습니다. 대신에, 그들은 포함하는 Component의 내부 상태 및 선택적으로 선언 할 수있는 인수에 따라 작동합니다.** 그러므로 이러한 `@Bean` 메소드는 다른 @Bean 메소드들을 호출할 수 없습니다. 이러한 메소드(lite `@Bean` method)는 특별한 runtime semantics 없이 말 그대로 특정 Bean 참조에 대한 팩토리 메소드 입니다. 긍정적인 효과는 런타임 시에 어떠한 GGLIB subclassing이 적용되지 않는 다는 것이고, 그럼으로 클래스 디자인에 관해서 어떠한 제한사항이 없습니다. (즉, 포함하는 클래스는 `final`)
>
> **공통된 시나리오에서, `@Bean` 메소드들은 `@Configuration` 클래스 내에서 선언되어, "full" 모드가 항상 사용되어지고 해당 메소드 간의 참조가 Container lifecycle 관리로 재지정(redirect)되도록 합니다.** 이러한 사실은 동일한 `@Bean` 메소드가 일반적인 자바 호출을 통해서 호출되어지는것을 막습니다, 이러한 것들은 'lite' 모드에서 작동 할 때 추적하기 어려 세밀한(subtle) 버그들을 줄이는 데 도움이 됩니다..

`@Bean` 과 `@Configuration` 애노테이션은 다음 섹션에서 깊게 논의됩니다. 첫 번째로, 우리는 Java-based configuration을 사용해서 Spring Container를 만드는 다양한 방법에 대해서 다룹니다.

---

### 1.12.2 Instantiating the Spring Container by Using `AnnotationConfigApplicationContext`

다음 섹션은 Spring 3.0에서 소개된 Spring `AnnotationConfigApplicationContext` 에 관한 document입니다. 이러한 다목적 `ApplicationContext` 구현은 `@Configuration` 클래스, `@Component` 클래스, JSR-330 metadata가 포함된 애노테이션이 붙은 클래스들을 input으로써 수용할 수 있습니다.

(매우 중요한 개념)
<u>**`@Configuration` 클래스들이 input으로써 제공되어진다면, `@Configuration` 클래스 자체는 Bean 정의로써 등록 되어지고  클래스 내에 선언되어진 모든 `@Bean` 정의로 등록되어집니다.**</u> 

`@Component` 와 JSR-330 클래스들이 제공되어질 때, 그들은 Bean 정의로써 등록되어지고, 의존성 주입 metadata (`@Autowired` 또는 `@Inject`)는 필요한 경우 해당 클래스내에서 사용되는 것으로 가정하니다.

#### Simple Construction

**`classPathXmlApplicationContext`를 인스턴스화 할 때 Spring XML 파일에서 input으로써 사용되어지는 것과 같은 방법으로, 개발자는 `AnnotationConfigApplicationContext`를 인스턴스화 할 때  `@Configuration` 클래스를 input으로써 사용할 수 있다.** 이러한 것들은 Spring Container에서 XML없이(XML-free) 사용할 수 있습니다.

```java
public static void main(String[] args){
   ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
   MyService myService = ctx.getBean(MyService.class);
   myService.doStuff();
}
```

이전에 언급했듯이, `AnnotationConfigApplicationContext`는 오직 `@Configuration` 클래스와 작업하는데 한계가 없다. 모든 `@Component` 또는 JSR-330 애노테이션 클래스들은 생성자에 input으로써 제공되어질 수 있다. 다음 예제에서 보여준다.

```java
public static void main(String[] args){
	ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
   MyService myService = ctx.getBean(MyService.class);
   myService.doStuff();
}
```

이전의 예제는 `MyServiceImpl`, `Dependency1`, `Dependency2`는 Spring 의존성 주입 애노테이션(`@Autowired`와 같은)을 사용한다고 가정하였다.

#### Building Container Programmatically by Using(Class<?>...)

개발자는 `AnnotationConfigApplicationContext`를 인자가 없는 생성자를 사용함으로써 인스턴스화 할 수 있고 `register()` 메소드를 사용함으로써 `AnnotationConfigApplicationContext`를 설정할 수 있습니다. 이러한 접근은 `AnnotationConfigApplicationContext`를 프로그램적으로 빌드할 때 매우 유용합니다. 다음예제에서 어떻게 이러한 접근방법을 사용하는지에 대해서 알려줍니다.

```java
public static void main(String[] args){
   AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
   ctx.register(AppConfig.class, OtherConfig.class);
   ctx.register(AdditionalConfig.class);
   ctx.refresh();
   MyService myService = ctx.getBean(MyService.class);
   myService.doStuff();
}
```

#### Enabling Component Scanning with `scan(String...)`

Component Scanning을 활성화 하기 위해서, 개발자는 `@Configuration` 클래스를 다음과 같이 어노테이션 시킬 수 있습니다.

```java
@Configuration
@ComponentScan(basePackages = "com.acme") // (1)
public class AppConfig {
   // ...
}
```

1. (1) 에노테이션은 Component Scanning을 가능하게 합니다.

> 경험많 Spring 사용자들은 Spring의 `context:` namespace 와 동등한 XML 선언에 친숙할 수 있습니다. 다음 예제에서 보여줍니다.
>
> ```xml
> <beans>
> 	<context:component-scan base-package="com.acme"/>
> </beans>
> ```

이전 예제에서,  `@Component` 애노테이션이 붙은 모든 클래스를 찾기 위해서  `com.acme` 패키지는 스캔이 되어집니다. 그리고 이러한 클래스들은 Spring Bean 정의로써 Container 내에 등록되어집니다. ``AnnotationConfigApplicationContext`는 같은 component-scanning 기능을 허용하기 위한 `scan(String...)` 메소드를 제공합니다. 다음 예제에서 보여줍니다.

```java
public static void main(String[] args){
   AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
   ctx.scan("com.acme");
   ctx.refresh();
   MyService myService = ctx.getBean(MyService.class);
}
```

>  **`@Configuration` 클래스는 `@Component`로 meta-annotated 되어집니다, 그래서 `@Configuration` 클래스는 component-scanning을 위한 후보자 입니다.** 이전의 예제에서, `AppConfig`가 `com.acme` 패키지 내에 정의되어져 있다고 가정하면, `scan()` 메소드를 호출하는 동안에 이러한 클래스가 선택되어집니다. `refresh()` 하자마자, 모든 `@Configuration` 클래스의 `@Bean` 메소드들은 처리되어지고, Container내에 Bean 정의로써 등록되어진다.

#### Support for Web Application with `AnnotationConfigWebApplicationContext`

**`AnnotationConfigApplicationContext`의 `WebApplicationContext` 변형은 `AnnotationConfigWebApplicationContext` 와 함께 사용할 수 있다.** 개발자는 Spring `ContextLoaderListener` 서블릿 리스너, Spring MVC `DispatcherServlet` 등을 설정할 때 이러한 구현을 사용할 수 있다. 다음의 `web.xml` 정보는 전형적인 Spring MVC web application을 설정한다. (`contextClass` context-param 및 init-param 사용에 주목)

```xml
<web-app>
	<!-- 기본적인 XmlWebApplicationContext 대신에 AnnotationConfigWebApplicationContext 사용을 위한 		ContextLoaderListener 설정 -->
   <context-param>
   	<param-name>contextClass</param-name>
      <param-value>
      	org.springframework.web.context.support.AnnotationConfigWebApplicationContext
      </param-value>
   </context-param>
   
   <!-- Configuration location은 하나 이상의 comma- 또는 space-delimited fully-qualified
		@Configuration 클래스들로써 이루어져있어야 한다. Fully-qualified 패키지들은 component-scanning을
		위해서 명시되어야 한다. -->e>com.
   <context-param>
   	<param-name>contextConfigLocation</param-name>
      <param-value>com.acme.AppConfig</param-value>
   </context-param>
   
  	<!-- contextLoaderListener 클래스는 DispatcherServlet 클래스의 로드보다 먼저 동작하여 비즈리스 로직층을 정의한 스프링 설정 파일을 로드한다. -->
   <listener>
   	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>
   
   <!-- Spring MVC DispatcherServlet을 선언 (일반적인 방법) -->
   <servlet>
   	<servlet-name>dispatcher</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <!-- 기본인 XmlWebApplicationContext 대신에 AnnotationConfigWebApplicationContext를 사용하기 			위한 DispatcherServlet 설정 -->
      
      <init-param>
      	<param-name>contextClass</param-name>
         <param-value>
         	org.springframework.web.context.support.AnnotationConfigWebApplicationContext
         </param-value>
      </init-param>
      
      <!-- 다시, config location은 무조건 하나 이상의 comma- 또는 spcae-delimited 
			그리고 fully-qualified @Configuration 클래스로 구성되어야 한다. -->
      <int-param>
      	<param-name>contextConfigLocation</param-name>
         <param-value>com.acme.web.MvcConfig</param-value>
      </int-param>
   </servlet>
   
   <!-- /app/* 으로오는 모든 요청을 dispatcher servlet으로 맵핑 -->
   <servlet-mapping>
   	<servlet-name>dispatcher</servlet-name>
      <url-pattern>/app/*</url-pattern>
   </servlet-mapping>
</web-app>
```

---

### 1.12.3 Using the `@Bean` Annotation

**@Bean**은 메소드 계층의 애노테이션이고 XML의 `<bean/>` 요소와 매우 비슷한 애노테이션이다. 이러한 애노테이션은 `<bean/>` 에 의해 제공되어지는 몇가지 속성을 제공해 준다. : `init-method`, `destory-method`, `autowiring`, `name`

개발자는 `@Configuration` 애노테이션이 붙은 클래스 또는 `@Component` 애노테이션이 붙은 클래스 내에서 `@Bean` 애노테이션을 사용할 수 있다.

#### Declaring a Bean

**Bean을 선언하기 위해서, 개발자는 `@Bean` 애노테이션을 메소드에 명시할 수 있습니다. 개발자는 이 메소드를 사용하여 지정된 타입의 `ApplicationContext` 내에서 메소드의 리턴 값으로 Bean 정의를 등록할 수 있습니다.** 기본적으로,  Bean name은 메소드 name과 같습니다. 다음 예제에서 `@Bean` 메소드 정의를 보여줍니다.

```java
@Configuration
public class AppConfig {
   
   @Bean
   public TransferServiceImpl transferService() {
      return new TransferServiceImpl();
   }
}
```

이전의 Annotation 기반의 설정은 다음 XML 기반의 설정과 완전히 동일하다.

```xml
<beans>
	<bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

양쪽 선언에서 `transferService` 로 명명된 Bean을 `ApplicationContext` 에서 사용할 수 있도록 하기 위해서, `TransferServiceImpl` 타입의 객체 인스턴스를 연결시켜라. 다음 텍스트 이미지에서 보여주듯이

```
transferService -> com.acme.TrasferServiceImpl
```

또한 개발자는 인터페이스(또는 base class) 리턴 타입을 가진 `@Bean` 메소드를 선언할 수 있다. 다음 예제에서 보여준다.

```java
@Configuration
public class AppConfig {
   
   @Bean
   public TransferService transferService(){
      return new TransferServiceImpl();
   }
}
```

그러나, 이러한 방법은 사전 타입 예측에 대한 가시성을 지정된 인터페이스 유형(`TransferService`)으로 제한(limit)합니다.
그리고, 전체 타입(`TransferServiceImpl`)을 컨테이너에 한번만 알려진 상태에서, 영향을 받는 Singleton Bean이 인스턴스화 되었습니다. Non-lazy singleton Bean들은 선언 순서에 따라서 인스턴스화 되어지고, 따라서 **또 다른 component가 선언되지 않는 타입과 일치시키려는 시기에 따라 다른 유형의 일치 결과가 표시될 수 있습니다.**(`@Autowired` `TransferServiceImpl`,  `TransferServiceImpl`은 `transaferService`가 인스턴스화 되어질 때 단 한번 resolve 합니다.)

> 만약 선언되어진 Service 인터페이스에 의해서 타입을 계속해서 참조한다면, `@Bean` 리턴 타입들은 해당 디자인 결정에 안전하게 참여할 수 있습니다. 그러나, 여러 인터페이스를 구현하는 Components 위하여 또는 자신의 구현 유형에 의하여 잠재적으로 참조되는 components를 위하여, 가능한 가장 특정한 리턴타입을 선언하는 것이 안전합니다. (최소한 Bean을 참조하는 의존성 포인트가 요구하는 정도)

#### Bean Dependencies

**`@Bean` 애노테이션이 붙여진 메소드는 해당 Bean을 빌드하기 위해 요구되어지는 의존성을 설명하는 임의의 수의 매개 변수를 가질 수 있습니다.** 예로들어, `TransterService`가 `AccountRepository`를 요구한다면, 우리는 메소드 인자로 의존성을 다음 예제와 같이 구체화시킬(materialize) 수 있습니다.

```java
@Configuration
public class AppConfig {
   @Bean
   public TransferService transferService(AccountRepository accountRepository){
      return new TransferServiceImpl(accountRepository);
   }
}
```

이러한 resolution 메커니즘은 생성자 기반의 의존성 주입과 매우 비슷하다. 더 많은 정보를 위해서 the relevant section(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-constructor-injection)를 참고해라.

#### Receving Lifecycle Callbacks

`@Bean` 애노테이션과 함께 정의된 모든 클래스들은 일반적인 Lifecycle callbacks을 지원하고 JSR-220의 `@PostConstruct` 와 `@PreDestroy` 애노테이션을 사용할 수 있습니다. 더 많은 정보를 위해서는 JSR-250 annotations(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)를 참고하자.

일반적인 Spring Lifecycle callbacks 또한 완전히 지원된다. Bean이 `InitializingBean`, `DisposableBean`, `Lifecycle`을 구현한다면, 각 메소드들은 Container에 의해서 호출되어진다.

`*Aware` 인터페이스의 표준 세트(ex. `BeanFactoryAware`, `BeanNameAware`, `MessageSourceAware`, `ApplicationContextAware`, ...)은 완전히 지원된다.

`@Bean` 애노테이션은 Spring XML `bean`요소의 `init-method` 또는 `destory-method` 속성들 처럼 **임의의 초기화 및 소멸 콜백 메소드를 설정하는 것을 지원**해 줍니다. 다음 예제에서 보여줍니다.

```java
public class BeanOne {
   
   public void init() {
      // initialization logic
   }
}

public class BeanTwo {
   
   public void cleanup() {
      // destruction logic
   }
}

@Configuration
public class AppConfig {
   
   @Bean(initMethod = "init")
   public BeanOne beanOne() {
      return new BeanOne();
   }
   
   @Bean(destoryMethod = "cleanup")
   public BeanTwo beanTwo() {
      return new BeanTwo();
   }
}
```

> 기본적으로, public `close` 또는 `shutdown`을 가지는 Java configuration으로 정의한 Bean 메소드는 자동적으로 소멸 콜백에 참여합니다. 만약 public `close` or `shutdown` 메소드를 가지고 Container가 소멸 되었을 때 이러한 메소드들이 호출되어지는 것을 바라지 않는다면, 개발자는 `@Bean(destroyMethod="")` 애노테이션을 Bean 정의에 추가 함으로써 기본적인 함축된 mode를 비활성화 시킬 수 있습니다.
>
> 라이프 사이클이 애플리케이션 외부에서 관리되므로 JNDI로 획득한 자원에 대해서는 기본적으로 이러한 것들을 수행할 수 있습니다. 특히, Java EE 어플리케션 서버에서 문제가 있는 것으로 알려져 있는 `DataSource`를 위해서 항상 실행하십시요.
>
> 다음의 예제는 `DataSource`를 위한 자동 소멸 콜백 호출을 어떻게 막는지에 대해서 보여준다.
>
> ```java
> @Bean(destroyMethod="")
> public DataSource dataSource() throws NamingException {
>    return (DataSource) jndiTemplate.lookup("MyDS");
> }
> ```
>
> 또한 `@Bean` 메소드에서, 개발자는 일반적으로 프로그램적인 JNDI 검색을 사용할 수 있습니다. Spring의 `JndiTemplate` 또는 `JndiLocatorDelegate` helpers 또는 straight JNDI `InitialContext` 를 사용함으로써 프로그램적인 JNDI 검색을 사용할 수있습니다. `JndiObjectFactoryBean` 변형을 사용하지 않는다. `JndiObjectFactoryBean` 은  실제 target 타입 대신에 `FactoryBean` 타입으로써 리턴타입을 선언하도록 강요한다. 이러한 것들을 사용하면 제공된 리소스를 참조하려는 다른 `@Bean` 메소드에서 상호 참조 호출을 사용하기가 더 어려워 진다.

이전에 작성된 예제의 `BeanOne`의 경우, 생성자 내에서 직접적으로 `init()` 메소드를 호출하는 것과 동일하다. 다음 예제에서 보여준다.

```java
@Configuration
public class AppConfig {
   
   @Bean
   public BeanOne beanOne() {
      BeanOne beanOne = new BeanOne();
      beanOne.init();
      return beanOne;
   }
   
   // ...
}
```

자바에서 직접적으로 작업을 할 때, 개발자는 객체와 함께 자신이 원하는 작업을 할 수 있고, 항상 Container lifecycle에 의존할 필요가 없습니다.

#### Specifying Bean Scope

Spring은 Bean의 Scope를 명시할 수 있도록 하기 위해서 `@Scope` 애노테이션을 지원합니다.

#### Using the `@Scope` Annotation

**개발자는 `@Bean` 애노테이션으로 정의 되어진 Bean을 특정한 범위를 가지도록 정의할 수 있습니다.** 
개발자는 Bean Scopes(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-factory-scopes) 섹션에 명시되어있는 표준 범위 모두를 사용할 수 있습니다.

기본 범위는 `singleton` 이지만, 개발자는 이러한 범위를 `@Scope` 애노테이션을 사용해서 오버라이드할 수 있습니다.
다음 예제에서 보여줍니다.

```java
@Configuration
public class MyConfiguration {
   
   @Bean
   @Scope("prototype")
   public Encryptor encryptor() {
		// ...
   }
}
```

#### `@Scope` 또는 `scoped-proxy`

**Spring은 scoped-proxy(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection)를 통해서 범위가 정해진 의존성의 작업의 편리한 방법을 제공 해 준다.** XML configuration을 사용할 때 proxy를 생성하는 가장 쉬운 방법은 `<aop:scoped-proxy/>` 요소이다. 자바에서 `@Scope` 애노테이션과 함께 Bean을 설정하는 것은 `proxyMode` 속성으로 동등한 지원(XML configuration과)을 제공한다. 기본값은 no proxy(`ScopedProxyMode.NO`)이고 하지만 개발자는 `ScopedProxyMode.TARGET_CLASS` 또는 `ScopedProxyMode.INTERFACES`를 명시할 수 있습니다.

만약 범위가 정해진 프록시 예제를 XML reference documentation에서 자바에서 사용하는  `@Bean` 으로 복사(port)한다면, 이러한 것들은 다음과 닮을 것입니다.

```java
// an HTTP Session-scoped Bean exposed as a proxy
@Bean
@SessionScope
public UserPreference userPreferences() {
   return new UserPreference
}

@Bean
public Service userService() {
   UserService service = new SimpleUserService();
   // 프록시된 userPreferences Bean 참조
   service.setUserPreferences(userPreferences());
   return service;
}
```

#### Customizing Bean Naming

**기본적으로, configuration class들은 Bean 결과의 name으로써 `@Bean` 메소드의 name을 사용한다.** 그러나 이러한 기능은 다음 예제에서 보여준 대로 name 속성과 함께 override 될 수 있다.

```java
@Configuration
public class AppConfig {
   
   @Bean(name = "myThing") //Bean의 name은 myThing이다.
   public Thing thing() {
      return new Thing();
   }
}
```

#### Bean Aliasing(별명)

**Naming beans에서 논의되어진 것 처럼, 단일 Bean에 여러 name을 지정하는 것이 때때로 바람직 하며, Bean Aliasing(Bean 별명) 이라고 부르기도 합니다.** `@Bean` 어노테이션의 `name` 속성은 이러한 목적에서 String 배열을 수용한다. 다음의 예제에서는 단일 Bean을 위해 많은 별명들을 설정하는 방법에 대해서 보여준다.

```java
@Configuration
public class AppConfig {
   
   @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
   public DataSource dataSource() {
      // DataSource Bean을 인스턴스화, 설정, 리턴하는 Code가 위치해야 한다.
   }
}
```

#### Bean Description

**때때로, 단일 Bean의 디테일한 텍스트형식의 설명을 제공하는것이 도움이 될 때가 있다.** 이는 모니터링 목적으로 Bean이 (JMX를 통해) 노출 될 때 특히 유용할 수 있습니다.

단일 `@Bean`에 설명을 추가하기 위해서, 개발자는 `@Description` 애노테이션을 추가할 수 있다. 다음 예제에서 보여준다.

```java
@Configuration
public class AppConfig {

	@Bean
   @Description("Provides a basic example of a bean")
   public Thing thing(){
      return new Thing();
   }
}
```

---

### 1.12.4 Using the `@Configuration` annotation

**`@Configuration`은 Bean 정의에 대한 소스인 객체를 가리키는 클래스 레벨의 애노테이션이다. `@Configuration` 클래스들은 `@Bean` 애노테이션이 붙은 public methods를 통해서 Bean을 선언한다. `@Configuration` 클래스들의 `@Bean` 메소드들을 호출하는 것은 inter-bean dependencies를 정의하는데 사용되어질 수 있습니다.** 더 일반적인 정보를 얻고 싶다면
Basic Concepts : @Bean and @Configuration(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts)을 참고 해 보자.

#### Injecting Inter-bean Dependencies

Bean이 또 다른 Bean의 의존성을 가지고 있다면, 의존성의 표현하는 것은 하나의 Bean 메소드가 다른것을 호출하는 것처럼 간단합니다. 다음 예제에서 보여줍니다.

```java
@Configuration
public class AppConfig {

	@Bean
	public BeanOne beanOne(){
		return new BeanOne(beanTwo());
	}
   
   @Bean
   public BeanTwo beanTwo(){
      return new BeanTwo();
   }
}
```

이전 예제에서, `beanOne`은 생성자 인자를 통해 `beanTwo` 참조를 수용합니다.

> **inter-bean dependencies를 선언하는 메소드는 오직 `@Bean`메소드가 `@Configuration` 클래스 내에 선언되어졌을 때에만 동작한다. 개발자는 inter-bean dependencies를 기본적인 `@Component` 클래스를 사용함으로써 선언할 수 없다.**

#### Lookup Method Injection

이전에 언급했듯이, **lookup method injection**(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection)는 개발자가 드물게 사용하는 발전된 기능이다. **이러한 기능은 singleton 범위의 Bean이 prototype 범위의 Bean 의존성을 가질 때 유용하게 사용된다.** 이러한 타입의 configuration을 위해서 자바를 사용하는 것은 이러한 패턴을 구현하기 위한 많은 방법을 제공 해 준다. 다음 예제는 어떻게 lookup 메소드 주입을 사용하는지에 보여주는 예제이다.

```java
public abstract class CommandManager {
   public Object process(Object commandState) {
      // 적절한 Command 인터페이스의 새로운 인스턴스를 가져옵니다.
      Command command = createCommand();
      // Command instance에 상태 설정
      command.setState(commandState);
      return command.execute();
   }
   
   // okay.. 어디서 이러한 메소드를 구현하지?
   protected abstract Command createCommand();
   
}
```

**Java configuration을 사용함으로써, 개발자는 `CommandManger`의 서브클래스를 abstract `createCommand()` 메소드가 오버라이드 되어지는 곳에 생성할 수 있다. abstract `createCommand()` 메소드는 새로운 (prototype) command 객체를 찾습니다**. 다음 예제에서 어떻게 이러한 것들을 사용하는지에 대해서 알려줍니다. 

```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
   AsyncCommand command = new AsyncCommand;
   // 의존성 주입이 필요한 경우 여기에 명시해라.
   return command;
}

@Bean
public CommandManager commandManager(){
   // createCommand()를 사용하여 새로운 익명의 CommandManager 구현을 반환합니다.
   // 새로운 프로토타입의 Command 객체를 리턴하기 위한 오버라이드
   return new CommnadManager() {
      protected Command createCommand() {
         return asyncCommand();
      }
   }
}
```

#### Further Information About How Java-Based-Configuration Works Internally

`@Bean` 애노테이션이 붙여진 메소드가 2번 호출되는 것을 보여주는 다음 예제를 고려해 보자.

```java
@Configuration
public class AppConfig {

   @Bean
   public ClientService clientService1() {
      ClientServiceImpl clientService = new ClientServiceImpl();
      clientService.setClientDao(clientDao());
      return clientService;
   }
   
   @Bean
   public ClientService clientService2() {
      ClientServiceImpl clientService = new ClientServiceImpl();
      clientService.setClientDao(clientDao());
      return clientService;
   }
   
   @Bean
   public ClientDao clientDao(){
      return new ClientDaoImpl();
   }
}
```

<u>`clientDao()`는 `clientService1()` 에서 한번, `clientService2()` 에서 한번 호출됬다. 이러한 메소드들은 `ClientDaoImpl` 의 새로운 인스턴스를 생성하고 리턴하기 때문에, 일반적으로 2개의 인스턴스가 예측되어야 합니다(각 서비스 마다 하나씩). 이것은 명백히 문제가 될 것 입니다.</u> : Spring에서 인스턴스화 된 Bean은 기본적으로 싱글 톤 범위를 갖습니다.
**지금부터는 마법에 대해서 설명할 것입니다 : 모든 `@Configuration` 클래스들은 시작할 때 `CGLIB` 와 함께 서브클래스화 됩니다. 서브 클래스에서, 자손 메소드는 부모 메소드를 호출 하고 새로운 인스턴스를 생성하기 전에 모든 캐시된(범위가 지정된) Bean을 위해서 Container를 첫번째로 확인합니다.****

> 이러한 동작은 Bean의 범위에 따라서 달라질 것입니다. 여기에서는 Singleton에 대해서만 이야기 하고 있습니다.

> Sprign 3.2부터, GGLIB 클래스는 `org.springframework.cglib`에 리패키징 되어, Spring 핵심 JAR 파일 내에 직접적으로 포함되어 있기 때문에 classpath에 CGLIB를 추가할 필요가 없어졌습니다. 

> **시작시간에 CGLIB가 동적으로 기능을 추가하기 때문에 몇가지의 제약사항이 생깁니다. 특히, configuration class들은 무조건 `final` 이 되면 안됩니다. 그러나 Spring 4.3부터, 기본 주입을 위하여 `@Autowired` 또는 기본이 아닌 단일 생성자 선언 사용을 포함하여 모든 생성자가 configuration class에서 허용됩니다.**
>
> 개발자가 CGLIB-imposed 제한을 피하는 것을 선호한다면, `@Bean` 메소드들을 `@Configuration` 클래스가 아닌 클래스(ex. 평범한 @Component 클래스)에 선언하는 것을 고려해 보아라. **`@Bean` 메소드 사이의 메소드 간(cross-method) 호출은 인터셉트 되지 않음으로, 독점적으로 생성자 또는 메소드 레벨에서 종속성 주입에만 의존해야 합니다.**

---

### 1.12.5 Composing Java-based Configurations

**Spring Java-based Configuration 특징은 개발자가 설정의 복잡성을 감소할 수 있는 애노테이션을 구성할 수 있게 해준다.**

#### Using the `@Import` Annotation

**Spring XML 파일에서 `<import/>` 요소가 configuration을 모듈화하기 위해 사용되어지는 것처럼(Much as), `@Import` 애노테이션은 다른 Configuration 클래스로부터 `@Bean` 정의를 로딩할 수 있도록 해준다.** 다음 예제에서 보여주는 것처럼.

```java
@Configuration
public class ConfigA {
   
   @Bean
   public A a() {
      return new A();
   }
}

@Configuration
@Import(ConfigA.class)
public class Config B{
   
   @Bean
   public B b() {
      return new B();
   }
}
```

Context가 인스턴스화 될 때, `ConfigA.class` 와 `ConfigB.class` 모두를 명시하지 않고 다음 예제와 같이 `ConfigB`만 오직 명시적으로 제공하면 됩니다. (컨테이너 인스턴스화를 단순화 한다.)

```java
public static void main(String[] args){
   ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);
   
   // Bean A 와 Bean B 모두 이용 가능할 것이다.
   A a = ctx.getBean(A.class);
   B b = ctx.getBean(B.class);
}
```

**이러한 접근은 구성 중에 잠재적으로 많은 수의 `@Configuration` 클래스를 기억할 필요 없이 <u>하나의 클래스만 처리</u>하면 됨으로 컨테이너 인스턴스화를 단순화합니다.**

> Spring Framework 4.2 부터, `@Import`는 일반적인 Component 클래스들을 참조할 수 있고, `AnnotationConfigApplicationContext.regist` 메소드와 유사하다. `@Import` 애노테이션은 개발자가 Component scanning을 피하고 싶을 때 <u>모든 Component를 명확히 정의하기 위한 진입 지점</u>으로써 몇가지 configuration 클래스들을 사용함으로써 특히 유용하다. 

#### Injecting Dependencies on Imported `@Bean` Definitions

앞의 예제는 작동하지만 단순합니다. 대부분의 실질적인 시나리오에서, Bean은 configuration class에서 서로 의존성을 가지고 있습니다. XML을 사용할 때, 이러한 것들은 어떠한 컴파일러도 포함되지 않기 때문에 이슈가 아니었습니다. 그래서 개발자는 `ref="someBean"`을 선언할 수 있었고 Container가 초기화되는 동안에 스프링이 이러한 의존성들을 처리하는 것에 대해서 믿었습니다. **`@Configuration` 클래스들을 사용할 때, 자바 컴파일러는 다른 Bean에 대한 참조가 유효한 Java 구문이어야 한다는 점에서(in that) 설정 모델에 제약사항을 두었습니다.**

운 좋게도, 이러한 문제를 해결하는 것은 간단합니다. 우리가 이미 논의 했듯이, `@Bean` 메소드는 Bean 의존성들을 설명할 수 있는 임의의 수의 인자를 가질 수 있습니다. 여러 `@Configuration` 클래스들이 존재하는 다음의 실질적인 시나리오에 대해서 고려해 보아라. 각 Bean은 다른 `@Configuration` 클래스에 선언되어 있는 Bean들을 의존한다.

```java
@Configuration
public class ServiceConfig {
   
   @Bean
   public TransferService transferService(AccountRepository accountRepository){
      return new TransferServiceImpl(accountRepository);
   }
}

@Configuration
public class RepositoryConfig {
   
   @Bean
   public AccountRepository accountRepository(DataSource dataSource){
      return new JdbcAccountRepository(dataSource);
   }
}

@Configuration
@Import({ServiceCOnfig.class, RepositoryConfig.class})
public class SystemTestConfig {
   
   @Bean
   public DataSource dataSource() {
      // return new DataSource
   }
}

public static void main(String[] args){
   ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
   // 모든 configuraton 클래스에 연결됩니다.
   TransferService transferService = ctx.getBean(TransferService.class);
   transferService.transfer(100.00, "A123", "C456");
}

```

이러한 것들은 같은 결과를 일으킬 수 있는 또 다른 방법이다. **`@Configuration` 클래스들은 궁극적으로 Container의 다른 Bean이다. 즉, `@Configuration` 클래스는 `@Autowired` 와 `@Value` 주입 그리고 모든 다른 Bean 처럼 똑같은 기능을 사용할 수 있습니다.**

> 해당 방법으로 주입하려는 의존성이 가장 쉬운 종류인지 확인하세요. `@Configuration` 클래스들은 Context 초기화 동안에 꽤 일찍 처리되어진다. 이러한 방식으로 의존성을 강제로 삽입하면 예기치 않은 이른 초기화가 발생할 수 있습니다. 가능하면, 이전 예제처럼 매개변수 기반의 의존성 주입에 의지하세요(resort to).
>
> **또한 `@Bean`을 통한 `BeanPostProcessor` 또는 `BeanFactoryPostProcessor` 정의에 특히 주의하세요. 이러한 것들은 보통 `static @Bean` 메소드로써 선언되어야 하고, 그들을 포함하는 configuration 클래스의 인스턴스화를 발생시키면 안된다.** **그렇지 않으면, `@Autowired` 와 `@Value`는 너무 일찍 Bean 인스턴스로 생성됨으로 Configuration 클래스에서 작동하지 않습니다.**

다음은 어떻게 하나의 Bean이 또 다른 Bean에 autowired되어지는지에 대해서 보여주는 예제이다.

```java
@Configuration
public class ServiceConfig {
   
   @Autowired
   private AccountRepository accountRepository;
   
   @Bean
   public TransferService transferService() {
      return new TrasferServiceImpl(accountRepository);
   }
}

@Configuration
public class RepositoryConfig {

   private final DataSource dataSource;
   
   // @Configuration 클래스에 생성자 주입은 Spring Framework 4.3부터 지원하고, Target Bean이 오직 하나의 생성자만 정의 했다면, @Autowired를 명시해 줄 필요가 없다는 것에 주목해야 합니다!!!!
   public RepositoryConfig(DataSource dataSource) {
      this.dataSource = dataSource;
   }
   
   @Bean
   public AccountRepository accountRepository() {
      return new JdbcAccountRepository(dataSource);
   }
}

@Configuration
@Import({ServiceConfig.class, RespotiroyConfig.class})
public class SystemTestConfig {
   
   @Bean
   public DataSource dataSource() {
      // return new DatSource
   }
}

public static void main(String[] args){
   ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
   // 모든 configuration 클래스들에 연결합니다.
   TransferService transferService = ctx.getBean(TransferService.class);
   transferService.transfer(100.00, "A123", "C456");
}
```

> **`@Configuration` 클래스에 생성자 주입은 Spring Framework 4.3부터 지원합니다. 만약 Target Bean이 오직 하나의 생성자만 정의 했다면, @Autowired를 명시해 줄 필요가 없다는 것에 주목해야 합니다.**

#### Fully-qualifying imported beans for ease of navigation

**이전 시나리오에서, `@Autowired`를 사용은 잘 작동하고 원하는 모듈성을 지원하지만 어디에 autowired Bean 정의가 선언되었는지 판별하는 것은 아직도 다소 모호하다.** 예로들어, 개발자가 `ServiceConfig`를 보고 있을때 , 어디에 `@Autowired AccountRepository` Bean이 선언되어있는지 어떻게 알 것인가? 이러한 것들은 코드에서 명확하지가 않고 이것은 괜찮을 수도 있습니다. **Spring Tool Suite는 모든 것이 어떻게 wired되어 있는지 보여주는 그래프를 렌더링 할 수 있는 툴링을 제공한다는 것을 명심해야 합니다.** 또한, 개발자가 사용하는 IDE는 쉽게 모든 선언과 `AccoutRepository` 타입의 사용을 쉽게 찾아줄 것이고, 해당 타입을 반환하는 `@Bean` 메소드의 위치를 개발자에게 보여 줄 것이다.

**이러한 모호성이 허용되지 않고 IDE 내에서 하나의 `@Configuration`에서 다른 `@Configuration` 클래스로 직접 탐색하려는 경우, configuration 클래스 자체를 autowiring 하는 것을 고려해라.** 다음 예제에서 어떻게 이러한 것들을 사용하는지에 대해서 보여준다.

```java
@Configuration
public class ServiceConfig {
   
   @Autowired
   private RepositoryConfig repositoryConfig;
   
   @Bean
   public TransferService transferService(){
      // 설정 클래스를 @Bean 메소드로 탐색 합니다.
      return new TransferServiceImpl(repositoryConfig.accountRepository());
   }
}
```

**이전의 상황에서, `AccountRepository`가 정의된 곳은 완전히 명백하다. 그러나 `ServiceConfig`는 `RespositoryConfig` 매우 밀접하게 연결되어 있습니다. 이것이 바로 trade-off 입니다. 이러한 밀접한 결합은 인터페이스 또는 추상 클래스 기반의 `@Configuration` 클래스를 사용함으로써 다소 완화할 수 있습니다.** 다음 예제에서 고려해 보세요.

```java
@Configuration
public class ServiceConfig {
   
   @Autowired
   private RepositoryConfig repositoryConfig;
   
   @Bean
   public TransferService transferService() {
      return new TransferServiceImpl(repositoryConfig.accountRepository());
   }
}

@Configuration
public interface RepositoryConfig {
   
   @Bean
   AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {
   
   @Bean
   public AccountRepository accountRepository() {
      return new JdbcAccountRepository(...);
   }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class}) // 구체적인 설정 import
public class SystemTestConfig {
   @Bean
   public DataSource dataSource(){
      // return DataSource
   }
}

public static void main(String[] args){
   ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
   TransferService transferService = ctx.getBean(TransferService.class);
   transferService.transfer(100.00, "A123", "C456");
}
```

**여기서의 `ServiceConfig`는 구체적인 `DefaultRepositoryConfig`와 관련하여(respect to) 느슨하게(loosely) 결합되어 있습니다.** 그리고 내장된 IDE 툴링은 여전히 유용합니다. 개발자는 `RepositoryConfig` 구현 계층 타입을 쉽게 얻을 수 있습니다. 이러한 방법에서, `@Configuration` 클래스와 해당 종속성을 탐색하는 것은 인터페이스 기반의 코드를 탐색하는 일반적인 프로세스와 다르지 않습니다.

> 만약 개발자가 특정 Bean의 생성 순서에 영향을 주고 싶다면, `@Lazy`(시작이 아닌 최초 액세스시 생성) 또는 `@DependsOn`특정한 다른 Bean(현재 Bean 이전에 특정한 다른 Bean이 생성되어지게 만들고, 후자의 직접적인 의존성이 의미하는 것 이상으로(beyond)) 로써 몇몇의 Bean들을 선언하는 것을 고려해야 한다.

### Conditionally Include @Configuration Classes or @Bean Methods

**임의의 시스템 상태에 의존하여, 종종 완전한 `@Configuration` 클래스 또는 심지어 개인적인 `@Bean` 메소드를 조건적으로 활성화 또는 비활성화 시키는 것은 유용하다.** 이러한 경우의 일반적인 예는 스프링 환경에서 특정한 프로파일이 활성화된 경우에만`@Profile` 애노테이션을 사용해서 Bean을 활성화 시킬 수 있다. (자세한 내용은 Bean Definition Profiles(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-definition-profiles)를 참조)

`@Profile` 애노테이션은 실제로 더많은 융통성 있는`@Conditional` 애노테이션을 사용함으로써 실제로 구현되어질 수 있다. `@Conditional` 애노테이션은 `@Bean` 이 등록되기전에 참고해야만 하는 특정한 `org.springframework.context.annotation.Condition` 구현을 나타낸다.

`Condition` 인터페이스의 구현은 `true` 또는 `false`를 리턴하는 `matches(...)` 메소드를 제공한다. 예로들어 다음의 리스트는 실질적인 `@Profile` 을 위해 사용되어진 `Condition` 구현을 보여준다.

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
   // @Profile 애노테이션 속성을 읽는다.
   MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
   if(attrs != null) {
     	for(Object value : attrs.get("value")) {
         if(context.getEnvironment().acceptsProfiles(((String[]) value))) {
   			return true;         
         }
      }
      return false;
   }
   return true;
}
```

더 많은 정보를 알고 싶다면 `@Conditional` 의 Java document를 참고해 보자.

#### Combinig Java and XML Configuration





























