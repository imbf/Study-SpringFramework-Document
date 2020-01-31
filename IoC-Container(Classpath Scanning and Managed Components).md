

## 1.10 Classpath Scanning and Managed Components

Spring Container 내에서 각각의 `BeanDefinition`을 생산하는 configuration metadata를 명시하기 위해 이 **챕터(IoC Container)**의 대부분의 예제는 XML을 사용한다. 이전 챕터(Annotation based configuration) 에서는 어떻게 source 레벨의 애노테이션을 통해서 많은 configuration metadata를 제공하는지에 설명했다. 그러나 이러한 예제에서도 "base" Bean definitions은 XML 파일에 정의되어 있으며, 애노테이션은 오직 의존성 주입에만 사용합니다. 이번 챕터에는 classpath를 스캔함으로써 후보자 components를 암묵적으로 검사하기 위한 옵션에 대해 설명합니다. **후보자 Components는 필터 기준에 일치하고 Container에 등록된 대응하는 Bean 정의가 있는 클래스 입니다.** Bean 등록을 수행하기 위해 XML을 사용하는 요구는 제거됩니다. 대신에, 개발자는 `@Component`와 같은 애노테이션과 AspectJ 타입의 표현식, Container에 등록된 Bean정의를 가진 Class들을 선택하기 위한 사용자 정의 필터 기준등을 사용할 수 있습니다.

> Spring 3.0에서 시작으로, Spring JavaConfig 프로젝트로 부터 제공되어진 많은 기능들은 Spring Framework 핵심 중 일부분이다. 이러한 기능들은 개발자가 전통적인 XML 파일로 Bean을 설정하기 보다는 Java를 사용해서 Bean을 정의할 수 있게끔 해준다. `@Configuration`, `@Bean`, `@Import`, `@DependsOn` 과 같은 에노테이션을 참고해 보아라. 이러한 애노테이션은 어떻게 사용하는지에 대해서는 예제를 통해서 알려줄 것이다.

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
> 클래스가 다음의 Stereotype 들 중에 하나로 annotated가 되어 있을 때, Spring은 자동적으로 Application Context에 Class들을 등록할 것입니다. 이를 통해 다른 클래스에서 클래스를 의존성 주입에 사용할 수 있게되며 이는 어플리케이션을 빌드하는데 필수적 입니다.

**`@Repository`  애노테이션은 저장소(DAO)의 역할 또는 stereotype을 충족시키는 모든 클래스를 위한 마커입니다.** 이 마커의 용도중 하나로 예외 자동변환이 있습니다. 이러한 거들은 Exception Translation에서 묘사되어 집니다.

Spring은 더 많은 stereotype annotations를 지원한다 : `@Component`, `@Service`, `@Controller`, `@Component`는 Spring이 관리하는 Component를 위한 일반적인 stereotype annotation들이다. `@Respository`, `@Service`, `@Controller` 는 보다 구체적인 사용 사례(persistent service, presentation layers, ...)를 위해서 `@Component`를 구체화 시킨것들 입니다. 그러므로 개발자는 자신의 Component class들에게 `@Component` 애노테이션을 달 수 있지만 대신에 `@Repository`, `@Service`, `@Controller` 애노테이션을 달 수 있다. 그러면 class들은 tool 또는 aspect와 관련된 프로세스 처리하기 위해 더 적절하게 된다. 예로들어, 이러한 stereotype annotation들은 pointcuts을 위한 이상적인 대상이 됩니다. `@Respository`, `@Service`, `@Controller`는 미래에 출시될 Spring Framework에 추가적인 의미를 제공합니다. 그러므로 만약 service layer에서 `@Service` 또는  `@Component` 사이에서 어떠한 애노테이션을 사용할지 선택하게 된다면, `@Service` 애노테이션이 보다 더 확실한 선택일 것이다. 비슷하게, 앞에서 언급했듯 `@Respository`는 persistence layer에서 자동 예외 번역을 위한 마커로써 이미 제공되어집니다.

---

### 1.10.2 Using Meta-annotations and Composed(구성된) Annotations

**Spring 으로부터 제공되어진 많은 애노테이션들은 meta-annotations로써 사용되어질 수 있습니다. meta-annotation은 다른 애노테이션에 적용되어질 수 있는 에노테이션이다.** 예로들어 이전에 언급되어진  `@Service` 애노테이션은 `@Component`  에 meta-annotated 되어졌다. 다음 예제에서 참고 해 보자.

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

> 







