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





























