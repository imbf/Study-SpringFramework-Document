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
> 또한 `@Bean` 메소드에서, 개발자는 일반적으로 프로그램적인 JNDI 검색을 사용할 수 있습니다. Spring의 `JndiTemplate` 또는 `JndiLocatorDelegate` helpers 또는 straight JNDI `InitialContext` 사용함으로써 

























