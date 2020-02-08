## 1.13 Environment Abstraction (환경 추상화)

**`Environment` 인터페이스는 애플리케이션 환경의 두 가지 주요 측면인 `profiles` 와 `properties` 를 모델링하는 Container의 통합된 추상화 입니다.**

<u>**profile은 지정된 profile이 활성화된 경우에만 Container에 등록될 명명된 논리 Bean 정의 그룹입니다.**</u> Bean들은 XML로 정의된 Profile 또는 annotations가 붙여진 profile에 할당되어질 것입니다. **profile과 관련있는 `Environment` 객체의 역할은 어떠한 profiles이 현재 활성화 되어 있고, 어떠한 profiles이 기본적으로 활성화 되어야 하는지 결정합니다.**

**Properties는 대부분의 모든 어플리케이션에서 중요한 역할을 수행하고 다양한 소스로부터 비롯될 수 있다**
다양한 소스 : properties.files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc `Properties` objects, `Map` objects, ...

**properties와 관련있는 `Environment` 객체의 역할은 Property source를 설정하고 그들(property source)로 부터 properties를 추출할 수 있게 편리한 service 인터페이스를 사용자에게 제공해준다.**

### 1.13.1 Bean Definition Profiles

**Bean Definition profiles는 핵심 Container에 다른 환경에서 다른 Bean들을 등록할 수 있도록 허락해 주는 메커니즘을 제공합니다.** "environment" 라는 사용자마다 다른 것들을 의미하고, 이러한 특징들은 많은 사용 사례에 도움을 줍니다.
다음의 것들을 포함한다.

- QA 또는 프로덕션 환경에서 JNDI로부터 동일한 데이터 소스를 찾는 것과 비교하여 개발중인 내장메모리 데이터 소스에 대해 작업하는 것
- 어플리케이션을 performance 환경에 배포할 때만 인프라 모니터링을 등록
- 사용자 A 또는 사용자 B를 위한 Bean들의 사용자정의 구현 등록

`DataSource` 를 요구하는 실제 애플리케이션에서 첫번째 사용 사례를 고려해 보자. 테스트 환경에서 설정은 다음의 예제와 비슷하다.

```java
@Bean
public DataSource dataSource() {
   return new EmbeddedDatabaseBuilder()
      .setType(EmbeddedDatabaseType.HSQL)
      .addScript("my-schema.sql")
      .addScript("my-test-data.sql")
      .build();
}
```

에플리케이션을위한 dataSource가 어플리케이션 서버의 JNDI 디렉토리에 등록되었다고 가정해, 어떻게 이 어플리케이션이 QA 또는 production 환경에 배포되는지에 대해서 고려 해 보자. `dataSource` Bean은 다음의 리스트와 비슷하다.

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
   Context ctx = new InitialContext();
   return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource")
}
```

문제는 어떻게 현재 환경을 기반으로하는 두가지 변형 중에서 어떻게 바꿀 것인가가 문제이다. 시간이 지남에 따라, Spring 사용자는 이러한 작업을 수행할 수 있는 여러가지 방법을 고안했습니다(devise). **이러한 것들은 보통 시스템 환경 변수의 조합 또는 환경 변수의 값에 의존하는 설정 파일을 분석할 수 있는`${placeholder}`토큰을 포함하는 XML `<import/>` 구문에 의존합니다.** Bean Definition profiles는 이러한 문제의 해결책을 제공하는 Container의 핵심 기능입니다.

만약 특정한 환경의 Bean Definition 예제에서 보여주는 사례를 일반화하면, 우리는 결국 특정한 Contexts에서 특정한 Bean Definitions를 등록할 필요가 있습니다.(end up with : 결국 ~하게되다.) **개발자는 상황 A 에서 Bean 정의의 특정한 profile 및 상황 B 에서 또다른 profile을 등록하길 원한다고 말할 수 있습니다.** 우리는 이러한 요구를 반영하기 위해 우리의 설정을 업데이트 하기 시작할 것입니다.

####  Using `@Profile`

**`@Profile` 애노테이션은 하나이상의 특정한 profiles가 활성화 될 때 Component가 등록할 자격이 있다는것을 가리킵니다.** 이전 예제를 사용함으로써, `dataSource` 설정을 다음과 같이 재작성 할 수 있습니다.

```java
@Configuration
@profile("development")
public class StandaloneDataConfig {
   
   @Bean
   public DataSource dataSource() {
      return new EmbeddedDatabaseBuilder()
         .setType(EmbeddedDatabaseType.HSQL)
         .addScript("classpath:com/bank/config/sql/schema.sql")
         .addScript("classpath:com/bank/config/sql/test-data.sql")
         .build();
   }
}
```

```java
@Configuration
@Profile("productio")
public class JndiDataConfig {
   
   @Bean(destroyMethod="")
   public DataSource dataSource() throws Exception {
      Context ctx = new InitialContext();
      return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
   }
}
```

> 이전에 언급했듯이, `@Bean` 메소드 에서는, 개발자는 일반적으로 Spring의 `JndiTemplate` / `JndiLocatorDelegate` helpers 또는 stragiht JNDI `IntialContext`를 사용함으로써 프로그래밍 방식의 JNDI 조회를 사용하도록 선택합니다.그리고, `FactoryBean`타입으로써 리턴타입을 선언하는 것을 강요하는  `JndiObjectFactoryBean`는 사용하지 않습니다.

Profile String은 간단한 Profile name(예로들어, `production`) 또는 profile 표현식을 포함할 수 있다. profile 표현식은 더 복잡한 profile 로직이 표현되는 것을 허용합니다.(예로들어, `proudction & us-east`). 다음 연산자는 profile 표현식에서 제공되어지는 연산자 입니다.

- `!` : profile에 대한 논리적 "not"
- `&` : profiles에 대한 논리적 "and"
- `|` : profiles에 대한 논리적 "or"

> 개발자는 `&`와 `|` 연산자를 괄호(parentheses) 사용 없이 혼합해서 사용할 수 없다. 예로들어, `production & us-east | eu-central` 은 유효하지 않는 표현식이다. 이러한 유효하지 않는 표현식은 `production & (us-east | eu-central)`와 같은 유효한 표현식으로 바꿀 수 있다.

**개발자는 `@Profile` 애노테이션을 사용자가 구성한 애노테이션을 생성할 목적으로 meta-annotation으로써 사용할 수 있다.** 다음의 예제는 `@Profile("production")`을 위한 drop-in 대체로써 사용자 `@Production` 애노테이션을 정의할 수 있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
   
}
```

> **만약 `@Configuration` 클래스가 `@Profile` 애노테이션이 붙여져 있다면 모든 `@Bean` 메소드와 `@Import` 애노테이션과 연관된 해당 클래스들은 하나이상의 특정한 Profile이 활성화 되지 않는다면 무시되어진다(bypassed).** 만약 `@Component` 또는 `@Configuration` 클래스가 `@Profile({"p1", "p2"})` 애노테이션으로써 표시되어져 있다면, 해당 클래스는 profile "p1" 또는 "p2"가 활성화 되지 않는 한(unless) 등록되어지거나 또는 처리되어지지 않는다. 만약 주어진 profile이 Not 연산자(`!`)가 앞에 붙어 있다면, 어노테이션이 붙여진 요소들은 오직 해당 profile이 활성화 되지 않을 때 등록되어질 것입니다. 예로들어, `@Profile({"p1", "!p2"})`가 요소에 붙여 졌을 때, 등록은 profile "p1"이 활성화 되거나 또는 profile "p2"가 활성화 되지 않으면 발생할 것입니다.

**`@Profile` 애노테이션은 configuration 클래스에서 오직 특정한 Bean을 포함하기 위해서 메소드 레벨에서 선언되어질 수 있습니다.** (예로들어, 특정한 Bean에 대한 대체 변동을 위해), 다음의 예제에서 보여줍니다.

```java
@Configuration
public class AppConfig {

   @Bean("dataSource")
   @Profile("development") // (1)
   public DataSource standaloneDataSource(){
      return new EmbeddedDatabaseBuilder()
         .setType(EmbeddedDatabaseType.HSQL)
         .addScript("classpath:com/bank/cofig/sql/schema.sql")
         .addScript("classpath:com/bank/config/sql/test-data.sql")
         .build()
   }
   
   @Bean("dataSource")
   @Profile("production") // (2)
	public DataSource jndiDataSource() throws Exception {
      Context ctx = new InitialContext();
      return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
   }
}
```

1. `standaloneDataSource` 메소드는 오직 `development` profile에서만 사용 가능하다. 
2. `jndiDataSource` 메소드는 오직 `production` profile에서만 사용 가능하다.

> `@Bean` 메소드의 `@Profile` 처럼, 특별한 시나리오가 적용 가능하다 : 동일한 Java 메소드 name의 `@Bean` 메소드가 오버로드된 경우 (생성자 오버로드와 유사(analogous to)), 모든 오버로드된 메소드에서  `@Profile` 조건은 일관되게 선언되어야 한다. **만약 해당 조건이 일관되지 않으면, 오버로드된 메소드 중 첫번째 조건만 중요한 사항이 됩니다. 그러므로 `@Profile`은 특정한 인자로 오버로드된 메소드를 선택하기 위해 사용할 수 없습니다.** 동일한 Bean을 위한 모든 팩토리 메소드들 간의 Resolution은 Spring 생성자 resolution 알고리즘을 생성시간 때 따릅니다.
>
> 개발자가 다른 Profile 조건과 함께 대체 Bean을 정의하고 싶다면, 이전 예제에서 보여주는 것 처럼  `@Bean` name 속성을 사용해 동일한 Bean name을 가르키는 고유한 Java 메소드 name을 사용해라. 모든 인자의 특징이 모두 같다면(예로들어, 모든 변형이 인자가 없는 팩토리 메소드를 가진다면), 우선(in the first place) 유효한 자바 클래스에 이러한 배열을 나타내는 유일한 방법 입니다.( 특정한 name과 인자의 특징에 대한 메소드는 하나만 있을 수 있기 때문입니다.)

#### XML Bean Definition Profiles

XML에 대응되는 것은 `<beans>` 요소의 `profile` 속성입니다. 이전 샘플 설정은 다음과 같이 두가지의 XML 파일들로 재 작성 될 수 있습니다.

```xml
<beans profile="development"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jdbc="http://www.springframewrok.org/schema/jdbc"
       xsi:schemaLocation="...">
	
   <jdbc:embedded-database id="dataSource">
   	<jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
      <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
   </jdbc:embedded-database>
</beans>
```

```xml
<beans profile="production"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jee="http://www.springframnework.org/schema/jee"
       xsi:schemaLocation="...">

   <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

다음 예제에서와 같이 동일한 파일에서 `<beans/>` 요소를 중첩하고 분리하는 것을 피할 수 있습니다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:jee="http://www.springframework.org/schema/jee"
       xsi:schemaLocation="...">
	
   <!-- 다른 Bean 정의 -->
   
   <beans profile="development">
   	<jdbc:embedded-database id="dataSource">
      	<jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
         <jdbc:script location="clsspath::com/bank/config/sql/test-data.sql"/>
      </jdbc:embedded-database>
   </beans>
   
   <beans profile="production">
   	<jee:jndi-lookup id="dataSource" jndi-name="java.comp/env/jdbc/datasource"/>
   </beans>
   
</beans>
```

**`spring-bean.xsd`은 파일의 마지막 요소와 같은 요소만 허용하도록 제한되어 있습니다.** 이러한 것들은 XML 파일에서 혼란(clutter)을 일으키지(incurring) 않고 유연성을 제공합니다.

> XML은 앞에서 설명한 profile 표현식을 지원하지 않습니다. 그러나 `!` 연산자를 사용함으로써 profile을 무효화(negate) 할 수 있습니다. 다음 예제와 같이 중첩된 profile을 사용함으로써 논리적인 "and"를 적용할 수도 있습니다.
>
> ```xml
> <beans xmlns="http://www.springframework.org/schema/beans"
>     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>     xmlns:jdbc="http://www.springframework.org/schema/jdbc"
>     xmlns:jee="http://www.springframework.org/schema/jee"
>     xsi:schemaLocation="...">
>    
>    <!-- 다른 Bean 정의 -->
>    
>    <beans profile="production">
>    	<beans profile="us-east">
>       	<jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
>       </beans>
>    </beans>
> </beans>
> ```
>
> 이전 예제에서, `production` 과 `us-east` profile이 둘다 활성화 되면, `dataSource` Bean이 노출되어졌다.

#### Activating a Profile

**설정을 업데이트 했음으로(Now that), Spring에게 어떠한 profile을 활성화 할 것인지 지시해야 합니다.** 샘플 애플리케이션을 지금 시작했다면, Container가 `dataSource` 라고 명명된 Bean을 찾을 수 없음으로 `NoSuchBeanDefinitionException`  예외가 던져질 것이다.

profile을 활성화 시키는 것은 여러가지 방법으로 수행 되어질 수 있지만, 가장 간단한 방법은 `ApplicationContext` 를 통해 사용 가능한 `Environment` API에 대해서 프로그래밍 방식으로 이러한 것들을 수행하는 것 입니다. 다음 예제에서 어떻게 이러한 것들을 수행하는지에 대해서 보여줍니다.

```java
public static void main(String[] args){
   AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
	ctx.getEnvironment().setActiveProfiles("development");
   ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
   ctx.refresh();
}
```

**게다가, 개발자는 `spring.profiles.active` property를 통해서 선언적으로 profiles를 활성화 시킬 수 있다.** `spring.profiles.active` property는 시스템 환경변수, JVM 시스템 properties, `web.xml`의 서블릿 context 인자, 심지어는 JNDI의 항목(see PropertySource Abstraction)으로 지정할 수 있습니다. 통합 테스트에서, 스프링 테스트 모듈의 `@ActiveProfiles` 애노테이션을 사용함으로써 활성 프로파일을 선언할 수 있습니다. (context configuration with environment profiles(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/testing.html#testcontext-ctx-management-env-profiles) 를 참고해라.)

**Profiles은 "either-or" 제안이 아니라는 것을 명심해라. 개발자는 한번에 여러개의 Profiles를 활성화 시킬 수 있다. 프로그램적으로, 개발자는 `setActiveProfiles()` 메소드에 여러개의 profile names를 제공할 수 있다.** `setActiveProfiles()` 메소드는 `String...` varargs를 받아들일 수 있다. 다음 예제는 여러개의 profiles을 활성화 시키는 예제이다.

```java
ctx.getEnvironment().setActiveProfiles("profile1","profile2");
```

`spring.profiles.active`는 profiles name가 컴마(,)로 구분된 리스트를 다음 예제와 같이 수용할 수 있다.

```
-Dspring.profiles.active="profile1,profile2"
```

#### Default Profile

기본적인 profile은 기본적으로 활성화 되어 있는 profile을 포함한다. 다음 예제를 고려해 보자.

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {
   
   @Bean
   public DataSource dataSource() {
      return new EmbeddedDatabaseBuilder()
         .setType(EmbeddedDatabaseType.HSQL)
         .addScript("classpath:com/bank/config/sql/schema.sql")
         .build();
   }
}
```

**만약 어떠한 profile도 활성화 되지 않으면, `dataSource`는 생성되어질 것이다.** 개발자는 하나 이상의 Bean을 위한 기본 정의를 제공하는 방법으로써 이러한 것들을 사용할 수 있다. **만약 어느 하나의 profile이라도 활성화 되면, 기본적인 profile은 적용되지 않을 것이다.**

개발자는 `Environment` 의 `setDefaultProfiles()` 또는 `spring.profiles.default` property를 사용함으로써 기본 profile의 이름을 바꿀 수 있습니다.

---

### 1.13.2 `PropertySource` Abstraction

**Spring의 `Environment` 추상화는 property source의 설정가능한 계층에서 검색 작업(operation)을 제공합니다.** 다음의 리스트를 고려해 보자.

```java
public static void main(String[] args){
   ApplicationContext ctx = new GenericApplicationContext();
   Environment env = ctx.getEnvironment();
   boolean containsMyProperty = env.containsProperty("my-property");
   System.out.println("Does my environment contain the'my-property' property? " + containsMyProperty)
}
```

이전 설명에서, Spring에게 `my-property` property가 현재 환경에 정의되어 있는지 요청하는 고 수준의 방법에 대해서 살펴 보았다. 이러한 질문에 답하기 위해서, `Environment` 객체는 일련의 `PropertySource` 객체 검사를 수행합니다. **`PropertySource`는 모든 키-값 소스에 대한 간단한 추상화입니다, 그리고 Spring의 `StandardEnvironment`는 2가지의 PropertySource 객체들과 함께 설정이 되어있다. 하나는 일련의 JVM system properties에 대해서 나타내고(System.getProperties()) 그리고 다른 하나는 일련의 시스템 환경 변수를 나타낸다(System.getenv()).**

> 이러한 기본적인 property 소스들은 독립형 어플리케이션에서 사용하기위해서  `StandardEnvironment`애 존재합니다.
> `standardServletEnvironment`는 servlet config 와 servlet context parameters를 포함한 추가적인 기본 property 소스와 함께 채워집니다(populated). 이러한 것들은 선택적으로 `JndiPropertySource` 를 활성화 시킬 수 있습니다. 더 자세한 내용은 Java document를 참고하세요!!

구체적으로 개발자가 `StandardEnvironment`를 사용할 때, `env.contansProperty("my-property")` 호출은 
`my-property` 시스템 property 또는 `my-property` 환경 변수가 런타임시에 존재하는 경우 `true`를 리턴한다.

> 수행되는 검색은 계층적이다. 기본적으로, 시스템 properties는 환경 변수보다(over) 우선(precedence)합니다. 그러므로 `my-property` property가 양쪽에서 설정 되어져서 `env.getProperty("my-property")`를 호출하는 동안에 발생한다면, 시스템 property 값 "wins"가 리턴됩니다. **property 값들은 병합되어지지 않고 이전 항목으로 완전히 재정의 된다는 것을 명심해야 합니다.**
>
> 일반적인 `StandardServletEnvironment`를 위해서, 완전한 계층이 명시되어 있습니다. 가장 높은 우선순위 항목은 가장 위에 위치해 있습니다.
>
> 1. Servletconfig parameters ( 해당되는 경우 - 예 : `DispatcherServlet` context의 경우 )
> 2. ServletContext parameters ( web.xml context-param 항목)
> 3. JNDI environment variables ( `java:comp/env/` 항목들)
> 4. JVM system properties ( `-D` command-line 인자 )
> 5. JVM system environment ( 작동하는 시스템 환경 변수 )

가장 중요한것으로, 전체 메커니즘은 설정할 수 잇습니다. 이러한 검색에 통합하려는 사용자 지정 properties 소스가 있을 수 있습니다. 다음과 같이 진행하려면, 사용자의 `PropertySource` 를 인스턴스화 하고 구현해야 한다. 그리고 현재 `Environment`를 위해 일련의 `PropertySources`에 `PropertySource`를 추가해야 한다. 다음의 예제는 어떻게 사용하는지에 대해서 보여준다.

```java
public static void main(String[] args){
   ConfigurableApplicationContext ctx = new GenericApplicationContext();
   MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
   sources.addFirst(new MyPropertySources());
}
```

위의 코드에서, `MyPropertySource` 는 검색에 가장 높은 우선순위로써 추가되어 졌다. `MyPropertySource`가 
`my-property` property를 포함한다면 탐색 되어지고 리턴되어질 것입니다. `MutablePropertySources` API는 일련의 property 소스를 정확하게 조작(manipulation) 할 수 있는 많은 메서드를 제공합니다.





























