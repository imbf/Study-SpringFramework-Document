

## 1.10 Classpath Scanning and Managed Components

Spring Container 내에서 각각의 `BeanDefinition`을 생산하는 configuration metadata를 명시하기 위해 이 **챕터(IoC Container)**의 대부분의 예제는 XML을 사용한다. 이전 챕터(Annotation based configuration) 에서는 어떻게 source 레벨의 애노테이션을 통해서 많은 configuration metadata를 제공하는지에 설명했다. 그러나 이러한 예제에서도 "base" Bean definitions은 XML 파일에 정의되어 있으며, 애노테이션은 오직 의존성 주입에만 사용합니다. 이번 챕터에는 classpath를 스캔함으로써 후보자 components를 암묵적으로 탐색하기 위한 옵션에 대해 설명합니다. **후보자 Components는 필터 기준에 일치하고 Container에 등록된 대응하는 Bean 정의가 있는 클래스 입니다.** Bean 등록을 수행하기 위해 XML을 사용하는 요구는 제거됩니다. 대신에, **개발자는 `@Component`와 같은 애노테이션과 AspectJ 타입의 표현식, Container에 등록된 Bean정의를 가진 Class들을 선택하기 위한 사용자 정의 필터 기준등을 사용할 수 있습니다.**

> Spring 3.0에서 시작으로, Spring JavaConfig 프로젝트로 부터 제공되어진 많은 기능들은 Spring Framework 핵심 중 일부분이다. 이러한 기능들은 개발자가 전통적인 XML 파일로 Bean을 설정하기 보다는 Java를 사용해서 Bean을 정의할 수 있게끔 해준다. `@Configuration`, `@Bean`, `@Import`, `@DependsOn` 과 같은 에노테이션을 참고해 보아라. 이러한 애노테이션은 어떻게 사용하는지에 대해서는 예제를 통해서 알려줄 것이다.

---

### @Bean vs @Component

#### @Bean

**개발자가 컨트롤이 불가능한 외부 라이브러리들을 Bean으로 등록**하고 싶은 경우에 사용된다.

```java
@Configuration
public class ApplicationConfig {

   // 객체를 반환하는 Method 를 만들고 @Bean 어노테이션을 부였다.
   // Bean 어노테이션에 name을 지정하지 않았음으로 Method 이름을 CamleCase로 변경한 것이 Bean id
   @Bean
   public ArrayList<String> arrray() {
      return new ArrayList<String>();
   }
}
```

**의존 관계가 필요한 경우** : Student 객체의 경우 생성자에서 ArrayList를 주입 받도록 코드를 짜 놓았다. 이럴 때에는 array() 메소드를 호출함으로써 의존성을 주입할 수 있다.

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

#### @Component

**개발자가 직접 작성한 Class를 Bean으로 등록**하기 위한 어노테이션이다.

```java
// 개발자가 사용하기 위해 직접 작성한 Class이다.
// 이러한 클래스를 Bean으로 등록하기 위해 상단에 @Component 어노테이션을 사용할 수 있다.
@Component
public class Student {
   public Student() {
      System.out.println("hi");
   }
}
```

```java
// @Component 역시 아무런 추가 정보가 없다면 Class의 이름을 Camelcase로 변경한 것이 Bean id로 사용된다.
// 하지만 @Bean과는 다르게 @Component는 name이 아닌 value를 이용하여 Bean의 이름을 지정한다.
@Component(value="myStudent")
public class Student {
   public Student() {
      S
   }
}
```

**@Bean과 @Component는 각자 선언할 수 있는 타입이 정해져있어 해당 용도외에는 컴파일러 에러를 발생시킨다.**

#### @Configuration

`@Configuration`은 스프링 IoC Conatiner에게 해당 클래스는 Bean configuration class임을 알려주는 것이다.

---

### 1.10.1 `@Component` and Further Stereotype Annotations

> **what is the stereotype ?**
>
> 고정 또는 일반적인 패턴을 따르는것; 특히 그룹 구성원이 공통적으로 보유하고 지나치게 단순화된 의견, 편견적 태도 도는 비판적 판단을 나타내는 표준화 된 정신적 그림
>
> **what is annotation ?**
>
> 관련된 결정에 대한 요약을 제공하는 설명 또는 설명을 통해 추가된 메모를 의미한다.
>
> **what is the stereotype annotation ?** @Component, @Controller, @Service, @Respository
>
> <u>클래스가 다음의 Stereotype 들 중에 하나로 annotated가 되어 있을 때, Spring은 자동적으로 Application Context에 Class들을 등록할 것입니다. 이를 통해 다른 클래스에서 클래스를 의존성 주입에 사용할 수 있게되며 이는 어플리케이션을 빌드하는데 필수적 입니다.</u>

**`@Repository`  애노테이션은 저장소(DAO)의 역할 또는 stereotype을 충족시키는 모든 클래스를 위한 marker입니다.** 이 마커의 용도중 하나로 예외 자동변환이 있습니다. 이러한 거들은 Exception Translation(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/data-access.html#orm-exception-translation)에서 묘사되어 집니다.

**Spring은 더 많은 stereotype annotations를 지원한다 : `@Component`, `@Service`, `@Controller`, `@Component`는 Spring이 관리하는 Component를 위한 일반적인 stereotype annotation들이다.** `@Respository`, `@Service`, `@Controller` 는 보다 구체적인 사용 사례(persistent service, presentation layers, ...)를 위해서 `@Component`를 구체화 시킨것들 입니다. **그러므로 개발자는 자신의 Component class들에게 `@Component` 애노테이션을 달 수 있지만 대신에 `@Repository`, `@Service`, `@Controller` 애노테이션을 달 수 있다. 그러면 class들은 tool 또는 aspect와 관련된 프로세스 처리하기 위해 더 적절하게 된다.** 예로들어, 이러한 stereotype annotation들은 pointcuts을 위한 이상적인 대상이 됩니다. `@Respository`, `@Service`, `@Controller`는 미래에 출시될 Spring Framework에 추가적인 의미를 제공합니다. 그러므로 만약 service layer에서 `@Service` 또는  `@Component` 사이에서 어떠한 애노테이션을 사용할지 선택하게 된다면, `@Service` 애노테이션이 보다 더 확실한 선택일 것이다. 비슷하게, 앞에서 언급했듯 `@Respository`는 persistence layer에서 자동 예외 번역을 위한 마커로써 이미 제공되어집니다.

---

### 1.10.2 Using Meta-annotations and Composed(구성된) Annotations

**Spring 으로부터 제공되어진 많은 애노테이션들은 meta-annotations로써 사용되어질 수 있습니다. meta-annotation은 다른 애노테이션에 적용되어질 수 있는 애노테이션이다.** 예로들어 이전에 언급되어진  `@Service` 애노테이션은 `@Component`  에 meta-annotated 되어졌다. 다음 예제에서 참고 해 보자.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // (1)
public @interface Service{
   
   // ...
}
```

1. `Component` 는 `@Service`가 `@Component`와 동일한 방식으로 처리되도록 합니다.

개발자는 meta-annotations를 "composed annotations"를 생성하기 위해 결합할 수 있다. 예로들어, Spring MVC의 `@RestController` 애노테이션은 `@Controller` 와 `@ResponseBody`로 구성되어 진다.

추가적으로, **composed 애노테이션은 사용자 정의를 허용하기 위해 meta-annoation 으로 부터의 속성을 재선언할 수 있습니다.** 이것은 meta-annoation의 속성 중 일부만 노출하려는 경우에 특히 유용합니다. 예로들어, Spring의 `@SessionScope` 애노테이션은 scope name을 `session`에 하드 코딩 하지만 여전히 `proxyMode` 사용자 정의를 허용합니다. 다음 리스트는 `SessionScope` 애노테이션의 정의를 보여줍니다.

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {
     
   /**
    * Alias for {@link Scope#proxyMode}.
    * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
    */
   @AliasFor(annotation = Scope.class)
   ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;
 
}
```

다음과 같이 proxyMode를 선언하는 것 없이 @SessionScope를 사용할 수 있다.

```java
@Service
@SessionScope
public class SessionScopedService{
   // ...
}
```

개발자는 proxyMode를 위해서 값을 오버라이드 할 수 있다. 다음 예제를 참고 해 보자.

```java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
   // ...
}
```

더 많은 정보를 얻고 싶다면, Spring Annotation Programming Model 위키 페이지를 참고 해 보자.

---

### 1.10.3 Automatically Detecting Classes and Registering Bean Definitions

Spring은 자동적으로 stereotyped 클래스를 탐색하고 해당 `BeanDefinition`을 `ApplicationContext`에 등록한다. 다음의 두개의 클래스는 자동감지에 적합하다.

```java
@Service
public class SimpleMovieLister {
   
   private MovieFinder movieFinder;
   
   public SimpleMovieLister(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
   }
}
```

```java
@Repository
public class JpaMovieFinder implements MovieFinder {
   // 명확성을 위한 구현 생략(elide)
}
```

이러한 클래스를 자동등록하고 해당 Bean을 등록하기 위해서, 개발자는 `@Configuration` 클래스에 `@ComponentScan`을 추가해야한다. `@ComponentScan`의 `backPackages` 속성은 두 클래스를 위한 공통 부모 패키지이다. ( 각 class의 부모 패키지를 포함하는 comma- 또는 semicolon- 또는 space-separated 리스트를 명시할 수 있다.)

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig {
   // ...
}
```

> 간결성을 위해서, 이전 예제는 annotation의 속성값을 사용할 수 있습니다. (즉, `@ComponentScan("org.example")`)

다음의 대안은 XML을 사용합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

> `<context:component-scan>`의 사용은 `<context:annotation-config>`의 기능을 암묵적으로 사용할 수 있게 해줍니다. `<context:component-scan>`을 사용할 때에는 `<context:annotation-config>` 요소를 포함할 필요가 없다.

> **classpath 패키지를 스캔하려면 classpath에 해당 디렉토리 항목이 있어야합니다.** JAR파일을 Ant와함께 빌드 시킬 때, JAR task의 파일 전용 스위치를 활성하 하지 마십시요. 또한 일부 환경에서 보안정책에 기초하여 classpath 디렉토리가 노출되지 않을 수 있습니다. 예로들어, JDK 1.7.0_45 이상에서 동작하는 독립형 어플리케이션( manifests 에서 'Trusted-Library' 설정이 필요하다. - https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources 를 참고해라. )
>
> JDK 9의 모듈 경로(jigsaw)에서, Spring의 classpath 스캔은 예측한대로 일반적으로 작동합니다. 그러나, Component 클래스가 `module-info` descriptor로 내보내졌는지 확인하세요. 만약 Spring이 클래스의 non-public 멤버를 호출할거라고 예상한다면, class가 'opened' 되었는지 확인하세요. ( 즉, class들은 `module-info` descriptor에서 `exports` 선언 대신에 `opens`선언을 사용합니다.  )

<u>**게다가, `AutowiredAnnotationBeanPostProcessor` 와 `CommonAnnotationBeanPostProcessor`는 Component- scan 요소를 사용할때 함축적으로 포함되어집니다. 이러한 사실은 XML에서 제공되어지는 Bean Configuration metadata 없이도 두개의 Component 들이 autodetected 와 wired 되어진다는 것을 의미한다.**</u> 

> 개발자는 `annotation-config`  속성의 값을 `false` 로 포함시킴으로써 `AutowiredAnnotationBeanPostProcessor` 과 `CommonAnnotationBeanPostProcessor` 등록을 쓰지 않을 수도 있습니다.

---

### 1.10.4 Using Filters to Customize Scanning

**기본적으로, `@Component`, `@Repository`, `@Service`, `@Controller`, `@Configuration`, `@Component` 애노테이션이 붙은 사용자 정의 애노테이션이 붙은 클래스들은 오직 탐색된 후보자 Components이다.** 그러나 개발자는 이러한 동작을 사용자 정의 필터를 적용함으로써 수정하고 확장할 수 있다. `@ComponentScan` 애노테이션에 `includeFilters` 또는 `excludeFilters` 속성으로써 필터를 추가해라. 또는 XML configuration 내의 `<context:component-scan>` 요소의 자손 요소로써 `<context:include-filter/>`  또는 `<context:exclude-filter/>` 요소를 추가해라. 각 필터의 요소는 `type` 과 `expression` 속성을 필요로 한다. 다음은 필터링 옵션에 대한 설명이다.

<img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200201104136828.png" alt="image-20200201104136828" style="zoom:50%;" />

다음의 예제는 모든 `@Respository` 애노테이션을 무시하고 "stub" respositories 를 대신해서 사용하는 configuration 예제이다.

```java
@Configuration
@ComponentSca(basePackages = "org.example",
          includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
          excludeFilters = @Filter(Repository.class))
public class AppConfig {
   // ...
}
```

다음은 동일한 XML 예제를 보여준다.

```xml
<beans>
	<context:component-scan base-package="org.example">
   	<context:include-filter type="regex" expression=".*Stub.*Repository"/>
      <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
   </context:component-scan>
</beans>
```

> **개발자는 어노테이션에 `use-default-filters="false"`를 세팅함으로써 또는 `<component-scan/>` 요소 속성에 `use-default-filters="false"`를 제공함으로써 기본 필터를 사용할 수 없게 할 수 있다.** 이렇게 하면 실제로 어노테이션 또는 `@Component` , `@Repository`, `@Service`, `@Controller`, `@RestController`, `@configuration` 메타 어노테이션이 있는 클래스의 자동 감지를 사용할 수 없습니다.

---

### 1.10.5 Defining Bean Metadata within Components

**Spring Components는 Bean definition metadata를 Container에 기여할 수 있다. 개발자는 `@Configuration`이 붙여진 클래스 내에서 Bean metadata를 정의하기위해 `@Bean` 애노테이션을 사용할 수 있다.** 다음의 예제에서 어떻게 이러한 것들을 하는지 보여준다.

```java
@Component
public class FactoryMethodComponent {
   
   @Bean
   @Qualifier("public")
   public TestBean publicInstance() {
      return new TestBean("publicInstance");
   }
   
   public void doWork(){
      // Component 메소드 구현 생략
   }
}
```

이전 클래스는 `doWork()` 메소드에 어플리케이션 특정 코드가 있는 Spring Component이다. 위 클래스는 `publicInstance()` 메소드를 참조하는 팩토리 메소드를 가진 Bean 정의에 기여한다. `@Bean` 애노테이션은 팩토리 메소드와 `@Qualifier` 애노테이션을 통한 qualifier 값과 같은 Bean 정의 properties를 식별한다. 다른 명시할 수 있는 메소드 계층의 애노테이션은 `@Scope`, `@Lazy`, 그리고 사용자 qualifier 애노테이션이다. 

> Component 초기화를 위한 역할 외에도, 개발자는 `@Autowired` 또는 `@Inject` 가 붙여져 있는 의존성 주입 지점에 `@Lazy` 애노테이션을 위치할 수 있다. 이러한 문맥에서, `@Lazy` 애노테이션은 lazy-resolution 프록시 의존성 주입을 야기시킨다.

이전에 논의한 것 처럼, Autowired 필드와 메소드는 `@Bean` 메소드의 autowiring을 위한 추가적인 지원과 함께 제공됩니다. 다음의 예제는 어떻게 처리하는지에 대해서 보여줍니다.

```java
@Component
public class FactoryMethodComponent {
   private static int i;
   
   @Bean
   @Qualifier("public")
   public TestBean publicInstance(){
      return new TestBean("publicInstance");
   }
   
   // 사용자 qualifier의 사용과 메소드 인자들의 autowiring
   @Bean
   protected TestBean protectedInstance(@Qualifier("public") TestBean spouse,
                                       @Value("#{privateInstance.age}") String country){
      TestBean tb = new TestBean("protectedInstance", 1);
      tb.setSpouse(spouse);
      tb.SetCountry(country);
      return tb;
   }
   
   @Bean
   private TestBean privateInstance(){
      return new TestBean("privateInstance", i++);
   }
   
   @Bean
   @RequestScope
   public TestBean requestScopedInstance(){
      return new TestBean("requestScopedInstance", 3);
   }
}
```

위의 예제에서 `String` 메소드 인자인 `country`를 `privateInstance` 라고 명명된 다른 Bean의 `age` property의 값에 autowire 합니다. Spring 표현 언어 요소는 `#{ <expression> }` 표기법(notatin)을 통해서 property 값을 정의합니다. `@Value` 어노테이션의 경우, 표현식 resolver는 표현식 텍스트를 분석(resolve)할 때 Bean 이름을 찾도록 미리 구성됩니다.

Spring Framework 4.3부터, **현재 Bean 생성을 일으키는 requesting 주입 지접에 접근하기 위해 `InjectionPoint` 유형(또는 더 구체적인 서브클래스인 `DependencyDescriptor`)의 팩토리 메소드 매개 변수를 선언 선언할 수도 있습니다.** 이는 기존 인스턴스의 주입이 아닌 실제 Bean 인스턴스 생성에만 적용된다는 것을 명심해야 한다. **결과적으로, 이 기능은 프로토 타입 범위의 Bean에 가장 적합합니다.** 다른 범위의 경우, 팩토리 메소드는 주어진 범위에서 새로운 Bean 인스턴스 생성을 유발하는 주입 지점만 볼 수 있습니다. (예로들어, lazy Singleton Bean의 생성을 유발하는 종속성). 이러한 상황에서 개발자는 제공된 의존성 주입 메타데이터를 semantic care와 함께 사용할 수 있습니다. 다음의 예제에서 어떻게 `InjectionPoint`를 사용하는지 보여줍니다.

```java
@Component
public class FactoryMethodComponent {
   
   @Bean @Scope("prototype")
   public TestBean prototypeInstance(InjectionPoint injectionPoint){
      return new TestBean("prototypeInstance for " + injectionPoint.getMember());
   }
}
```

일반적인 Spring `Component` 의 `@Bean` 메소드는 Spring `@Configuration` 클래스내의 해당(counterpart) 메소드와 다르게 처리됩니다. **`@Component` 클래스들은 <u>GGLIB</u>로 향상(enhance)되지 않아 메소드와 필드의 호출을 가로막지(intercept) 않는다는 점이 차이점이다.** **GGLIB 프록싱은 `@Configuration` 클래스의 `@Bean` 메소드 내에서 필드 또는 메소드를 호출하여 collaborating 객체에 대한 Bean metadata 참조를 작성하는 수단이다.** 이러한 메소드들은 일반적인 Java semantics으로 호출되는 것이 아니라 `@Bean` 메소드에 대한 프로그래밍 호출을 통해 다른 Bean을 참조 할 때도 일반적인 lifecycle 관리 및 Spring Bean 프록시를 제공하기 위해 컨테이너를 통과합니다(go through). 이에 반해서(In contrast), 일반적인 `@Component`  클래스의 `@Bean` 메소드 내의 필드 또는 메소드가 호출되는 것은 특별한 GGLIB 프로세싱 또는 다른 제약사항의 적용 없이 표준 Java Semantics를 가진다

> **개발자가 `@Bean` 메소드를 `static`으로써 선언하면, 메소드를 포함하는 configuration class를 인스턴스로써 생성하지 않고 호출되어질 것이다. 이러한 것들은 특히 post-processor Bean( ex. `BeanFactoryPostProcessor` 또는 `BeanPostProcessor` 타입)을 정의할 때 의미가 있다.** 이러한 Bean들은 Container lifecycle 에서 일찍 초기화가 되고, 해당 시점에서 configuration의 다른 부분을 유발시키지 않아야 하기 때문이다.
>
> 정적인 `@Bean` 메소드들을 호출하는 것은 기술적인 한계로 인해  `@Configuration` 클래스(이전 섹션에서 설명) 내에서 조차도 Container에 의해서 가로막히지 않습니다. GGLIB 서브클래스는 non-static 메소드들만 오버라이드 할 수 있다. **결과적으로, `@Bean` 메소드들의 직접적인 호출은 표준 Java semantics를 가지고, 독립 인스턴스가 팩토리 메소드 자체에서 직접 리턴됩니다.**
>
> `@Bean` 메소드들의 Java 언어 가시성은 Spring Container Bean 정의 결과에 즉각적인 영향을 미치지 않습니다. 개발자는 팩토리 메소드를 non-`@Confiugration` 클래스에 선언할 수 있고 정적 메소드에 대해서도 팩토리 메소드를 자유롭게 선언할 수 있다. 그러나 `@Configuration` 클래스 내의 일반적인 `@Bean` 메소드들은 오버라이드가 필요하다. - 즉, 그들은 `private` 또는 `final` 로써 선언되면 안된다.
>
> `@Bean` 메소드들은 주어진 component 또는 configuration 클래스의 base 클래스 에서 발견되어질 뿐만 아니라 component 또는 configuration 클래스로부터 구현되어진 interface에 선언되어진 Java 8 기본 메소드들에서 발견됩니다. Spring 4.2 부터, 이러한 사실은 Java 8 기본 메소드들을 통해서 다중 상속이 가능한 복잡한 configuration 구성에서 많은 융통성을 가질 수 있게 합니다.
>
> 결과적으로, 단일 클래스는 런타임시에 이용가능한 의존성에 따라 사용할 여러 팩토리 메소드 배열로서 동일한 Bean에 대해 여러 `@Bean` 메소드를 보유 할 수 있습니다. 이러한 사실은 다른 configuration 시나리오에서 가장 탐욕적인 생성자 또는 팩토리 메소드를 고르는 것과 같은 알고리즘 입니다. 구성 시간에 만족할 수 있는 의존성 수가 가장 많은 변형(variant)이 선택됩니다. Container가 여러개의 `@Autowired` 생성자 중에서 선택하는 방식과 유사하게끔

---

### 1.10.6 Naming Autodetected Components 

**Scanning process의 부분으로써 Component가 자동 탐색될 때, Component의 Bean name은 해당 Scanner에 알려진 `BeanNameGenerator` 전략에 의해서 생성되어진다.** **기본적으로, name `value`를 포함하는 Spring stereotype 애노테이션(`@Component`, `@Repository`, `@Service`, `@Controller`) 해당 Bean 정의에 해당 name을 제공합니다.**

만약 이러한 애노테이션들이 name `value`를 가지고 있지 않거나, 모든 탐색된 Component(사용자 필터에 의해서 발견된 것)에 대한 name `value` 값을 포함하고 있지 않은 경우, 기본 Bean name 생성기는 대문자로 규정되지 않는 non-qualified class name(camelCase)을 리턴할 것이다. 예로들어, 다음의 component 클래스가 탐색되었다면, name은 `myMovieLister` 과 `movieFinderImpl`이 될것이다.

```java
@Service("myMovieLister")
public class SimpleMovieLister {
   // ...
}
```

```java
@Respository
public class MovieFinderImpl implements MovieFinder {
   // ...
}
```

개발자가 기본적이 Bean-naming 전략에 대해 의존하는 것을 원치 않을 경우, 사용자 정의 Bean-naming strategy를 제공할 수 있다. 첫번째로, `BeanNamingGenerator` 인터페이스를 구현하고, 인수없는 기본 생성자를 포함해야 합니다. 그 다음에(Then), Scanner를 설정할 때 fully qualified 클래스 name을 제공해야 합니다. 다음 예제에서 annotation과 Bean definition에 대한 예제를 보여 줄 것입니다.

> 만약 동일한 non-qualified 클래스 name을 가진 여러 자동 탐색된 component 때문에 naming 충돌이 발생한다면(동일한 name을 가지지만 다른 패키지에 존재하는 클레스들), 생성된 Bean name을 위해서 개발자는 완전히 qualified된 클래스 name을 기본으로하는 `BeanNameGenerator`를 설정해야 할 필요가 있다. Spring Framework 5.2.3 부터, `org.springframework.context.annotation`에 위치한 `FullyQualifiedAnnotationBeanNameGenerator`는 이러한 목적에서 사용될 수 있다.

```java
@Configuration
@ComponentScan(basePackages = "org.example", nameGerator = MyNameGerator.class)
public class AppConfig {
   // ...
}
```

```xml
<beans>
	<context:component-scan base-package="org.example" name-generator="org.example.MyNameGenerator"/>
</beans>
```

**일반적으로 다른 구성 요소가 명시적으로 참조할 때마 주석으로 name을 정의하는 것을 고려해라. 반면에, 자동 생성되는 name은 Container가 wiring을 담당할 때 적합하다.**

---

### 1.10.7 Providing a Scope for Autodetected Components

**일반적으로 Spring이 관리하는 component와 같이, 자동 탐색된 Component를 위한 가장 일반적이고 기본 범위는 `Singleton` 이다.** 그러나, 때때로 개발자는 `@Scope` annotation에 명시되어질 수 있는 또 다른 scope가 필요하다. 개발자는 애노테이션안에 scope의 name을 제공할 수 있다. 다음 예제에서 보여준다.

```java
@Scope("prototype") // scope의 name을 제공하는 예제이다.
@Repository
public class MovieFinderImpl implements MovieFinder {
   // ...
}
```

> <u>**`@Scope` 애노테이션은 오직 구체적인 Bean class(`component` 애노테이션이 붙은) 또는 팩토리 메소드 (`@Bean` 메소드 )에서만 검사됩니다.**</u> XML Bean 정의와는 대조적으로, Bean 정의 상속에 대한 개념은 없으며, 클래스 레벨의 상속 계층은 metadata 목적과 관련이 없습니다.

구체적으로 Spring Context 내의 web-specific 범위인 request 또는 session에 대한 정보는 Request, Session, Application, WebSocket Scopes를 참고해 보아라. **이러한 범위를 위해 미리 빌드된 애노테이션과 마찬가지로, 개발자는 자신의 scoping 애노테이션을 Spring meta-annotation 접근을 사용해서 구성할 수 있습니다.** 예로들어, `@Scope("prototype")` 으로 메타 주석이 달린 사용자 정의 주석은 사용자 정의 프록시 방식(mode)을 선언할 수도 있습니다.

> **애노테이션 기반의 접근법에 의존하는 것 보다 범위 해결을 위한 사용자 정의 전략을 제공하기 위해, 개발자는 `ScopeMetadataResolver` 인터페이스를 구현할 수 있습니다. 인자가 없는 기본 생성자를 포함해야 합니다.** 그러면 Scanner를 설정할 때 완전히 qualified된 클래스를 제공할 수 있습니다. 다음의 예제는 애노테이션과 Bean 정의에 대해서 보여줍니다.

```java
// @Configuration이 붙은 설정 클래스에 @ComponentScan 애노테이션을 붙여야 한다.

@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class) 
public class AppConfig{
   // ...
}
```

```xml
<beans>
	<context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>
```

특정한 non-Singleton 범위를 사용할 때, 이러한 것들은 범위가 정해진 객체를 위한 프록시들을 생성하는데 필수적이다. 이러한 이유는 Scope Beans as Dependencies 에서 묘사되어 있다. 이러한 목적에서, 범위가 지정된 프록시 속성은 Compnent-scan 요소로 이용 가능하다. 3가지의 가능한 값으로는 : `no`, `interfaces`, `targetClass`가 있다. 예로들어, 다음의 configuration은 표준 JDK dynamic proxies를 일으킨다.

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
   // ...
}
```

```xml
<beans>
	<context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

---

### 1.10.8 Providing Qualifier Metadata with Annotations

`@Qualifier` 애노테이션은 Fine-tuning Annotation-based Autowiring with Qualifiers 에서 논의 되었다. 해당 섹션의 예제에서는 `@Qualifier` 애노테이션과 autowire 후보자를 검색할 때 더 섬세한 제어를 제공하기 위한 사용자 정의 qualifier 애노테이션의 사용에 대해서 설명하였습니다. 이러한 예제들은 XML Bean 정의 기반이였기 때문에, qualifier metadata는 XML의 Bean 요소의 `qualifier` 또는 `meta` 자식요소를 사용함으로써 후보자 Bean 정의를 지원해 주었다. **Component의 자동감지를 위해 classpath 스캔에 의존하는 경우, 개발자는 후보자 클래스에서 타입 계층의 어노테이션과 함께 qualifier metadata를 제공할 수 있습니다.** 다음의 3가지 예제는 이러한 경우를 설명합니다.

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog{
   // ...
}
```

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

> **대부분의 애노테이션 기반의 대안과 마찬가지로, <u>annotation metadata는 클래스 정의 자체에 바인딩</u> 되어 있다는 것을 명심해야 한다. 반면에 XML의 사용은 자신의 qualifier metadata내에서 변형을 제공하기 위해 동일한 타입의 여러 Bean을 허용합니다.** metadata는 각 class 마다가 아니라 instance마다 제공되어지기 때문입니다.
>
> **인스턴스마다 동일한 타입의 여러 Bean이 제공되어 지면 qualifier metadata내에서 변형을 제공한다.**

---

### 1.10.9 Generating an Index of Candidate Components

**classpath scanning은 매우 빠르지만, 컴파일 시간에 후보자의 정적 리스트를 생성함으로써 큰 어플리케이션의 시작능력을 향상 시킬 수 있습니다.** Component-Scan의 목표가 되는 모든 모듈들은 이러한 방법을 무조건 사용해야 합니다.

> 기존 `@ComponentScan` 또는 `<context:component-scan>` 지시문은 특정 패키지에서 후보를 스캔하도록 context를 요청하는 것입니다. **`ApplicationContext`가 이러한 index를 검색할 때, `ApplicationContext`는 classpath를 스캔하는 것보다 index를 사용한다.**

**index를 생성하기 위해, Component 스캔 지시문(directives)의 target인 component를 포함하는 각 모듈에 의존성을 추가하십시요.** 다음의 예제는 어떻게 Maven에서 사용하는지에 대해서 보여준다.

```xml
<dependencies>
	<dependency>
   	<groupId>org.springframework</groupId>
      <artifactId>spring-context-indexer</artifactId>
      <version>5.2.3.RELEASE</version>
      <optional>true</optional>
   </dependency>
</dependencies
```

Gradle의 4.5버전과 이전에는, 의존성은 `compileOnly` 설정에서 선언되어야만 했다. 다음 예제에서 보여준다.

```groovy
dependencies {
   compileOnly "org.springframework:spring-context-indexer:5.2.3.RELEASE"
}
```

> Gradle 4.6버전 이상부터, 의존성은 `annotationProcessor` 설정에서 정의되어야 한다. 다음의 예제에서 보여준다.

```groovy
dependencies {
   annotationProcessor "org.springframework:spring-context-indexer:{spring-version}"
}
```

**이 프로세스는 jar 파일에 포함된 META-INF/spring.components 파일을 생성합니다.**

> **IDE에서 이러한 방식으로 작업할때, 후보자 component가 업데이트 될 때 index가 최신상태가 되도록 `spring-context-indexer`를 annotation processor로써 등록해야한다.**

> **META-INF/spring.components 가 classpath에서 발견되어질 때 index는 자동적으로 활성화됩니다.** 만약 index가 여러 라이브라리를 위해서 부분적으로 사용가능하고 전체 어플리케이션을 위해서는 빌드 되어지지 않으려면, 개발자는 일반적인 classpath 배열로 대체할 수 있습니다(인덱스가 전혀 없는 것처럼). 이러한 기능을 사용하기 위해서는 classpath의 root의 `spring.properties`파일 또는 시스템 property에서 `spring.index.ignore`을 `true`로 설정 해야 합니다.