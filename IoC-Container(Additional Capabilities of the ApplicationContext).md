## 1.15 Additional Capabilities of the `ApplicationContext`

챕터 소개에서 논의 되었던 것처럼, **`org.springframework.beans.factory` 패키지는 Beans을 관리하고 다루기 위해 프로그래밍 적인 방법들을 포함해 기본적인 기능들을 제공한다.** `org.springframework.context` 패키지는  `BeanFactory` 인터페이스를 상속하는 ApplicationContext 인터페이스를 추가하고, 이 외에도 어플리케이션 프레임워크를 지향하는 스타일로
추가적인 기능을 지원하기위해 다른 인터페이스들을 상속합니다. <u>**많은 사람들은 프로그래밍 방식으로 `ApplicationContext` 를 생성하지 않고 완전히 선언적인 방식으로 `ApplicationContext`를 사용하지만, 대신 `ContextLoader`와 같은 지원 클래스에 의존하여  Java EE web 어플리케이션의 일반적인 시작 프로세스의 부분으로써 ApplicationContext를 자동적으로 인스턴스화 한다.**</u> (ApplicationContext의 매우 중요한 내용!!)

보다 프레임워크 지향적인 스타일로 `BeanFactory` 기능을 향상시키기 위해, Context 패키지는 다음과 같은 기능을 제공해 준다.

- `MessageSource` 인터페이스를 통해서 i18n-style에서 메세지에 접근할수 있다.
- `ResourceLoader` 인터페이스를 통해서 URLs 와 files 같은 리소스에 접근할 수 있다.
- 이벤트 publication, 즉 `ApplicationEventPublisher` 인터페이스의 사용을 통해 `ApplicationListener` 인터페이스를 구현하는 Beans
- 여러 (계층적) contexts를 로딩하여, `HierarchicalBeanFactory` 인터페이스를 통하여 어플리케이션 웹 계층과 같은 하나의 특별한 계층에 각각의 contexts가 집중할 수 있게 만듭니다.

---

### 1.15.1 Internationalization(국제화) Using `MessageSource`

**`ApplicationContext` 인터페이스는 `MessageSource` 라고 불리는 인터페이스를 상속하고, 국제화(i18n) 기능을 제공한다. 또한 Spring은 계층적으로 메세지를 분석할 수 있는 `HierarchicalMessageSource` 인터페이스를 제공한다.** 이러한 인터페이스들은 Spring이 메세지 분석에 영향을 미치는 기초를 제공합니다. 이러한 인터페이스에 정의된 메소드들은 다음을 포함합니다.

- `String getMessage(String code, Object[] args, String default, Locale loc)` : `MessageSource` 로부터 메시지를 검색하기 위해 사용되어지는 기본 메소드이다. 특정한 locale에 어떠한 메세지도 존재하지 않는다면, 기본 메시지가 사용된다. 전달된 모든 인수는 표준 라이브러리에서 제공되어지는 `MessageFormat` 기능을 사용하여 대체 값이 됩니다.
- `String getMessage(String code, Object[] args, Locale loc)` : 본질적으로는 바로 위의 메소드 같지만 차이점이 한가지 있다 : 어떠한 기본 메시지도 명시되어질 수 없다. 만약 메시지가 찾을 수 없다면, `NoSuchMessageException` 예외가 던져질 것이다.
- `String getMessage(MesageSourceResolvable resolvable, Locale locale)` : 이전 메소드에서 사용되어지는 모든 properties는 이 메서드와 함께 사용할 수 있는 `MessageSourceResolvable` 라고 명명된 클래스에 쌓여져 있다.

`ApplicatioContext`가 로드되어질 때, `ApplicationContext` 는 자동적으로 Context에 정의되어진 `MessageSource` Bean을 찾는다(search for). 이러한 Bean은 무조건 `messageSource` name을 가져야 한다. **만약 이러한 Bean들이 발견되면, 이전 메소드의 모든 호출들은 message source에 위임된다.** **만약 어떠한 메세지 source도 발견되지 않는다면, `ApplicationContext` 는 동일한 이름의 Bean을 포함하는 부모를 찾을 것이다.** 만약 그렇다면, `ApplicationContext` 는 해당 Bean을 `MessageSource` 로써 사용할 것이다. 만약 `ApplicationContext`가 메시지를 위한 어떠한 source도 찾지 못한다면, 위에 정의된 메소드의 호출을 받아들일 수 있는 텅 빈 `DelegatingMessageSource`가 인스턴스화 될 것이다.

Spring은 2가지의 `MessageSource` 구현을 제공한다. 첫 번째는 `ResourceBundlemessageSource` 이고 두 번째는 `StaticMessageSource` 이다. 두개의 구현체 모두 중첩된 메시지를 처리를 수행하기 위해 `HierarchicalMessageSource`를 구현한다. **`StaticMessageSource`는 드물게 사용되지만 message를 source에 추가하기 위한 프로그램적인 방법을 제공한다.** 다음의 예제는 `ResourceBundleMessageSource`를 보여준다.

```xml
<beans>
	<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
   	<property name="basenames">
      	<list>
         	<value>format</value>
            <value>exceptions</value>
            <value>windows</value>
         </list>
      </property>
   </bean>
</beans>
```

이 예제에서는 클래스 경로에 정의된 `format`, `exceptions`, `windows` 라고 불리는 3개의 resource bundles(꾸러미, 묶음)을 가지고 있다고 가정한다. 메세지를 분석하기 위한 모든 요청은 `ResourceBundle` 객체들을 통해서 메세지들을 분석하는 JDK-표준 방법으로 다루어 진다. 예제의 목적을 위해서, 두개의 resource bundle(꾸러미, 묶음) 파일들의 내용이 다음과 같다고 가정하자.

```
# in format.properties
message=Alligators rock!
```

```
# in exceptions.properties
argument.required=The {0} argument is required
```

다음의 예제는 `MessageSource` 기능을 실행하기 위한 프로그램을 보여준다. 모든 `ApplicationContext` 구현들은 `MessageSource` 구현이고 `MessageSource` 인터페이스로 캐스트 되어질 수 있다는 것을 기억해야 한다.

```java
public static void main(String[] args){
   MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
   String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
   System.out.println(message);
}
```

위 프로그램으로 부터의 결과는 다음과 같다.

```
Alligators rock!
```

요약하자면, `MessageSource`는 classpath의 root에 존재하는 `beans.xml`이라고 불리는 파일에 정의되어 있다. `messageSource` Bean 정의는 자신의 `basenames` property를 통하여 많은 resource bundles를 참조한다. **리스트에서 `basenames` property로 전달되는 3가지의 파일들은 classpath의 root에 파일로써 존재하고 각각 `format.properties`, `exceptions.properties`, `windows.properties` 라고 불린다.**

다음의 예제는 message 조회에 전달되는 인자에 대해서 보여준다. 이러한 인자들은 `String` 객체로 변환되어지고 조회 메시지에서 placeholders에 삽입되어진다.

```xml &lt;beans&gt;    &lt;!-- this MessageSource is being used in a web application --&gt;    &lt;bean id=&quot;messageSource&quot; class=&quot;org.springframework.context.support.ResourceBundleMessageSource&quot;&gt;        &lt;property name=&quot;basename&quot; value=&quot;exceptions&quot;/&gt;    &lt;/bean&gt;    &lt;!-- lets inject the above MessageSource into this POJO --&gt;    &lt;bean id=&quot;example&quot; class=&quot;com.something.Example&quot;&gt;        &lt;property name=&quot;messages&quot; ref=&quot;messageSource&quot;/&gt;    &lt;/bean&gt;&lt;/beans&gt;
<beans>

    <!-- 이 messageSource는 웹 어플리케이션에서 사용되어진다. -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- 위의 messageSource를 해당 POJO에 주입 -->
    <bean id="example" class="com.something.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>

```

```java
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", Locale.ENGLISH);
        System.out.println(message);
    }
}
```

`execute() ` 메소드 호출로 부터의 결과는 다음과 같다.

```
The userDao argument is required.
```

**국제화("i18n")와 관련하여, Spring의 다양한 `MessageSource` 구현들은 표준 JDK ResourceBundle으로써 동일한 대체 규칙 및 locale resolution을 따른다.** 짧게 말해서, 이전에 정의되었는 `messageSource` 와 함께 계속 했을 때, 만약 개발자가message를 The British(`en-GB`) locale에 대하여 분석하고 싶다면, 개발자는 `format_en_GB.properties`, `exceptions_en_GB.properties`, `windows_en_GB.properties` 라고 불리는 각각의 파일들을 생성하면 된다.

일반적으로 locale 분석은 어플리케이션의 주변 환경에 의해서 관리되어진다. 다음의 예제에서 메시지가 분석되어지는 locale은 수동으로 명시되어집니다.

```
# in exceptions_en_GB.properties
argument.required=Ebagum lad, the {0} argument is required, I say, required.
```

```java
public static void main(String[] args){
   MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
   String message = resources.getMessage("argument.required", new Object [] {"userDao"}, "Required" Locale.UK);
   System.out.println(message);
}
```

위의 프로그램이 실행되는 것으로부터의 결과는 다음과 같다.

```
Ebagum lad, the 'userDao' argument is required, I say, required
```

개발자는 Bean이 정의되어 있는 모든 `MessageSource`에 대한 참조를 얻기 위해 `MessageSourceAware` 인터페이스를 사용할 수 있다. **`MessageSourceAware` 인터페이스를 구현하는 `ApplicationContext`에 정의된 모든 Bean들은 생성되어지고 설정 되어질 때 애플리케이션의 context의 `MessageSource`에 의해 주입되어진다.**

> `ResourceBundleMessageSource` 의 대안으로, Spring은 `ReloadableResourceBundleMessageSource` 클래스를 제공한다. 이러한 변형(대체)클래스는 동일한 bundle 파일 형식을 제공하지만 표준 JDK 기반의 `ResourceBundleMessageSource` 구현보다 더 유연하다. 특히, `ReloadableResourceBundleMessageSource` 클래스는 모든 Spring 자원 위치(classpath 뿐만 아니라)로부터 파일을 읽는 것을 허용하게 하고 bundle property 파일들의 hot 재로딩을 지원합니다(사이에서 효율적으로 캐싱하는 동안). 더 자세한 정보를 알고 싶다면 `ReloadableResourceBundleMessageSource` java document를 참고 해 보세요. 

---

### 1.15.2 Standard and Custom Events

**`ApplicationContext`의 이벤트 핸들링은 `ApplicationEvent` 클래스와 `ApplicationListener` 인터페이스를 통해서 제공되어진다.**  만약 `ApplcationListener` 인터페이스를 구현한 Bean이 context에 배치되어 있다면, `ApplicationEvent`가 `ApplicationContext`에 게시될 때마다 해당 Bean이 통지된다. 이러한 것들은 표준 Observer design pattern 이라고 한다.

> Spring 4.2 부터, 이벤트 인프라는 발전되었고 애노테이션 기반의 모델 뿐만아니라 모든 임의의 이벤트를 발생할 수 있는 능력 또한 제공한다(즉, `ApplicationEvent`로 부터 반드시 상속받을 필요가 없는 객체). 이러한 객체들이 published 될 때 우리는 개발자를 위해서 이벤트 안에 이러한 것들을 랩핑한다.

다음의 테이블을 Spring이 제공하는 표준 이벤트에 대한 설명을 나타낸 것이다.

**Table 7. Built-in Events**

| Event                        | Explanation                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| `ContextRefreshedEvent`      | **`ApplicationContext`가 초기화되어지거나 다시 초기화 되어질 때 게시된다.**(예로들어, `ConfigurableApplicatonContext` 인터페이스의 `refresh()`메소드 사용). 여기에서 "initialized" 의 의미는 모든 Bean들이 로딩되었고, post-processore Bean이 탐색되고 활성화 되었고, Singleton Bean이 미리 인스턴스화 되었고 `ApplicationContext`가 사용을 위해서 준비가 된 상태를 가르킨다. context가 소멸되지 않는한, `ApplicationContext`가 실제로 "hot" refresh를 지원하는 경우, refresh는 여러번 유발되어 질 수 있습니다. 예로들어, `XmlWebApplicationContext`는 hot refresh를 지원하지만 `GenericApplicationContext`는 지원하지 않습니다. |
| `ContextStartedEvent`        | **`ConfigurableApplicationContext` 인터페이스의  `start()` 메소드를 사용해서 `ApplicationContext`가 시작할 때 게시된다.** 여기에서, "started" 의미는 모든 `Lifecycle` Beans이 명확한 시작 신호를 받을 수 있다는 것을 의미한다. 일반적으로, 중단 한 후에 Bean을 재시작하기 위해서 이러한 신호는 사용되지만 자동 시작을 위해 설정되지 않는 Components를 시작하는데 사용되어질 수도 있습니다(예로들어, 초기화시 시작되지 않는 components) |
| `ContextStoppedEvent`        | **`ConfigurableApplicationContext` 인터페이스에서 `stop()` 메소드를 사용하여 `ApplicationContext`가 중단될 때 게시된다.** 여기서 "stopped" 의미는 모든 `Lifecycle` Bean들은 명확한 stop 신호를 받을 수 있다는 것을 의미한다. 중단된 context는 `start()` 호출을 통해서 재시작되어질 수 있습니다. |
| `ContextClosedEvent`         | **`ConfigurableApplicationContext` 인터페이스의 `close()` 메소드를 사용함으로써 또는 JVM shutdown hook을 통해서 `ApplicationContext`가 닫혀질 때 게시된다.** 여기에서 "closed"의 의미는 모든 Singleton Bean들은 소멸되어 질 수 있다는 것을 의미한다. Context가 닫혀지면, 수명이 다하여 새로 고치거나 다시시작할 수 없습니다. |
| `RequestHandledEvent`        | **모든 Beans에 HTTP 요청이 서비스 되었음을 말하는 웹에 특화된 이벤트이다.** 이 이벤트들은 요청이 완료된 후에 생성된다. 이 이벤트는 Spring의 `DispatcherServlet`을 사용하는 웹 어플리케이션에만 오직 적용가능하다. |
| `ServletRequestHandledEvent` | **서블릿에 특화된 context 정보를 추가한 `RequestHandledEvent`의 서브클래스이다.** |

**개발자는 사용자 정의 이벤트를 생성하고 게시(publish)할 수 있다.** 다음의 예제에서는 Spring의 `ApplicationEvent` 기본 클래스를 상속하는 간단한 클래스를 보여준다.

```java
public class BlackListEvent extends ApplicationEvent {
	private final String address;
   private final String content;
   
   public BlackListEvent(Object source, String address, String context){
      super(source);
      this.address = address;
      this.content = content;
   }
   
   // accessor and other methods ...
}
```

사용자 정의 `ApplicationEvent`를 게시하기 위해서, `ApplicationEventpublisher` 의 `publishEvent()` 메소드를 호출해야 한다. 일반적으로, 이러한 것들은 `ApplicationEventPublisherAware` 을 구현한 클래스를 생성하고 Spring Bean으로써 `ApplicationEventPublisher`를 등록하면 수행할 수 있다.  다음은 이러한 클래스의 예제를 보여준다.

```java
public class EmailService implements ApplicationEventPublisherAware {
   
   private List<string> blackList;
   private ApplicationEventPublisher publisher;
   
   public void setBlackList(List<String> blackList){
      this.blackList = blackList;
   }
   
   public void setApplicationEventPublisher(ApplicationEventPublisher publisher){
      this.publisher = publisher;
   }
   
   public void sendEmail(String address, String content) {
      if(blackList.contains(address)) {
         publisher.publishEvent(new BlackListEvent(this, address, content));
         return;
      }
      // send email...
   }
   
}
```

설정 시간에, Spring Container는 `EmailService`가 `ApplicationEventPublisherAware`를 구현하는 것과 자동적으로 `setApplicationEventPublisher()`를 호출하는 것을 감지한다. 실제로 전달된 매개변수는 Spring Container 자체입니다. 개발자는 Application Context의 `ApplicationEventPublisher` 인터페이스를 통해서 상호작용 합니다.

사용자 정의 `ApplicationEvent`를 수용하기 위해서, 개발자는 `ApplicationListener`를 구현한 클래스를 생성할 수 있고 Spring Bean으로써 등록할 수도 있습니다. 다음의 예제는 이러한 클래스의 예시를 보여줍니다.

```java
public class BlackListNotifier implements ApplicationListener<BlackListEvent> {
   
   private String notificationAddress;
   
   public void setNotificationAddress(String notificationAddress) {
      this.notificationAddress = notificatonAddress;
   }
   
   public void onApplicationEvent(BlackListEvent event) {
      // notificationAddress를 통해 적절한 객체(parties)에게 알립니다.
   }
}
```

**`ApplicationListener`는 일반적으로(generically) 사용자 이벤트 타입으로 매개 변수화(parameterized) 된다는 사실을 명심해야 합니다.** (이전 예제에서는 `BlackListEvent`) 이러한 사실은 `onApplicationEvent()`는 **type-safe**를 할 수 있고 다운캐스팅을 위한 어떠한 요구도 피한다는 것을 의미합니다. **개발자는 원하는대로 많은 이벤트 리스너를 등록할 수 있지만 기본적으로 이벤트 리스너는 이벤트를 동기적으로 받는다는 사실을 알고 있어야 합니다.** 이것은 모든 리스너가 이벤트 처리를 완료할 때 까지 `publishEvent()` 메소드가 차단됨을 의미합니다. 이러한 동기적이고 싱글 스레드 접근방법의 한가지 장점은 리스너가 이벤트를 수용했을 때 트랜잭션 context가 사용가능 한 경우 리스너가 publisher의 트랙잭션 컨텍스트 내에서 작동한다는 것입니다. 이벤트 publication에 관한 또 다른 전략이 필요하다면, Spring의 `ApplicationEventMulticaster` 인터페이스와 설정 옵션을 위한 `SimpleApplicationEventMulticaster` 구현을 위해 java document를 참조 해 보세요.

다음 예제는 위의 클래스 각각을 설정하고 등록하는데 사용된 XML 기반의 Bean정의 예제입니다.

```xml
<bean id="emailService" class="example.EmailService">
	<property name="blackList">
   	<list>
      	<value>known.spammer@example.org</value>
         <value>known.hacker@example.org</value>
         <value>john.doe@example.org</value>
      </list>
   </property>
</bean>

<bean id="blackListNotifier" class="example.BlackListNotifier">
	<property name="notificationAddress" value="blacklist@example.org"/>
</bean>
```

`emailService` Bean의 `sendEmail()` 메소드가 호출 될 때 블랙리스트에 올릴 이메일 메시지가 있으면 `BlackListEvent` 유형의 사용자 정의 이벤트가 게시됩니다. `blackListNotifier` Bean은 `ApplicationListener`로써 등록되어지고 `BlackListEvent`를 수용하여, 해당 당사자에게 알려질 수 있습니다.

> Spring의 이벤팅 방법은 동일한 ApplicationContext 내에서 Spring Beans 간에 간단한 의사소통을 위해서 디자인 되었습니다. 그러나 더 복잡한(sophisticated) 기업 통합 요구를 위해, 별도로 유지관리되는 Spring 통합 프로젝트는 경량, 패턴 중심, 잘 알려진 Spring 프로그래밍 모델을 기반으로 하는 이벤트 중심의 아키텍처를 위해 완벽한 지원을 제공합니다.

> **What is The Java method Signature?**
>
> **Java Signature is the Combination of the method name and the parameter list.**
>
> <img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200210214457782.png" alt="image-20200210214457782" style="zoom:50%;" />

#### Annotation-based Event Listeners

**Spring 4.2 부터, 개발자는 `@EventListner` 애노테이션을 사용함으로써 관리되는 Bean의 public 메소드에 이벤트 리스너를 등록할 수 있습니다.** `BlackListNotifier`는 다음과 같이 다시 쓰여질 수 있습니다.

```java
public class BlackListNotifier {
   
   private String notificationAddress;
   
   public void setNotificationAddress(String notificationAddress){
      this.notificationAddress = notificationAddress;
   }
   
   @EventListener
   public void processBlackListEvent(BlackListEvent event) {
      // notificationAddress를 통해서 적절한 당사자에게 알립니다.
   }
}
```

**이 method signature는 listen하는 이벤트 타입을 유연한 이름과 특정한 리스너 인터페이스의 구현 없이 다시 선언합니다.** 실제 이벤트 유형이 구현 계층에서 제네릭 인자를 분석하는 한 제네릭을 통해서 이벤트 유형은 범위를 좁힐 수 있습니다.

메소드가 여러 이벤트를 리스닝 해야하거나 또는 전혀(at all) 어떠한 인자도 없이 리스너를 정의하는 것을 원한다면, 이러한 이벤트 타입은 애노테이션 자체에서 정의되어 질 수 있습니다. 다음의 예제는 어떻게 사용하는지에 대해서 알려주는 예제 입니다.

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
   // ...
}
```

이러한 이벤트 리스너는  애노테이션의 `condition` 속성을 `SpEL` 표현식으로 정의해서 사용함으로써 추가적인 런타임 필터링을  추가할 수 있습니다.  spEL 표현식은 특정한 이벤트를 위한 메소드의 실질적으로 호출하기 위해 일치해야 합니다.

다음의 예제는 이벤트의 `content` 속성이 `my-event`와 동일하다면 어떻게 notifier를 호출되어지도록 재 작성되는지에 대해서 보여준다.

```java
@EventListener(condition="#blEvent.content == 'my-event'")
public void processBlackListEvent(BlackListEvent blEvent) {
	// notificationAddress를 통해서 적절한 당사자에게 알립니다.   
}
```

각 SpEL 표현식은 전용(dedicated) 컨텍스트에 대해 평가됩니다. 다음은 선택적인 이벤트 프로세싱을 위하여 이러한 것들을 사용하기 위해서 context에서 사용가능하도록 만들어진 항목들에 대해서 나열한 테이블이다.

**Talbe 8. Event SpEL available metadata**

| Name            | Location           | Description                                                  | Example                                                      |
| --------------- | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Event           | root object        | 실질적인 `ApplicationEvent`                                  | `#root.event` 또는 `event`                                   |
| Arguments array | root object        | 메소드를 호출하기 위해서 사용되어지는 인자(객체 배열 로써)   | `#root.args` 또는 `args`; 첫 번째 인자를 액세스 하기 위한`args[0]`, ... |
| Argument name   | evaluation context | 모든 메소드 인자의 name. 어떠한 이유로든 name을 사용할 수 없는 경우 (예, 컴파일 된 바이트 코드에 디버그 정보가 없기 때문에) 개별 인수 `#a<#arg>` 구문을 사용하여 이용할 수도 있습니다. 여기서 `<#arg>`는 인수의 인덱스를 나타냅니다(stand for).(0부터 시작) | `#blEvent` 또는 `#a0` ( `#p0` 또는 `#p<#arg>` 인자 표기법을 별명으로써 사용할 수 있다.) |

method signature가 게시된 임의의 객체를 실제로 참조하더라도, `#root.event`를 사용해서 unerlying event에 접근할 수 있습니다.

만약 다른 이벤트의 처리 결과로써 이벤트를 게시할 필요가 있다면, 개발자는 게시되어야만 하는 이벤트를 리턴하기 위해 method signature를 바꿀 수 있다. 다음 예제에서 보여준다.

```java
@EventListener
public ListUpdateEvent handleBlackListEvent(BlackListEvent event) {
	//notificationAddress를 통해서 적절한 당사자에게 전달된다 그리고 ListUpdateEvent를 게시합니다.
}
```

> 이러한 기능은 asynchronous listeners를 위해서는 지원되지 않습니다.

이러한 새로운 메소드는 위의 메소드로 핸들링 되어지는 모든 `BlackListEvent`를 위해서 새로운 `ListUpdateEvent`가 게시됩니다. 만약 개발자가 여러개의 이벤트를 게시하고 싶다면, 이벤트의 `Collection`을 리턴할 수도 있습니다.

#### Asynchronous Listeners

만약 이벤트를 비동기적으로 처리하는 특정 리스터가 필요하다면, 개발자는 일반적인 `@Async` 지원을 다음 예제와 같이 재 사용 할 수 있습니다.

```java
@EventListener
@Async
public void processBlackListEvent(BlackListEvent event){
   // 별도의(separate) 스레드에서 BlackListEvent가 처리되어집니다.
}
```

비동기적 이벤트를 사용할 때 다음의 제한에 대해서는 알아야 합니다.

- 만약 비동기전인 에러가 `Exception`을 던진다면, 이러한 예외는 호출자에게 전파되어(propagated)지지 않습니다. 더 많은 정보를 위해서 `AsyncUncaughtExceptionHandler`에 대해서 참조하세요.
- 비동기적 이벤트 리스너 메소드들은 값을 리턴해서 순차적인 이벤트 게시가 불가능합니다. 만약 다른 이벤트의 처리 결과로 또 다른 이벤트를 게시하고 싶다면, 수동적으로 이벤트를 게시하기 위해서 `ApplicationEventPublisher`의 의존성을 주입하세요

#### Ordering Listeners

하나의 리스턴가 다른 리스너가 호출 되기전에 먼저 호출되고 싶다면, 개발자는 `@Order` 애노테이션을 메소드 선언에 추가할 수 있습니다. 다음 예제에서 보여줍니다.

```java
@EventListener
@Order(42)
public void processBlackListEvent(BlackListEvent event) {
   // notificationAddress를 통해서 적절한 당사자에게 통보한다.
}
```

#### Generic Events

**개발자는 제네릭을 사용해서 이벤트 구조를 추가적으로 정의할 수 있습니다.** `EntitiyCreatedEvent<T>`를 사용하는 것을 고려 해보아라. `EntitiyCreatedEvent<T>` 에서 `T`는 실제 생성되는 엔티티의 타입을 의미한다. 예로들어, `Person` 을 위한 오직 `EntitiyCreatedEvent`를 받기 위한 다음의 정의를 생성할 수 있습니다.

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
   // ...
}
   
```

타입 삭제 때문에, 발생하는 이벤트가 이벤트 리스너가 필터링하는 제네릭변수를 분석하는 경우에만 위의 이벤트 리스너가 작동한다. (즉, `class PersonCreatedEvent extends EntitiyCreatedEvent<Person>{...}`).

특정한 환경에서, 모든 이벤트들이 같은 구조를 따른다면(앞의 예제에서 이벤트의 경우와 마찬가지로) 이러한 것들은 꽤 지루할(tedious) 것이다. **이러한 경우에, 런타임 환경이 제공하는 것 이상으로(beyond) 프레임 워크를 가이드하기 위해 `ResolvableTypeProvider`를 구현할 수 있습니다.** 다음 예제에서 어떻게 구현하는지에 대해서 알려줍니다.

```java
public class EntitiyCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {
   
   public EntitiyCreatedEvent(T entitiy) {
      super(entity);
   }
   
   @Override
   public ResolvableType getResolvableType() {
      return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getSource()));
   }
}
```

> 이것은 ApplicationEvent 뿐만 아니라 이벤트로써 보내지는 임의의 객체에서도 작동합니다.

---

### Conveinet Access to Low-level Resources

application contexts의 이해와 최선의 사용을 위해서, 개발자는 Spring의 `Resource` 추상화와 친해져야 한다. Resources에서 묘사되어진다.

Application context는 `ResourceLoader` 이다. `ResourceLoader`는 `Resource` 객체를 로드하기 위해 사용되어진다. `Resource`는 본질적으로 `java.net.URL` 클래스의 기능이 더 풍부한 버전입니다. 사실, `Resource` 클래스의 구현은 `java.net.URL` 인스턴스를 적절하게 랩핑합니다. **`Resource`는 classpath, 파일 시스템 위치, 표준 URL로 설명이 가능한 모든곳,  기타 변형 등 거의 모든 위치에서 저수준 리소스를 얻을 수 있습니다.** 만약 resource location String이 특별한 접두사 없이 매우 간단한 경로라면, 이러한 경로로 부터 오는 자원들은 실제의 application context type에 적절하고 구체적이다.

**특별한 콜백 인터페이스, `ResourceLoaderAware`를 구현하기 위해 application context에 배치되어있는 Bean들을 설정할 수 있고 application context 자체가 `ResourceLoader`로써 전달되어 초기화시에 자동으로 다시 호출됩니다.** 개발자는 정적 리소스에 액세스하는데 사용되어지는 `Resource` 타입의 properties를 노출시킬 수 있다. 그들은 다른 속성과 마찬가지로 주입 되어집니다. 개발자는 간단한 `String` 경로로써 이러한 `Resoure` properties를 명시할 수 있고, Bean이 배치되어질 때 이러한 text String으로부터 실제  `Resource` 객체로의 자동 변환에 의존할 수 있습니다.

`ApplicationContext` 생성자에 제공되어지는 경로 또는 위치 경로들은 실제 리소스 문자열이고, 특별한 context 구현에 따라서 적절하게 다루어 집니다. 예로들어, `ClassPathXmlApplicationContext` 는 classpath location과 같은 간단한 위치 경로를 다룹니다. **실제 context 타입에 상관 없이 classpath 또는 URL로부터 정의를 로드하기 위하여 특별한 접두사를 붙인 위치 경로를 사용할 수 있습니다.**

---

### 1.15.4 Convenient ApplicationContext Instantiation for Web Applications

예로들어, 개발자는 `ContextLoader`를 사용함으로써 `ApplicationContext` 인스턴스를 선언적으로 생성할 수 있다. 물론 `ApplicationContext` 구현 중에서 하나를 사용함으로써 `ApplicationContext`를 프로그램적으로 생성할 수 있다.

개발자는 `ContextLoaderListener`를 사용함으로써 `ApplicationContext`를 등록할 수 있다. 다음 예제에서 보여준다.

```xml
<context-param>
	<param-name>contextConfigLocation</param-name>
   <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

리스너는 `contextConfigLocation` 인자를 검사한다. 만약 인자가 존재하지 않다면, 릴스너는 기본적으로 `/WEB-INF/applicationContext.xml`을 기본적으로 사용한다. parameter가 존재한다면, 리스너는 미리 정의된 delimters(comma, semicolon, and whitespace)를 사용해서 `String`을 분리하고, application contexts가 검색하는 위치로써 찾은 값들을 사용한다. Ant-style path patterns는 잘 지원된다. 예로들어, `/WEB-INF/*Context.xml`( `Context.xml`로 이름이 끝난 모든 파일) 과 `/WEB-INF/**/*Context.xml`( `WEB-INF` 의 하위 디렉토리에 있는 Context.xml로 끝나는 모든 파일)

---

### 1.15.5 Deploying a Spring `ApplicationContext` as a Java EE RAR File























 