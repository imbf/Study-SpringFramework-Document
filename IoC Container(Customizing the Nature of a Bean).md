## 1.6 Customizing the Nature of a Bean

Spring Framework는 Bean의 특성을 사용자 정의하는 데 사용할 수 있는 많은 인터페이스를 제공합니다.
이번 section은 다음과 같이 이루어져 있습니다.

- Lifecycle Callbacks
- ApplicationContextAware and BeanNameAware
- Other Aware Interface

### 1.6.1 Lifecycle Callbacks

Container의 Bean 수명주기 관리와 상호작용하기 위해서, 개발자는 Spring의 `InitializingBean` 과 `DisposableBean` 인터페이스를 구현해야한다. Bean이 초기화 및 소멸 할시 특정한 행동을 수행하기 위해, 초기화 할 때는 `afterPropertiesSet()`을 호출하고, 소멸할 때는 `destory()`를 호출한다.

> JSR-250 `@PostContruct` 와 `@PreDestory` 어노테이션은 일반적인 스프링 어플리케이션에서 생명주기 콜백을 받기위한 가장 좋은 관습(습관, 행동)으로 여겨진다. 이러한 어노테이션을 사용하는 것은 Bean이 Spring의 특정한 인터페이스에 연결(couple)되어 있지 않다는 것을 의미합니다. 
>
> 만약 개발자가 JSR-250 어노테이션을 사용하지 않기 원하면서, Spring의 특정한 인터페이스와의 연결도 하지 않길 원한다면, `init-method` 와 `destory-method` Bean 정의 metadata를 고려해보면 좋다.

내부적으로, Spring Framework는 `BeanPostProcessor` 구현을 사용하여서 적절한 메소드를 찾고 호출 할 수 있는 모든 콜백 인터페이스를 처리합니다. 만약 개발자가 사용자 정의 기능 또는 스프링이 지원해주지 않는 수명주기 행동이 필요하다면, `BeanPostProcessor`를 사용해서 구현할 수 있다. 더 많은 정보가 필요하다면, Container Extension Points를 보아라.

초기화 및 소멸 콜백 외에도, Spring이 관리하는 객체가 시작과 종료 프로세스에 참여할 수 있게끔 하기 위해서 `Lifecycle` 인터페스를 구현할 수 있습니다.

수명주기 콜백 인터페이스는 이번 섹션에서 설명되어질것입니다.

#### Initialization Callbacks

`org.springframework.beans.factory.InitializingBean` 인터페이스는 Container가 Bean의 모든 필요한 properties를 설정하고 난 뒤에 Bean이  초기화 작업을 수행할 수 있게 끔 해줍니다. `InitializingBean` 인터페이스는 단일 메소드를 명시합니다.

```java
void afterPropertiesSet() throws Exception;
```

개발자가 불편하게 Spring과의 코드를 연결지어야 하기 떄문에 `InitializingBean` 인터페이스를 사용하지 않을 것을 권고합니다. 그 대신에, `@PostContstruct` 애노테이션 또는 POJO 초기화 메소드를 명시할 것을 제안합니다. **이러한 경우에 XML 기반의 configuration metadata에서, 개발자는 init-method 속성을 인자가 없는 void 의 메소드 이름을 명시하기 위해서 사용한다.** Java configuration에서, 개발자는 `@Bean`의 `initMethod` 속성을 사용할 수 있습니다.

다음 예제를 참고해보자.

```xml
<!-- Initialization Callbacks를 사용하기 위해 권장되는 코드 -->
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean{
   
   public void init(){
      // do some initialization work
   }
}
```

위의 예제는 다음에 보여질 예제와 거의 대부분 같은 역할을 수행한다.

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean{
   
   @Override
   public void afterPropertiesSet(){
      // do some initialization work
   }
}
```

2개의 예제 중에서 첫번째의 예제가 Spring과의 코드를 연관짓지 않았다. (더 권장된다.)

#### Destruction Callbacks

`org.springframework.beans.factory.DisposableBean` 인터페이스를 구현하는 것은 Bean이 자신을 포함하는 컨테이너가 소멸되어질 때 callback을 호출하게끔 한다. `DisposableBean` 인터페이스는 단일 메소드를 명시한다.

```java
void destory() throws Exception;
```

개발자가 불필요하게 Spring과 Code를 연결해야 하기 때문에, `DisposableBean` 콜백 인터페이스를 사용하지 않을 것을 권고한다. 그 대신에 `@PreDestory` 애노테이션을 사용하거나 Bean의 정의에 의해 제공되어지는 generic method를 명시할 것을 권고한다. XML 기반의 configuration metadata에서, 개발자는  `<bean/>` 요소의 `destory-method` 속성을 사용할 수 있다. Java configuration에서 개발자는 `@Bean` 의 `destoryMethod` 속성을 사용할 수 있다. Receiving Lifecycle Callbacks를 참고해보자.

다음 Bean의 정의를 보자.

```xml
<!-- Destruction Callbacks를 사용하기 위해 권장되는 코드 -->
<bean id="exampleInitBean" class="examples.ExampleBean" destory-methd="cleanup"/>
```

```java
public class ExampleBean{
   public void cleanup(){
      // do some destruction work (ex. 폴링된 연결 해제)
   }
}
```

위의 예제는 다음 보여질 예제와 거의 같은 역할을 수행한다.

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {
   
   @Override
   public void destory(){
      // do some destruction work (ex. 폴링된 연결 해제)
   }
}
```

위의 두 예제 중에서 첫 번째 예제는 Spring과 코드를 연결짓지 않았다. (권고)

> 개발자는 `<bean>` 요소의 `destory-method` 속성에 특별한 추론된(inferred) 값을 할당할 수 있습니다. 이러한 추론된 값들은 Spring이 자동적으로 특정 Bean Class에서 public close 및 public shutdown을 자동적으로 찾을 수 있게 지시한다. (`java.lang.AutoCloseable` 또는 `java.io.Closeable` 을 구현하는 어느 클래스도 일치할 것이다.) 개발자는 또한 이러한 메소드를 전체 Bean에 적용시키기 위해서 `<beans>` 요소의 `default-destory-method` 속성에 추론된 값을 세팅할 수 있다. 이러한 것은 Java configuration의 기본 동작임을 명심해야한다.

#### Default Initialization and Destory Methods

Spring의 특정한  `InitializingBean` 과 `DisposableBean` 콜백 인터페이스를 사용하지 않고 initialization 과 destory 메소드 콜백을 작성할 때,  개발자는 `init()`, `initialize()`, `dispose()`, ... 등의 이름을 가진 메소드를 사용해야한다. 
이상적으로 이러한 lifecycle 콜백 메소드의 이름은 모든 개발자들이 같은 메소드 이름을 사용하고 일관성을 확실히 하기 위해서 프로젝트 전역에 걸쳐서 표준화되어있다.

**개발자는 Spring Container가 모든 Bean의 명명된 초기화 및 소멸 메소드의 이름을 찾도록 설정할 수 있다.** 이러한 사실은 어플리케이션 개발자들이 어플리케이션 class를 작성할 수 있고 각 Bean의 정의에 `init-method="init"`속성을 설정하지 않아도  `init()` 이라고 불리우는 initialization 콜백을 사용할 수 있다는 것을 의미한다. Spring IoC Container는 Bean이 생성될 때 그러한 메소드를 호출한다.(앞에서 설명한 표준 lifecycle 콜백 계약에 일치하여) 이 기능은 또한 초기화 및 소멸 메소드 콜백에 대한 일관된 명명 규칙을 시행합니다.

초기화 메소드의 이름이 `init()` 이고 소멸 메소드의 이름이 `destroy()` 라고 가정하면, 개발자가 정의할 클래스는 다음 예제의 클래스와 유사합니다(resemble).

```java
public class DefaultBlogService implements BlogService{
   
   private BlogDao blogDao;
   
   public void setBlogDao(BlogDao blogDao){
      this.blogDao = blogDao;
   }
   
   // 초기 callcack 메소드
   public void init(){
      if(this.blogDao == null){
         throw new IllegalStateException("The [blogDao] property must be set.")''
      }
   }
}
```

```xml
<beans default-init-method="init">
	
   <bean id="blogService" class="com.something.DefaultBlogService">
   	<property name="blogDao" ref="blogDao"/>
   </bean>
   
</beans>
```

가장 상위 단계의 `<beans/>` 요소에서 `default-init-method` 속성이 존재하는 것은 Spring IoC Container가 초기화 메소드 콜백으로써 Bean class의 init이라고 불리는 메소드를 인지하도록 야기시킨다. Bean이 생성되어지고 조립되어질 때, Bean 클래스가 다음과 같은 메소드를 가진다면, 적절한 시기에 호출이 되어질 것이다.

개발자는 가장 상위 단계의 `<beans/>` 요소에 `default-destory-method` 속성을 사용함으로써 소멸 메소드 콜백을 설정할 수 있다.

만약 존재하는 Bean 클래스들이 규칙에 어긋나게 명명된 callback 메소드를 이미 가지고 있다면, 개발자는 `<bean/>` 요소 자체의 `init-method` 속성과 `destory-method`속성을 사용함으로써 메소드의 이름을 명시함으로써 기본 콜백 메소드를 오버라이드 할 수 있다.

Spring Container는 Bean에 모든 의존성이 제공되어진 후에 설정된 초기화 콜백이 즉시 호출되도록 보증합니다. **따라서(thus) 초기화 콜백은 원시 Bean 참조에서 호출됩니다. 원시 Bean 참조는 AOP 인터셉터 등이 아직 Bean에 적용되지 않습니다.**  target Bean이 완전히 먼저 생성되고 그리고(and then) AOP 인터셉터 체인과 같이 AOP proxy가 적용됩니다. target Bean 과 프록시가 별도로 정의 된 경우, 코드는 프록시를 무시하고(bypassing) 원시 target Bean과 상호작용 할 수 있다. 따라서(hence) 인터셉트를 `init` method에 적용하는 것은 일관성이 없습니다. 그렇게 하면 target Bean의 lifecycle을 프록시 또는 인터셉트에 연결하고 코드가 원시 target Bean과 직접 상호작용 할 때 이상한 의미를 남길 수 있기 때문입니다.

#### Combining Lifecycle Mechanisms

Spring 2.5 부터, Bean lifecycle의 행동을 제어하기 위해 3가지 옵션을 가집니다.

- `Initializingbean` 과 `DisposableBean` 콜백 인터페이스
- 사용자 `init()` 과 `destroy()` 메소드
- `@PostConstruct` 와 `@PreDestory` 애노테이션 (개발자는 이러한 매커니즘을 조합해서 주어진 Bean을 제어할 수 있습니다.)

> Bean에 여러개의 lifecycle mechanism이 설정되어 있고, 각각의 mechanism이 서로 다른 메소드 이름으로 설정되어져 있을 때, 각각의 설정된 메소드는 다음에 정의된 순서대로 실행되어질 것이다. 

여러개의 lifecycle mechanism들이 같은 Bean에 다른 초기화 방법으로 설정되어져 있다면 다음과 같이 호출되어집니다.

1. `@PostConstruct` 애노테이션이 달린 메소드
2. `InitializingBean` 콜백 인터페이스에 의해 정의된 `afterPropertiesSet()` 메소드
3. 사용자가 설정한 `init()` 메소드

소멸 메소드 역시 같은 순서로 호출된다.

1. `@PreDestory` 애노테이션이 달린 메소드
2. `DisposableBean` 콜백 인터페이스에 의해 정의된 `destory()` 메소드
3. 사용자가 설정한 `destory()` 메소드

#### Startup and Shutdown Callbacks (콜백의 시작과 종료)





























