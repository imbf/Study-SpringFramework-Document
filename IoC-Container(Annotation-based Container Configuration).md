## 1.9 Annotation-based Container Configuration

> #### Spring 설정에 XML 보다 annotation이 더 좋나?
>
> annotation-based configuration 의 소개에서는 XML 보다 Annotation 기반의 configuration이 더 좋느냐 하는 질물을 유발하였습니다. 우리는 "때에 따라 다르지"라고 짧게 대답할 수 있습니다. 좀 더 길게 답하자면 각각의 접근법은 장점과 단점을 보유하고 있습니다. 보통, 어떠한 전략이 더 적합한지 결정하는 것은 개발자에게 달려(be up to)있습니다. **개발자가 정의하는 방식에 따라서, annotation들은 선언에 많은 수의 context를 제공하며 짧고 간결한 설정을 제공합니다.** **반면에 XML은 소스코드를 조작하거나 다시 컴파일 할 필요없이 components 간에 연결짓는데 더 훌륭합니다.** <u>일부 개발자들은 소스에 가까운 wiring을 선호하는 반면에 다른 사람들은 주석이 달린 클래스가 더이상 POJO가 아니며, 구성이 분산되어 제어하기 더 어렵다고 주장합니다.</u>
>
> 어느 선택을 하던간, Spring은 두가지의 스타일과 심지어는 같이 섞어논 스타일까지도 수용할(accommodate) 수 있습니다. Spring의 JavaConfig option 을 통해서 지적할만한 가치가 있습니다. Spring은 annotations이 비침투 방법으로 target 요소 소스코드를 건드리지 않고 tooling 측면에서 사용되어질 수 있도록 한다. 모든 설정 스타일은 Spring Tool Suite로부터 지원되어진다.

**XML 설정의 대안은 annotation-based 설정에 의해서 제공되어진다. annotation-based configuration은 angle-bracket 선언 대신에 구성요소를 wiring하기 위해 bytecode metadata에 의존한다.** **Bean wiring을 설명하기위해 XML을 사용하는 것 대신에, 개발자는 관련된 클래스나, 메소드, 필드 선언에서 annotation을 사용함으로써 component class 자체에서 설정을 한다.** Example: The `RequiredAnnotationBeanPostProcessor` 에서 언급했듯이, `BeanPostProcessor`을 annotation과 함께 사용하는 것은 Spring IoC Container를 확장하는 가장 일반적인 방법이다. 예로들어, Spring 2.0에서는 `@Required` 에노테이션과 함께 필요한 properties을 시행할 수 있는 가능성이 도입되었습니다. Spring 2.5 에서는 스프링 의존성 주입을 운영하기 위해서 동일한 일반적인 접근방식을 따를 수 있도록 하였습니다. 근본적으로, `@Autowired` annotation은 Autowiring Collaborator에 설명된것처럼 같은 기능을 제공하고 더 섬세한 제어와 더 넓은 적용성을 가집니다. Spring 2.5는 또한 `@PostConstruct` 와 `@PreDestory` 같은 JSR-250 annotation을 지원을 추가하였습니다. Spring 3.0에서는 `@Inject` 와 `@Named` 같은 `javax.inject` 패키지에 포함된 JSR-330 annotations을 추가 지원합니다. 이러한 annotation에 관한 더 자세한 정보는 relevant section 에서  찾을 수 있습니다.

> **Annotation injection은 XML injection 이전에 수행되어진다. 그러므로, XML configuration은 양쪽 접근을 통해 wired 된 properties에 대한 annotation을 오버라이드 합니다.**

**언제나 그렇듯이, 개별 Bean 정의로 Annotation을 등록할 수 있지만, 다음 태그를 포함하여 암시적으로 등록 할 수도 있습니다.(`context` namespace 포함에 주목해야 한다.)**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

암시적으로 등록되어진 post-processors는 `AutowiredAnnotationBeanPostProcessor`, `CommonAnnotationBeanPostProcessor`, `PersistenceAnnotationBeanPostProcessor`, `RequiredAnnotationBeanPostProcessor`들을 포함한다.

> `<context:annotation-config/>` 는 오직 정의된 같은 applicationcontext 내의 Bean에서 annotation만 찾습니다. 즉, `DispatcherServlet`의 `WebApplicationConext`에 `<context:annotation-config/>`를 넣으면  서비스가 아닌 컨트롤러의 `@Autowired` Bean을 검사합니다. 자세한 정보는 DispatcherServlet을 참조하십시요.

---

### 1.9.1 @Required

`@Required` annotation은 Bean Property의 setter methods에 적용됩니다. 다음 예제를 참고 해 봅시다.

```java
public class SimpleMovieLister{
   private MovieFinder movieFinder;
   
   @Required
   public void setMovieFinder(MovieFinder movieFinder){
      this.movieFinder = movieFinder;
   }
   
   // ...
}
```

**이 annotation은 영향을 받는 Bean property를 configuration time에서 Bean definition의 명확한 property value를 통해 또는 autowiring을 통해서 populated 함을 나타냅니다.** 이 Container는 영향을 받은 Bean의 property가 populated 되지 않으면 예외를 던집니다. 이러한 프로세스는 나중에(later on) 명백한 실패를 허용해, `NullPointerException` 인스턴스를 피합니다. Spring은 개발자가 Bean class 자체에 assertion 를 넣을 것을 권고한다. 다음과 같이 진행하는 것은 필수 reference 와 값들을 Container 외부에서 class를 사용할 때 조차도 적용됩니다..

> `@Required` annotation은 공식적으로 Spirng Framework 5.1에서 필수 설정(또는 Bean property setter methods와 함께(along with) InitializingBean.afterPropertiesSet() 의 사용자 구현)에 생성자 주입을 사용하는 것을 찬성하여 폐지되었습니다.

---

### 1.9.2 Using `@Autowired`

> JSR 330's `@Inject` 애노테이션은 스프링의 `@Autowired` 애노테이션을 대신에 사용되었다.

개발자는 `@Autowired` 애노테이션은 생성자에 다음 예제에서 보여주는 것처럼 적용할 수 있다.

```java
public class MovieRecommender{
   private final CustomerPreferenceDao customerPreferenceDao;
   
   @Autowired
   public MovieRecommender(CustomerPreferenceDao customerPreferenceDao){
      this.customerPreferenceDao = customerPreferenceDao;
   }
   
   //...
}
```

> **Spring Framework 4.3 부터, `@Autowired` annotation(생성자에 붙여진)은 target Bean이 시작하기 위해 오직 하나의 생성자로 구성되어져 있다면 `@Autowired`는 더이상 필요하지 않게 되었습니다.** 그러나 만약 여러개의 생성자가 이용가능 하다면, 최소한 Container에게 명령하기 위해서 `@Autowired`가 최소한 하나에는 붙여져 있어야 한다.

**개발자는 `@Autowired` annotation을 전통적인 setter 메소드에 다음 예제와 같이 적용할 수 있다.**

```java
public class SimplemovieLister{
   
   private MovieFinder movieFinder;
   
   @Autowired
   public void setMovieFinder(MovieFinder movieFinder){
      this.movieFinder = movieFider;
   }
   
   // ...
}
```

**개발자는 `@Autowired` annotation을 임의의 이름을 가진 메소드와 여러개의 인자를 가진 메소드에 다음 예제에서 보여주는 것 처럼 적용할 수 있습니다.**

```java
public class MovieRecommender{
   
   private MovieCatalog movieCatalog;
   
   private CustomerPreferenceDao customerPreferenceDao;
   
   @Autowired
   public void prepare(MovieCatalog movieCatalog, CustomerPreferenceDao customerPreferenceDao){
      this.movieCatalog = movieCatalog;
      this.customerPreferenceDao = customerPreferenceDao;
   }
   
   //...
}
```

**개발자는 `@Autowired`를 필드와, 심지어는 생성자와 함께 필드를 섞어서 다음 예제에서 보여주는 것처럼 적용할 수 있습니다.**

```java
public class MovieRecommender{
   
   private final CustomerPreferenceDao customerPreferenceDao;
   
   @Autowired
   private MovieCatalog movieCatalog;
   
   @Autowired
   public MovieRecommender(CustomerPreferenceDao customerPreferenceDao){
      this.customerPreferenceDao = customerPreferenceDao;
   }
   
   //...
}
```

> target components(ex. `MovieCatalog` or `CustomerPreferenceDao`)가 `@Autowired`-annotaed 주입 지점을 위해 사용하는 type에 의해서 일관되게 일관되게 선언이 되어있는지 확인해야한다. 그렇지 않으면, 주입은 "no type match found" 에러 때문에 런타임시에 실패할 것이다.
>
> classpath scanning을 통해서 찾은 XML에 의해서 정의된 Bean 또는 Components class를 위해, Container는 보통 구체적인 타입을 미리(up front) 알고 있습니다. 그러나, `@Bean` 팩토리 메소드의 경우, 개발자는 선언된 리턴타입이 충분히  나타나는지 확인해야 합니다. 여러 인터페이스를 구현하는 components를 위해서 또는 잠재적으로 그들의 구현 타입에 의해서 언급되어지는 components를 위해서, factory method에 가장 명시적인 리턴 타입을 선언할 것을 고려해야한다. (최소한 Bean을 참조하는 주입 지점에 의해 요구되어지는 정도로 명시해야한다.)

**개발자는`ApplicationContext`에 `@Autowired` 을 해당 타입의 배열을 예상하는 필드 또는 메소드에 추가함으로써 특정 타입의 모든 Bean을 Spring이 제공하도록 지시할 수 있다.**

다음 예를 참고해 보자.

```java
public class MovieRecommender{
   @Autowired
   private MovieCatalog[] movieCatalogs;
   
   // ...
}
```

다음 예제와 같이 타입이 정해진 컬렉션에 똑같이 적용할 수 있다.

```java
public class MovieRecommender {
   
   private Set<MovieCatalog> movieCatalogs;
   
   @Autowired
   public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs){
      this.movieCatalogs = movieCatalogs;
   }
   
   // ...
}
```

> 개발자가 리스트나 배열 안에서 특정한 순서대로 정리되어지길 원한다면, target Bean은 `org.springframework.core.Ordered` 인터페이스를 구현할 수 있거나 또는 `@Order` 또는 표준 `@Priority` 애노테이션을 사용할 수 있습니다. 그렇지 않으면, 그들의 순서는 Container의 target Bean 정의에 대응하는 등록순서를 따를것이다.
>
> 개발자는 개별 Bean 정의에 대해 target class 레벨 및 @Bean 메소드에서 `@Order` 애노테이션을 선언할 수 있다.(동일한 Bean 클래스를 사용하는 여러개의 정의의 경우) `@Order` 값은 의존성 지점에서 우선순위에 영향을 미칠것이다. 그러나 `@Order` 값이 singleton 시작 순서에는 영향을 미치지 않는다는 것을 알아야 한다. singleton 시작 순서는 의존성 관계와 `@DependsOn` 선언에 의해서 결정되어지는 직교 관계이다.
>
> 표준 `javax.annotation.Priority` 애노테이션은 메소드에서 선언 되어질 수 없기 때문에  `@Bean` 계층에서 이용이 불가능하다. 이러한 의미는 각 타입을 위한 단일 Bean에서 `@Order` 값과 `@Primary` 의 조화를 통해 모델링 되어질 수 있습니다.

모든 타입의 `Map` 인스턴스는 예상되는 key값이 String인 경우 autowired 되어질 수 있습니다. map value는 예측되는 타입의 모든 Bean을 포함하고, key는 대응되는 Bean name을 포함한다. 다음 예제에서 보여준다.

```java
public class movieRecommender{
   
   private Map<String, MovieCatalog> movieCatalogs;
   
   @Autowired
   public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
      this.movieCatalogs = movieCatalogs;
   }
   
   // ...
}
```

**기본적으로, 주어진 주입 포인트에 대하여 어떠한 matching 후보자 Bean도 이용할 수가 없을 때 autowiring은 실패한다. 선언된 배열, 컬렉션, 맵 경우에, 최소한 하나의 matching 요소가 예상 되어야 한다.**

기본 동작은 annotated 메소드 와 필드를 필수 종속성을 나타내는 것으로 처리하는 것입니다. 개발자는 이러한 동작을 다음예 에서 설명되어지는 것처럼 바꿀 수 있습니다. 개발자는 프레임워크가 만족하지 못한 주입 포인트를 필요하지 않다고 표시함으로써 넘어가게 할 수 있습니다.(`@Autowired` 내의 `required`속성을 `false`로 세팅함으로써)

```java
public class simpleMovieLister {
   
   private MovieFinder movieFinder;
   
   @Autowired(required = false)
   public void setMovieFinder(MovieFinder movieFinder){
      this.movieFinder = movieFinder;
   }
   
   // ...
   
}
```

**필수가 아닌 메소드들은 만약 의존성이 만족되지 못하면(여러개의 인자를 가지고 있는경우 이것의 하나라도) 전부다 호출하지 못하도록 한다.  필수가 아닌 필드들은 이러한 경우에 전혀 채워지지(populate) 않음으로, 기본값이 올바른 장소에 남아있게 됩니다.**

 **`@Autowired` 의  `required` 속성이 잠재적으로 여러개의 생성자를 다룰 Spring 생성자 resolution 알고리즘 때문에 다소 다른 의미를 가지기 때문에 주입된 생성자와 factory method 인자는 특별한 경우이다**. 생성자와 팩토리 메소드 인자들은 기본적으로 필수적이지만 일치하는 Bean을 사용할 수 없는 경우 빈 인스턴스로 확인되는 다중 요소 삽입 지점(배열, 콜렉션, 맵)과 같은 단일 생성자 시나리오에서 몇 가지 특수 규칙이 있습니다. **생성자와 팩토리 메소드 인자들은 모든 의존성이 유일한 다수의 인자 생성자에 선언되어지는 흔한 구현 패턴을 허용합니다.** - 예로들어, `@Autowired` 애노테이션 없이 단일 public 생성자로써 선언되는 경우이다.

> 주어진 Bean class에서 오직 한개의 생성자가 `@Autowired`의 `required` 속성을 `true`로써 선언한다. Spring Bean으로써 사용되어질 때 생성자가 autowire임을 나타냅니다. 게다가 `required` 속성이 `true`로 세팅 되어져 있다면, 오직 하나의 생성자가 @Autowired 함께 annotated 될 수 있습니다. 필수적이지 않은 다중 생성자가 annotation을 선언한다면 , 그들은 autowiring을 위한 후보자로써 고려되어질 것이다. Spring Container에서 Bean을 매칭함으로써 의존성을 만족시키는 생성자가 선택되어질것이다. 만약 어떠한 후보자도 선택되어지지 않는다면, 상위/기본 생성자(만약 존재한다면)가 사용되어질 것이다. 만약 클래스가 오직 하나의 생성자를 시작하기위해 선언한다면, 이것은 annotation이 붙지 않아도 항상 사용되어질 것다. annotation이 달린 생성자는 public일 필요가 없습니다.
>
> **`@Autowired`의 `required`속성은 setter 메소드의 더이상 사용되지 않는 `@Required` 애노테이션 보다 더 권장됩니다.** `required` 속성을 `false`로 설정하는 것은 property가 autowiring 목적을 위해 필요하지 않다는것을 의미하고, autowired를 할 수 없는 경우 property는 무시됩니다. 다른 말로, `@Required` 는 Container에 의해 지원되는 모든 방법에 의해서 property를 설정한다는 점에서 강력합니다. 만약 어떠한 값도 설정되어 있지 않다면, 그에 알맞는 에러가 발생되어질 것이다.

대안적으로, 개발자는 특정 의존성의 필요하지 않는 특성을 Java 8의 `java.util.Optional` 을 통해서 표현할 수 있다. 다음 예제에서 보여준다.

```java
public class SimpleMovieLister{
   
   @Autowired
   public void setMovieFinder(Optional<MovieFinder> movieFinder){
      // ...
   }
}
```

Spring Framework 5.0 부터, 개발자는 `@Nullable` 애노테이션 또는 leverage Kotlin builtin null-safety support를 사용할 수 있습니다. (패키지의 모든 종류에서) 

```java
public class SimplemovieLister{
   
   @Autowired
   public void setMovieFinder(@Nullable MovieFinder movieFinder){
      // ...
   }
}
```

개발자는 의존성을 해결하기 위해서 잘 알려진 인터페이스를 위해 `@Autowired`를 사용할 수 있습니다. (ex. `BeanFactory`, `ApplicationContext`, `Environment`, `ResourceLoader`, `ApplicationEventPublisher`, `MessageSource`) 이러한 인터페이스 와 확장된 인터페이스(`ConfigurableApplicationContext` or `ResourcePatternResolver`)들은 어느 특별한 설정이 필요없이 자동적으로 의존성이 해결됩니다. 다음 예제는 `ApplicationContext` 객체를 autowire한 예제입니다.

```java
public class MovieRecommender{
   
   @Autowired
   private ApplicationContext context;
   
   public MovieRecommender(){
      
   }
   
   // ...
}
```

`@Autowired`, `@Inject`, `@Value`, `@Resource` 애노테이션은 Spring `BeanPostProcessor` 구현에 의해서 다루어진다. 개발자는 이러한 애노테이션을 사용자 정의 `BeanPostProcessor` 또는 `BeanFactoryPostProcessor` 타입 내에 적용할 수 없음을 의미한다. 이러한 타입의 애노테이션은 XML 또는 Spring `@Bean` 메소드를 사용함으로써 명백히 wired up 시켜야 한다.

---

### 1.9.3 Fine-tuning Annotation-based Autowiring with `@Primary`  

(`@Primary`를 사용한 어노테이션 기반의 Autowiring 미세 조정)

타입에 의한 autowiring은 여러 개의 후보자를 야기시키기 때문에, 종종 선택 프로세스 동안에 제어에 관심을 기울일 필요가 있다. 이러한 목표를 달성하기 위한 한가지 방법은 Spring의 `@Primary` 어노테이션이다. **여러 Bean이 단일 값 의존성에 autowired가 되어질 후보자인 경우, `@Primary`는 특정한 Bean을 preference로 지정해야 함을 나타낸다.** 만약 여러개의 후보자 사이에 정확히 하나의 가장 우선순위 Bean이 존재한다면, 이 Bean은 Autowired 값이 된다.

`firstMovieCatalog`를 `MovieCatalog`의 최우선 순위로써 정의하는 다음의 configuration을 고려해보자.

```java
@Configuration
public class MovieConfiguration{
   
   @Bean
   @Primary
   public MovieCatalog firstMovieCatalog(){
      // ...
   }
   
   @Bean
   public MovieCatalog SecondMovieCatalog(){
      // ...
   }
   
}
```

위의 구성에서, 다음의 `MovieRecommender`는 `firstMovieCatalog와` autowired 됩니다. (be autowired with)

```java
public class MovieRecommender{
   
   @Autowired
   private MovieCatalog movieCatalog;
   
   // ...
}
```

해당 Bean 정의는 다음과 같습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                         https://www.springframework.org/schema/context/spring-context.xsd">
	<context:annotation-config/>
   
   <bean class="example.SimpleMovieCatalog" primary="true">
   	<!-- Bean에 의해 요구되어지는 모든 의존성 주입 -->
   </bean>

   <bean class="example.SimpleMovieCatalog">
   	<!-- Bean에 의해 요구되어지는 모든 의존성 주입 -->
   </bean>
   
   <bean id="movieRecommender" class="example.MovieRecommender"/>
   
</beans>
```

---

### 1.9.4. Fine-tuning Annotation-based Autowiring with Qualifiers (qualifiers를 이용한 Annotation 기반의 미세조정 Autowiring)

**하나의 최상위 후보자가 결정되어질 수 있을 때, `@Primary`는 여러 인스턴스에서 타입에 의한 autowiring을 사용하기 위한 가장 효과적인 방법이다.** 선택 프로세스 동안에 더 많은 제어가 필요할 때, 개발자는 Spring의 `@Qualifier` 애노테이션을 사용할 수 있다. **개발자는 Qualifier 값을 특정한 인자와 연관시켜(associate) 특정 Bean이 각 인자를 위해 선택되도록 유형 일치 세트를 좁힐(narrowing) 수 있습니다.** 가장 간단한 경우에서, Qualifier는 가장 평범한 설명(descriptive) 값이다. 다음 예제를 참고하자. 

```java
public class MovieRecommender{
   
   @Autowired
   @Qualifier("main")
   private MovieCatalog movieCatalog;
   
   // ...
}
```

개발자는 `@Qualifier` 애노테이션을 개개의 생성자 인자 또는 메소드 인자에 명시할 수 있다. 다음의 예를 참고하자.

```java
public class MovieRecommender {

   private MovieCatalog movieCatalog;
   
   private CustomerPreferenceDao customerPreferenceDao;
   
   @Autowired
   public void prepare(@Qualifier("main") MovieCatalog movieCatalog, 
							CustomerPreferenceDao customerPreferenceDao) {
      this.movieCatalog = movieCatalog;
      this.customerPreferenceDao = customerPreferenceDao;
   }
   
   // ...
}
```

다음은 해당 Bean 정의를 보여주는 예제이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

   <context:annotation-config/>
   
   <bean class="example.SimpleMovieCatalog">
   	<qualifier value="main"/> // (1)
      
      <!-- 해당 Bean에 의해 요구되어지는 의존성 삽입 -->
   </bean>
   
   <bean class="example.SimpleMovieCatalog">
   	<qualifier value="action"/> // (2)
      
      <!-- 해당 Bean에 의해 요구되어지는 의존성 삽입 -->
   </bean>
   
   <bean id="movieRecommender" class="example.MovieRecommender"/>
   
</beans>
```

1. `main` qualifier 값을 가진 Bean은 같은 값으로 규정된 생성자 인수와 연결됩니다.
2. `action` qualifier 값을 가진 Bean은 같은 값으로 규정된 생성자 인수와 연결됩니다.

**대체 match의 경우, Bean name은 기본적인 qualifier 값으로 고려되어진다.** 그러므로 개발자는 중첩된 qualifier 요소 대신에 `id`가 `main`인 Bean을 정의할 수 있고, 같은 결과를 야기시킬 것이다. 그러나, **name에 의해서 특정한 Bean을 참조하는 규칙을 사용할 수 있음에도 불구하고, `@Autowired`는 기본적으로 선택적인 semantic qualifier을 사용한 type기반의 주입입니다.** **심지어 Bean name의 대체로 사용되어지는 qualifier 값은 타입 match 세트 내에서 좁은 의미를 항상 가지고 있습니다.** qualifier value는 고유한 Bean `id`에 대한 참조를 의미적으로 표현하지 않습니다. 좋은 qualifier 값은 `main` 또는 `EMEA` 또는 `persistent` 이며, Bean `id`의 독립적 특정 구성요소의 특성을 나타내며, 이는 익명 Bean 정의의 경우 이전 예제와 같이 자동 생성 될 수 있습니다. 

**Qualifier는 이전에 논의된 `Set<MovieCatalog>`와 같은 타입이 정의된 컬렉션에 사용된다. 이러한 경우에, 선언된 qualifiers에 따라(according to) 모든 일치하는 Beans는 컬렉션으로써 의존성이 주입되어진다.** **이러한 사실은 qualifier가 유일할 필요가 없다는 것을 함축한다.** 오히려 그들은 필터링 기준을 구성합니다. 예로들어, 개발자가 같은 qualifier 값 "action"으로 여러 `MovieCatalog` Bean을 정의할 수 있고, 이와같은 모든 Bean들은 `@Qualifier("action")`애노테이션이 붙은 `Set<MovieCatalog>` 에 주입됩니다.

> 타입-matching 내에서 qualifier 값들이 target Bean names에 대응해 선택하는것은, 주입 지점에서 `@Qualifier` 애노테이션이 필요하지 않는다. **만약 유일하지 않는 의존성 상황에 대하여 어떠한 해결 지시자(ex. qualifier 또는 Primary)도 없다면, Spring은 target Bean name에 대하여 의존성 주입 이름을 match할 것이다. 그리고 같은 이름의 후보자를 선택할 것이다.**

**즉, 만약 개발자가 name에 의한 annotation 기반의 의존성 주입을 표현하고자 한다면, 유형 일치 후보중에서 Bean anme으로 선택할 수 있다고 하더라도 `@Autowired`를 우선적으로 사용하지 마십시요.** **대신에 JSR-250 `@Resource` 애노테이션을 사용하세요. `@Resource` 애노테이션은 구성요소의 유일한 name에 의해 특정한 target 구성요소를 식별하기 위해 의미적으로 정의됩니다.**  matching process와 선언된 유형이 관계가 없는 애노테이션이다. `@Autowired`는 꽤(rather) 다른 의미를 가집니다: 타입에 의해서 후보자 Bean이 선택되어지고 난 후, 특정한 `String` qualifier 값은 이러한 선택된 후보자 내에서만 고려되어 집니다.

컬렉션, `Map`, 배열 타입으로 선언된 Bean에 대하여, `@Resource`는 좋은 선택이다. `@Resource`는 유일한 name에 의해 특정한 컬렉션 또는 배열 Bean을 참조한다. **즉, 4.3에서 현재 컬렉션의 경우,  `@Bean` 반환 타입 서명 또는 컬렉션 상속 계층 구조에서 요소 타입 정보가 유지되는 한 Spring의 `@Autowired` 타입 일치 알고리즘을 통해 Map, array 타입을 일치시킬 수 있습니다.** 이러한 경우에, 개발자는 같은 타입의 여러개의 컬렉션에서 Bean을 선택하기 위해 qualifier 값을 사용할 수 있습니다.

4.3 현재, `@Autowired`는 주입에 대한 자체 참조도 고려합니다.(즉, 현재 주입되어진 Bean에 대한 참조) 자신의 의존성을 주입하는 것은 대비책 이라는 것을 명심해라. 다른 Components의 일반적인 의존성 주입은 항상 우선(precedence)을 갖는다. 이러한 의미에서, 자체 참조는 일반적인 후보자 선택에 참여할 수 없고, 그러므로 절대로 최우선순위가 될 수 없다. 반대로, 자체 참조는 항상 낮은 우선 순위로 끝납니다. 실제로, 개발자는 자체 참조를 최후에 수단(resort) 으로써 사용해야 합니다.(ex. Bean의 트랜잭션 프록시를 통해 같은 인스턴스에서 다른 메소드를 호출하는 경우) 이러한 시나리오에서 영향을 받는 메소드를 별도의 대리자 Bean으로 제외(factor out)시키는 것을 고려하십시오. 그 대신에, 개발자는 `@Resource`를 사용하여 고유 이름으로 현재 Bean 프록시를 다시 얻을 수 있습니다.

> **같은 configuration class의 `@Bean` 메소드의 결과를 주입하기 위해 시도하는 것은 사실은 자체 참조 시나리오 입니다. **  실제로 필요한 경우(configuration class의 autowired 필드와 반대로) 메소드 시그니처에서 이러한 참조를 느리게 해결하거나 영향을 받는  `@Bean` 메소드를 정적으로 선언하여 설정 클래스 인스턴스 및 라이프 사이클에서 분리합니다. 그렇지 않으면 이러한 Bean들은 최상위 후보자로서 선택된 다른 configuration class들에서 Bean을 match하는 차선책을 고려해야한다.(가능하다면)

**`@Autowired`는 필드와 생성자, 여러 인자를 갖는 메소드에 적용하고, qualifier 애노테이션을 통해 parameter 단계에서 후보자 줄이기가 가능하다. 이와 대조적으로 `@Resource`는 단일 인자를 갖는 Bean Property setter 메소드와 필드를 위해 지원되어진다.** 결과적으로, 개발자는 주입 대상이 생성자 또는 다중 인자 메소드인 경우에 qualifier을 사용해야 한다.

개발자는 자신의 사용자 정의 qualifier 애노테이션을 생성할 수 있다. 이렇게 하기 위해서, 애노테이션을 정의하고 정의 내에 `@Qualifier`  애노테이션을 제공해야한다. 다음 예제에서 보여준다.

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {
   
   String value();
}
```

개발자는 사용자 정의 qualifier을 autowired된 필드와 인자에 다음 예제와 같이 사용할 수 있다.

```java
public class MovieRecommender {
   @Autowired
   @Genre("Action")
   private MovieCatalog actionCatalog;
   
   private MovieCatalog comedyCatalog;
   
   @Autowired
   public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog){
      this.comedyCatalog = comedyCatalog;
   }
   
   // ...
}
```

다음으로, 개발자는 후보자 Bean 정의를 위해 정보를 제공할 수 있습니다. 개발자는 `<bean/>` 태그의 sub요소로써 `<qualifier/>` 태그를 추가할 수 있고 사용자 qualifier 애노테이션을 match하기위해 `type`과 `value`를 명시합니다. 이러한 타입은 애노테이션의 완전히 qualified된 클래스 name에 대응해서 match 되어집니다.  그 대신에, 이름 충돌의 위험이 존재하지 않는 경우 편의상 짧은 클래스 name을 사용할 수 있습니다. 다음 예제는 두 가지 접근 방식을 보여줍니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- 해당 Bean이 필요한 모든 의존성 주입 -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- 해당 Bean이 필요한 모든 의존성 주입 -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

Classpath Scanning and Managed Components에서, 개발자는 XML에 qualifier metadata를 제공하는 대신(alternative to) 애노테이션 기반을 볼 수 잇을 것이다. 명시적으로 Providing Qualifier Metadata with Annotations를 참고해 보자.

**몇가지의 경우에서, 값 없이 애노테이션을 사용하는 것은 충분(suffice)할 것이다. 이러한 경우는 애노테이션이 보다 더 일반적인(generic) 목적에 서비스 되어지거나 여러 다른 타입의 의존성에 적용되어질 수 있을 때 더 유용하다.** 예로들어, 개발자는 어떠한 인터넷 연결도 사용할 수 없을 때 검색되어질 수 있는 오프라인 목록을 제공할 것이다. 첫번째로 다음 예제와 같이 간단한 애노테이션을 정의한다.

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface offline {
   
}
```

 그리고 다음의 예제처럼 autowired된 property 또는 필드에 애노테이션을 추가한다.

```java
public class MovieRecommender{
   @Autowired
   @Offline // (1)
   private MovieCatalog offlineCatalog;
   
   // ...
}
```

1. 이 라인에서는 `@Offline` 애노테이션을 추가하였다.

이제 Bean 정의에서 다음 예제와 같이 qualifier 유형만 필요합니다.

```xml
<bean class="example.SimplemovieCatalog">
	<qualifier type="Offline"/> // (1)
   <!-- 해당 Bean에 필요한 모든 의존성들을 주입 -->
</bean>
```

1. 이러한 요소는 qualifier을 명시한다.

**개발자는 또한 간단한 `value` 속성 대신에 이름이 붙여진 속성을 사용할 수 있는 사용자 qualfier 애노테이션을 정의할 수 있다.** 여러 속성 값이 autowired된 인자 또는 필드에 명시되어 있을때, Bean 정의는 autowire 후보로 간주되기 위해 이러한 모든 속성값과 일치해야 합니다. 다음의 annotation 정의 예제를 고려해 보자.

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier{
   
   String genre();
   
   Format format();
}
```

이경우에 `Format`은 enum이고, 다음과 같이 정의되어 있다.

```java
public enum Format{
   VHS, DVD, BLURAY
}
```

autowired된 필드는 사용자 qualifier와 함께 annotation이 붙여져 있고, 두개의 속성인(`genre` 와 `format`) 모두의 값을 다음 예제에서와 같이 포함한다.

```java
public class MovieRecommender{
   
   @Autowired
   @MovieQualifier(format=Format.VHS, genre="Action")
   priavte MovieCatalog actionVhsCatalog;
   
   @Autowired
   @MovieQualifier(format=Format.VHS, genre="Comedy")
   private MovieCatalog comedyVhsCatalog;
   
   @Autowired
   @MovieQualifier(format=Format.DVD, genre="Action")
   private MovieCatalog actionDvdCatalog;
   
   @Autowired
   @MovieQualifier(format=Format.BLURAY, genre="Comedy")
   private MovieCatalog comedyBluRayCatalog;
   
   //...
}
```

**최종적으로, Bean 정의는 일치하는 qualifier 값을 포함해야만 한다.** 이 예제는 `<qualifier/>` 요소 대신에 Bean meta 속성을 사용할 수 있다는 것을 설명하는 예제이다. 만약 가능하다면, `<qualifier/>` 요소와 속성이 우선시하지만, 다음 예의 마지막 두개의 Bean 정의에서와 같이 qualifier가 존재하지 않는 경우 `<meta/>`태그 내에 제공된 값으로 대체됩니다(fall back).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
	     <!-- 해당 Bean에 필요한 모든 의존성들을 주입 -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
	     <!-- 해당 Bean에 필요한 모든 의존성들을 주입 -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
	     <!-- 해당 Bean에 필요한 모든 의존성들을 주입 -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
	     <!-- 해당 Bean에 필요한 모든 의존성들을 주입 -->
    </bean>

</beans>
```

---

### 1.9.5 Using Generics as Autowiring Qualifiers

**`@Qualifier` 애노테이션 이외에, 개발자는 암시적 형식의 자격으로  Java generic 타입을 사용할 수 있다.** 

예로들어 다음과 같은 cofiguration을 갖고 있다고 가정 해 보자.

```javascript
@Configuration
public class MyConfiguration{
   
   @Bean
   public StringStore stringStore(){
      return new StringStore();
   }
   
   @Bean
   public IntegerStore integerStore(){
      return new IntegerStore();
   }
}
```

위의 Bean들이 제네릭 인터페이스를 구현한다고 가정해 보자. (즉 `Store<String>`  과 `Store<Integer>`). 

> ##### **Generic Interface**
>
> **제네릭** : 클래스에서 매개변수 형식의 자료형을 사용할 수 있는 기능이다.
>
> - 자료형에 얽매이지 않는 알고리즘을 작성할 수 있다.
> - 알고리즘의 재사용성을 높인다
> - 자료형으로 인해 생기는 프로그램 중복을 줄여준다.
>
> **제네릭 인터페이스** 
>
> ```java
> interface GenericInterface <TypeParameters> {
> 인터페이스 내용
> }
> ```
>
> **제네릭 인터페이스 예시**
>
> ```java
> public interface GenericInterface<T>{
>    public void setValue(T x);
>    public String getValueType();
> }
> ```
>
> **제네릭 인터페이스를 구현한 제네릭 예시**
>
> ```java
> public class GenericClass<T> implements GenericInterface<T>{
>    private T value;
>    @Override
>    public void setValue(T x){
>       value = x;
>    }
>    @Override
>    public String getValueType(){
>       return value.getClass().toString();
>    }
> }
> ```

개발자는 qualifier로써 사용되는 generic과 인터페이스를 `@Autowire`할 수 있다. 다음 예제에서 보여준다.

```java
@Autowired
private Store<String> s1; // <String> qualifier, stringStore Bean을 주입한다.

@Autowired
private Store<Integer> s2; // <Integer> qualifier, integerStore Bean을 주입한다.
```

제네릭 qualifier는 `List` 와 `Map` 인스턴스 및 배열을 autowiring 할 때 사용된다. 다음의 예는 제네릭 `List` 를 autowire하는 예제이다.

```java
// <Integer> 제네릭을 갖는 모든 Store Bean을 주입한다.
// 이 List에서는 Store<String> Bean들은 나타나지 않을 것이다.
@Autowired
private List<Store<Integer>> s;
```

---

### 1.9.6 Using `CustomAutowireConfigurer`

































