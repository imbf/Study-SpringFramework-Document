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
> 만약 개발자가 JSR-250 어노테이션을 사용하지 않기 원하면서, Spring의 특정한 인터페이스와의 연결도 하지 않길 원한다면, `init-method` 와 `destory-method` Bean 정의 metadgata를 고려해보면 좋다.

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

**`Lifecycle` 인터페이스는 고유한 lifecycle 요구사항이 있는 모든 객체를 위해 필수 메소드를 정의한다.** (예로들어, 몇 가지의 백 그라운드 작업의 중지 및 실행과)

```java
public interface Lifecycle{
   
   void start();
   
   void stop();
   
   boolean isRunning();
   
}
```

Spring이 관리하는 어떤 객체도  `Lifecycle` 인터페이스를 구현할 것입니다. 그리고 `ApplicationContext` 자체가 시작과 중지 신호를 받을 때, ApplicationContext는 문맥 안에서 정의된 모든 `Lifecycle ` 구현에 이러한 호출을 cascade(연결짓습니다. `LifecycleProcessor`에 위임하여 이를 수행합니다.

다음의 list를 참고해보세요.

```java
public interface LifecycleProcessor extends Lifecycle{
   
   void onRefresh();
   
   void onClose();
}
```

`LifecycleProcessor` 는 그 자체로 `Lifecycle` 인터페이스의 확장을 의미한다. `LifecycleProcessor`는 refresh 되고 close 될 때 상황에 반응하기 위해서 2가지의 다른 메소드를 추가하였다.

>  **일반적인 `org.springframework.context.Lifecycle` 인터페이스는 명백한 시작과 중지 알림에 대한 일반적인 계약이며 context refresh 시 자동 시작을 의미하지 않습니다.** 특정한 Bean의 자동시작을 더 섬세한 제어를 위해서는 `org.springframework.context.SmartLifecycle`을 대신 구현할 것을 고려해보아라.
>
> **또한, 중지 알림은 소멸 전에 발생할 것이라는 것을 보증하지 않는다는 것을 명심해라. 일반적으로 종료하면, 모든 Lifecycle Bean은 일반 소멸(destruction) 콜백이 전달(propagated)되기 전에 중지(stop) 알림을 첫번째로 받습니다.** 그러나 컨텍스트 수명 동안의 hot refresh 또는 중단된 refresh 시도시 destroy 메소드만 호출됩니다.

시작(startup) 및 종료(shutdown) 호출 순서가 중요할 수 있습니다. 만약 2개의 객체 사이에 depends-on 관계가 존재한다면, 의존하는 측은 이것의 의존성이 주입된 후 실행하고, 의존성이 주입되기 전에 중지(stop)된다. 그러나, 때때로(at times), 직접적인 의존성은 알려져 있지 않습니다. 특정 유형의 객체는 다른 유형의 객체보다 먼저 시작해야한다는 것을 알고있을 겁니다. 이러한 경우에 `SmartLifecycle` 인터페이스는 또다른 옵션을 지정할 수 있습니다. 즉, 슈퍼인터페이스인 `Phased`에 정의된 `getPhase()` 메소드.

다음은 `Phased` 인터페이스의 정의를 보여준다.

```java
public interface Phased{
	
   int getPhase();
   
}
```

다음은 `SmartLifecycle` 인터페이스의 정의를 보여준다.

```java
public interface SmartLifecycle extends Lifecycle, Phased {
   
   boolean isAutoStartup();
   
   void stop(Runnable callback);
}
```

시작할 때, 가장 낮은 단계의 객체가 첫번째로 실행한다. 멈출 때, 반대의 순서를 따른다. 그러므로 `SmartLifecycle`을 구현하고 `getPhase()` 메소드가 `Integer.MIN_VALUE`을 리턴하는 객체는 시작하기 위한 첫번째 객체이거나 중지하기 위한 마지막 객체 중 하나이다. 스펙트럼의 다른 쪽 끝에서, `Integer.MAX_VALUE`의 단계 값은 객체가 마지막으로 시작되고 먼저 중지되어야함을 나타낸다.(아마도 다른 실행중인 프로세스에 의존하기 때문이다.). 단계 값을 고려할 때 `SmartLifecycle`을 구현하지 않는 일반 `Lifecycle` 객체의 기본 단계는 `0`임을 알아야 하는 것이 중요하다. 따라서, 음수의 단계 값은 객체가 표준 구성 요소보다 먼저 시작함을 나타낸다. 양수의 단계 값에 대해서는 반대입니다.

**`SmartLifecycle`에 의해서 정의된 stop 메소드는 콜백을 수용합니다.** 구현의 종료 프로세스가 완료된 후 어떤 구현도 해당 콜백의 `run()` 메소드를 호출해야 합니다. **`LifecycleProcessor` 인터페이스의 기본 구현인 `DefaultLifecycleProcessor`가 각 단계의 객체 그룹이 callback을 호출하기 위해 timeout 값까지 기다리기 때문에, 필요한 경우 비동기 종료가 가능합니다.** 단계당 timeout의 기본값은 30초이다. 개발자는 context안에 lifecycleProcessor라고 명명한 Bean을 정의함으로써 기본 lifecycle 프로세서 인스턴스를 오버라이드 할 수 있다. 만약 개발자가 오직 timeout만 수정하고 싶다면, 다음과 같이 정의하면 충분하다(suffice).

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
   <!-- timeout 값(milliseconds) -->
   <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

앞에서 언급했듯이, LifecycleProcessor 인터페이스는 context의 refreshing and closing 을 위한 콜백 메소드도 정의한다. 후자는 `stop()`이 명시적으로 호출된 것처럼 종료 프로세스를 진행하지만 context가 닫힐 때 발생합니다. 반면에 refresh 콜백은 `SmartLifecycle` Bean의 또 다른 특징을 활성화합니다. context가 refresh 되어질 때(모든 객체가 인스턴스화 되고 초기화 되어지고 난 후), refresh 콜백이 호출된다. 그 시점에서(at that point), 기본 lifecycle processor는 각 `SmartLifecycle` 객체의 `isAutoStartup()` 메소드에 의해 리턴되어진 boolean 값을 체크합니다. 만약 `true`라면, 컨텍스트 또는 자신의 `start()` 메소드의 명백한 호출을 위해서 기다리는 것 대신에 해당 시점에서 객체가 시작되어집니다.(context refresh와는 다르게, context 시작은 기본 context 구현을 위해서 자동적으로 실행되지 않습니다.) `phase` 값과 "depends-on" 관계는 이전에 묘사한 것처럼 시작 순서를 결정합니다.

#### Shutting Down the Spring IoC Container Gracefully in Non-Web Applications                     (웹 어플리케이션이 아닌 곳에서의 정상적으로(Gracefully) Spring IoC Container의 종료)

> 이번 섹션은 웹 어플리케이션이 아닌 어플리케이션에서 적용할 수 있다. Spring의 웹 기반의 `ApplicationContext` 구현에는 관련된 web 어플리케이션이 종료될 때 Spring IoC Container를 정상적으로 종료하기 위한 코드가 이미 (정상적 장소에) 있습니다.

웹이 아닌 어플리케이션 환경에서(ex. rich client desktop environment) Spring의 IoC Container를 사용한다면, JVM 종료 후크(hook)를 등록하세요. 이렇게 하는 것은 정상적인 종료를 확실하게 하고, Singleton Bean에  관련된 destory 메소드를 호출하는 것을 보장합니다. 모든 자원이 release 되기 위해. 개발자는 destory 콜백을 정확히 설정하고 구현해야합니다.

종료 후크(hook)를 등록하기 위해, `configurableApplicationContext` 인터페이스에서 선언되어진 `registerShutdownHook()` 메소드를 구현해라.

다음의 예제를 참고해라.

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot{
   
   public static void main(final String[] args) thorws Exception{
   	ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
      
      // 위의 context에 대한 종료 hook 추가
      ctx.registerShutdown();
      
      // 애플리케이션 실행
      
      // 메인 메소드가 존재하고, 후크는 앱이 종료하기전에 호출된다.
   }
}
```

### 1.6.2 `ApplicationContextAware` and `BeanNameAware`

**`ApplicationContext`가 `org.springframework.context.ApplicationContextAware` 인터페이스를 구현하는 객체 인스턴스를 생성할 때, 그 인스턴스는 해당 ApplicationContext에 대한 참조를 제공받습니다.** 

다음의 예제는 AplicationContextAware 인터페이스의 정의를 보여줍니다.

```java
public interface ApplicationContextAware{
	void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

**그러므로, Bean은 `ApplicationContext` 인터페이스를 통해 또는 이러한 인터페이스(추가적인 기능을 제공하는 `ConfigurableApplicationContext` 인터페이스)의 서브클래스로 알려진 서브클래스에 대한 참조를 캐스팅하여 Bean을 작성한 ApplcationContext를 프로그래밍 방식으로 조작할 수 있다.** 한 가지의 용도는 다른 Bean을 프로그래밍 방식으로 검색하는 것이다. 이러한 방식은 때때로 매우 유용하다. 하지만, 일반적으로 이러한 방식의 코딩은 피하는 것이 좋다. 이러한 방식의 코딩은 Code를 Spring과 연결시키고, properties로써 Bean에게 collaborator를 제공하는 IoC 스타일을 따르지 않기 때문이다. `ApplicationContext`의 다른 메소드는 file resources, publishing application events, accessing a `MessageSource`에 대한 접근을 지원합니다. 이러한 추가적인 기능들은 Additional Capabilities of the `ApplicationContext`에 설명되어 있습니다.

Autowiring은 `ApplicationContext` 에 대한 참조를 얻는 또 다른 대안 입니다. 전통적인 `constructor`와 `byType` autowiring modes는 생성자의 인자 또는 setter 메소드의 인자를 위해 `ApplicationContext` 타입의 의존성을 제공할 수 있습니다. **더 유연하고 fields와 여러개의 parameter 메소드를 autowire 하기위한 기술을 포함하기 위해서, annotation 기반의 autowiring 기술을 사용해라**. 만약 이러한 기술을 사용한다면, `ApplicationContext`는 해당 필드, 생성자 또는 메서드가 `@Autowired` 애노테이션을 포함하는 경우 `ApplicationContext` 타입을 예상하는 필드, 생성자, 인스 또는 매서드 매개 변수로 Autowired 됩니다. 더 많은 정보를 원한다면, `@Autowired`를 참고하세요.

> **Autowiring이란 무엇인가?**
>
> - Autowiring은 한 Bean의 인스턴스를 또 다른 Bean의 인스턴스의 원하는 필드에 배치하여 발생합니다. 두 클래스는 모두 Bean이어야 합니다. 즉, `ApplicationContext`에 따라서 Bean들은 정의되어야 합니다.
> - Spring framework에서의 autowiring 의 기능은 개발자가 다른 객체의 의존성을 암묵적으로 주입할 수 있게끔 한다. 내부적으 setter 또는 생성자 주입을 사용한다.
>   기본 및 문자열 값을 주입하는데 Autowiring을 사용할 수 없고, reference를 주입하는데만 사용한다.

`ApplicationContext`가 `org.springframework.beans.factory.BeanNameAware` 인터페이스를 구현하는 클래스를 생성할 때, 이러한 클래스는 관련된 객체 정의에서 정의된 name에 대한 참조를 제공받습니다.

다음은 BeanNameAware 인터페이스의 정의를 보여준다.

```java
public interface BeanNameAware{
   
   void setBeanName(String name) throws BeansException;
   
}
```

콜백은 일반 Bean properties의 채운 후 `InitializingBean`, `afterPropertiesSet`, 사용자 init 메소드와 같은 초기화 콜백 전에 호출됩니다.

> **aware** : ~을 알고있는, 눈치 채고 있는

### 1.6.3 Other `Aware` Interfaces

`ApplicationContextAware`과 `BeanNameAware` 외에도, Spring은 다양한 범위의  `Aware` 콜백 인터페이스를 지원한다.
이러한 `Aware` 콜백 인터페이스는 Bean이 컨테이너에 특정 인프라 의존성이 필요함을 표시하도록하는 인터페이스이다.
일반적으로 이름은 의존성 타입을 나타냅니다.

다음은 가장 중요한  `Aware` 인터페이스를 요약한 테이블 입니다.

![image-20200127141132088](/Users/baejongjin/Library/Application Support/typora-user-images/image-20200127141132088.png)

이러한 인터페이스를 사용하는 것은 코드를 Spring API와 연결시키고 IoC 스타일을 따르지 않는다는 것을 다시한번 명심해라.
결과적으로, 컨테이너에 프로그래밍 방식으로 엑세스 해야하는 infrastructure Bean에게 권장됩니다.









































