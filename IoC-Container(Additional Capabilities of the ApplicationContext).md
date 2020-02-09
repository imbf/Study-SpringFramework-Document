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









