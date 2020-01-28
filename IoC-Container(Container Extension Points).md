## 1.8 Container Extension Points

일반적으로, 어플리케이션 개발자들은 `ApplicationContext`를 구현하는 서브 클래스가 필요하지 않습니다. 대신에 Spring IoC Container는 특별 통합 인터페이스를 구현을 플러그인 함으로써 확장되어진다. 다음의 몇가지 섹션은 이러한 통한 인터페이스에 관한 설명이다.

### 1.8.1 Customizing Beans by Using a `BeanPostProcessor`

**`BeanPostProcessor` 인터페이스는 사용자 인스턴스화 로직(오버라이드 가능), 의존성 해결 로직 등을 제공하기 위해 구현가능한 콜백 메소드를 정의한다. Spring Container가 인스턴스화, 설정, Bean 초기화를 끝낸다음 사용자가 정의한 로직을 실행하는 것을 원한다면, 하나 이상의 사용자 정의 `BeanPostProcesoor` 구현을 플러그인 할 수 있습니다.** 

개발자는 여러개의 `BeanPostProcessor` 인스턴스를 설정할 수 있고, `order` property를 설정함으로써 `BeanPostProcessor` 인스턴스가 실행하는 순서를 제어할 수 있습니다. `BeanPostProcessor`가 `Ordered` 인터페이스를 구현하는 경우에만 이러한 property를 설정할 수 있습니다. 만약 사용자 정의 `BeanPostProcessor`를 작성했다면, 개발자는 `Ordered` 인터페이스를 실행할 것을 고려해야한다.

더 많은 정보를 원한다면, javadoc의 `BeanPostProcessor`와 `Ordered` 인터페이스를 참고해보아라. 또한 programmatic registration of `BeanPostProcessor` 인스턴스를 참고해보아라.

> `BeanPostProcessor` 인스턴스는 Bean 또는 객체 인스턴스에서 동작한다. 즉, Spring IoC Container는 Bean 인스턴스를 인스턴스화 하고, `BeanPostProcessor` 인스턴스가 다음 자신의 작업을 진행한다.
>
> `BeanPostProcessor` 인스턴스는 Container 마다 범위가 지정 되어있다. 이러한 사실은 개발자가 Container 계층(hierarchies)를 사용하는 경우에만 관련있다. 만약 하나의 Container에 `BeanPostProcessor`를 정의했다면, 해당 Container내의 Bean에게만 사후처리 합니다. **다르게 말하면, 한 컨테이너에서 정의된 Bean들은 또 다른 Container에서 정의된 ` BeanPostProcessor`에 의해서 사후처리 되지 않습니다.** 두개의 컨테이너가 같은 계층의 부분이라고 하더라도 마찬가지 입니다.
>
> 실질적인 Bean 정의(즉 Bean을 정의하는 blueprint)를 바꾸기 위해서, 개발자는 `BeanFactoryPostProcessor`를 사용해야 한다. Customizing Configuration Metadata with a BeanFactoryPostProcessor 에 설명되어 있다.

`org.springframework.beans.factory.config.BeanPostProcessor` 인터페이스는 정확히 2개의 callback 메소드로 이루어져 있다. **이러한 클래스가 Container에 post-processor로써 등록이 되어져 있을 때, Container에 의해서 생성되어지는 각 Bean 인스턴스에 대하여, post-processor는 Container 초기화 메소드(ex. `InitializingBean.afterPropertiesSet()`, `init` 메소드로 선언되어진 메소드 등) 가 호출되기 이전에, 다른 Bean 초기화 콜백이 호출된 이후에 콜백을 얻는다.** post-processor는 콜백을 완전히 무시하는 것을 포함하여 Bean 인스턴스에 모든 조치를 취할 수 있습니다. Bean post-processor는 일반적으로 콜백 인터페이스를 확인하거나, 프록시로 Bean을 감쌀 수 있습니다. 몇가지의 Spring AOP infrastructure 클래스들은 Bean post-processor로써 proxy-wrapping logic을 제공하기 위해서 구현되어 있다.

`ApplicationContext`는 자동적으로 `BeanPostProcessor` 인터페이스를 구현한 configuration data에 의해 정의된  모든 Bean들을 탐색합니다. `ApplicationContex` 는 이러한 Bean들을 생성되지마자 나중에 호출될 수 있도록 post-processor로써 등록합니다. Bean post-processor는 다른 Bean과 마찬가지로 같은 방식으로 Container에 배치합니다.

**configuration 클래스에 `@Bean` 팩토리 메소드를 사용해서 BeanPostProcessor를 선언할 때, 리턴 타입은 구현 클래스 자체이거나 또는 최소 `org.springframework.beans.factory.config.BeanPostProcessor`인터페이스 이어야 합니다.** 그리고 명확하게 해당 Bean의 post-processor의 특징을 나타냅니다. 그렇지 않으면(otherwise) `ApplicationContext`는 완전하게 이것이 생성되기 전에 자동으로 BeanPostProcessor를 타입에 의해서 찾지 못합니다. `BeanPostProcessor`는 context에서 다른 Bean의 초기화에 적용하기 위해 일찍 인스턴스가 되어야 하기 때문에, 이른 타입 검색은 매우 중요합니다.

> **프로그램적으로 `BeanPostProcessor` 인스턴스 등록**
>
> `ApplicationContext` 자동 탐색을 통하여 `BeanPostProcessor`를 등록하는 권장된 접근방법 이외에, 개발자는 `addBeanPostProcessor` 메소드를 사용하여 `ConfigurableBeanFactory`에 프로그램저긍로 `BeanPostProcessor`를 등록할 수 있다. 등록하기 전에 선택적인 로직을 평가 할때 또는 계층의 context 전역에서 Bean post processor를 복사하는 경우에도 유용하게 사용할 수 있습니다. 그러나 프로그래밍 방식으로 추가된 `BeanPostProcessor` 인스턴스는 `Ordered` 인터페이스를 중요시 하지 않게 여기는 것을 주목해야 합니다. **이러한 프로그래밍 방식에서는, 등록 순서는 실행 순서를 지시(dictate)합니다.** 프로그래밍 방식으로 등록된 `BeanPostProcessor` 인스턴스는 명백한 순서에 관계없이 자동 탐색을 통해 이러한 것들이 등록되기 전에 미리 처리됩니다.



> **`BeanPostProcessor` 인스턴스와 AOP(Aspect Oriented Programming) 자동 프록시**
>
> `BeanPostProcessor` 인터페이스를 구현하는 클래스들은 특별하고, Container에 의해서 다르게 다루어진다.  **직접 참조하는 모든 `BeanPostProcessor` 인스턴스 및 Bean들은 `ApplcationContext`의 특수 시작 단계의 일부로써 시작시 인스턴스화 됩니다. 다음으로 모든 BeanPostProcessor 인스턴스는 정렬 된 방식으로 등록되어 Container의 모든 추가 Bean에게 적용됩니다.** AOP auto-proxying은 `BeanPostProcessor` 으로서 구현되어져 있기 때문에, `BeanPostProcessor` 인스턴스나 직접 참조하는 Bean은 자동 프록시에 자격이 없습니다. 이렇게 짜여진(weave into) 측면을 가지지 않습니다.
>
> 이러한 Bean의 경우, 정보 로그 메세지가 표시되어야 합니다 : Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying).
>
> 만약 `BeanPostProcessor`에 Bean을 autowiring을 사용하여 또는 `@Resource`(autowiring으로 대채될 수 있음)를 사용하여 연결하려고 하는 경우에, Spring은 type 일치 의존성 후보를 검색할 때 예기치 않은 Bean에 액세스 할 수 있도록 auto-proxing 또는 다른 종류의 Bean post-processing을 적합하지 않도록 만듭니다. 예로들어, `@Resource` 애노테이션이 달린 의존성이 있고 filed나 setter name이 Bean의 선언된 name과 직접적으로 일치하지 않고 name 속성이 사용되지 않는 경우 Spring은 다른 Bean에 엑세스하여 타입에 의해 일치시킵니다.

다음은 `ApplcationContext`에서 `BeanPostProcessor` 인스턴스를 어떻게 작성하고, 등록하고, 사용하는지에 대한 예제이다.

#### Example: Hello World, `BeanPostProcessor`-style

이번 첫 번째 예지는 기본적인 사용을 설명합니다. 이 예제는 Container에 의해서 각 Bean이 생성될 때 `toString( )` 메소드를 호출하하고 System 콘솔의 String 결과 값을 print하는 사용자 `BeanPostProcessor` 구현을 보여줍니다.

사용자 `BeanPostProcessor` 구현한 클래스의 definition을 보여줍니다.

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor{
   
   // 인스턴스화된 빈을 그대로(as-is) 반환
   public Object postProcessBeforeInitialization(Object bean, String beanName){
      return bean;	// 우리는 어느 타입의 객체 참조도 반환할 수 있다.
   }
   
   public Object postProcessAfterInitialization(Object bean, String beanName){
      System.out.println("bean '" + beanName + "'created : " + bean.toString());
      return bean;
   }
}
```

다음의 `beans` 요소는 `InstantiationtracingBeanPostProcessor` 를 사용합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:lang="http://www.springframework.org/schema/lang"
       xsi:schemaLoction="http://www.springframework.org/schema/beans
                          https://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/lang
                          https://www.springframework.org/schema/lang/spring-lang.xsd">
	<lang:groovy id="messenger" script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
   	<lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
   </lang:groovy>
   
   <!-- 위의 Bean(메신저)가 인스턴스화 될 때, 사용자 BeanPostProcessor구현은 사실을 시스템 콘솔에 출력한다 -->
   <bean class="scripting.InstancetiationTracingBeanPostProcessor"/>
   
</beans>
```

어떻게 `InstantiationTracingBeanPostProcessor`가 정의되는지 주목하십시요. 심지어는 이름도 없고, Bean이기 때문에 다른 Bean에게 의존성 주입이 되어질 수 있습니다. (우의 구성 또한 Groovy script가 지원하는 Bean을 정의합니다. Spring 동적 언어 지원에 대해서는 Dynamic Language Support장에 자세히 설명되어 있습니다.)

다음의 자바 어플리케이션은 이전의 코드와 설정을 실행합니다.

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot{
   
   public static void main(final String[] args) throws Exception{
      ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
      Messenger messenger ctx.getBean("messenger".Messenger.class);
      System.out.println(messenger);
   }
}
```

위의 어플리케이션의 출력은 다음과 유사(resemble)합니다.

```
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```

#### Example: The `RequiredAnnotationBeanPostProcessor`

사용자 `BeanPostProcessor` 구현과 함께(in conjunction with) 콜백 인터페이스 또는 애노테이션을 사용하는 것은 Spring IoC Container를 확장하는 일반적인 수단(mean)이다. 하나의 예시는 Spring의 `RequiredAnnotationBeanPostProcessor` - Spring 배포판과 함께 제공되고, 임의의 주석으로 표시된 Bean의 JavaBean 특성이 값으로 종속성이 실제 주입되도록 구성하는 `BeanPostProcessor` 구현 

### 1.8.2 Customizing Configuration Metadata with a `BeanFactoryPostProcessor`

다음으로 우리가 주목해야 할 확장 포인트는 `org.springframework.beans.factory.config.BeanFactoryPostProcessor` 입니다. 이 인터페이스의 의미는 `BeanPostProcessor`와 비슷하다. `BeanFactoryPostProcessor`의 가장 중요한 차이는 Bean configuration metadata에서 작동합니다. 즉, Spring IoC Container는 `BeanFactoryPostProcessor`가 configuration metadata를 읽도록 하고, 잠재적으로 `BeanFactoryPostProcessor` 인스턴스 이외의 다른 Bean을 인스턴스화하기 전에 이를 변경할 수 있습니다.

개발자는 여러개의 `BeanFactoryPostProcessor` 인스턴스를 설정할 수 있고, `Order` 프로퍼티를 설정함으로써 `BeanFactoryPostProcessor` 인스턴스의 실행 순서를 제어할 수 있습니다. 그러나 `BeanFactoryPostProcessor`가 `Ordered` 인터페이스를 구현해야지만 이러한 proeprty를 세팅할 수 있습니다. 만약 개발자가 자신의 `BeanFactoryPostProcessor`을 작성한다면, `Ordered` 인터페이스를 구현할 것을 고려하기를 추천한다. 

더 자세한 정보를 얻고 싶다면 java document의 `BeanFactoryPostProcessor` 와 `Ordered` 인터페이스를 참고해보자!!!

> 만약 실제 Bean 인스턴스(configuration metadata로부터 생성되는 객체)를 바꾸길 원한다면, `BeanPostProcessor`를 사용할 필요가 있다. `BeanFactoryPostProcessor` 내에서 Bean 인스턴스로 작업하는 것은 기술적으로 가능하지만(예로들어 BeanFactory.getBean()), 이러한 작업들은 미숙한 Bean 인스턴스화, 표준 Container lifecycle 위반등을 야기시킨다. 이러한 것들은 Bean Post Processing 우회화 같은 부정적인 결과를 야기시킬 수 있다.
>
> 또한, `BeanFactoryPostProcessor` 인스턴스는 Container마다 범위가 지정되어 있다. 만약 개발자가 Container 계층들을 사용한다면 관련이 있다. 하나의 Container에 `BeanFactoryPostProcessor` 를 정의한다면, 오직 해당 Container의 Bean 정의에 적용되어질 것이다. 한 Container의 Bean 정의는 다른 Container의 `BeanFactoryPostProcessor`에 의해 사후처리 되지 않는다. 비록 두개의 컨테이너가 같은 계층의 부분이라고 하더라도.

**Bean factory post-processor는 `ApplicationContext` 내에서 선언될 때 Container를 정의하는 configuration metadata에 변경사항을 적용하기 위해  자동으로 실행됩니다.** Spring은 많은 수의 미리 정의된 Bean factory post-processors(`PropertyOverrideConfigurer` and `PropertySourcesPlaceholderConfigurer`)를 포함한다. 개발자는 사용자 `BeanFactoryPostProcessor`를 사용할 수 있습니다. - 예로들어, 사용자 property editors를 사용함으로써

`ApplicationContext` 는 `BeanFactoryPostProcessor` 인터페이스를 구현하는 모든 Bean을 자동으로 감지합니다. `ApplicationContext`는 이러한 Bean들을 bean factory post-processor로써 적절한 시간에 사용합니다. 개발자는 이러한 post-processor Bean을 개발자가 원하는 어느 Bean에 배치할 수 있습니다.

> `BeanPostProcessor`와 마찬가지로(As with), 개발자는 `BeanFactoryPostProcessor` 늦게 초기화 하도록 설정하는 것을 원하지 않을 것이다. 만약 어떠한 Bean도 `Bean(Factory)PostProcessor` 를 참조하지 않는다면, 해당 post-processor는 전혀 인스턴스화 되지 않을 것 입니다. 따라서, post-processor 를 lazy initialization하는 것은 무시되어 질 것입니다. 그리고 `<beans/>`요소의 `default-lazy-init`속성을 `true`로 설정 하더라도 `Bean(Factory)PostProcessor`는 열렬히 인스턴스화될 것입니다.

#### Example: The Class Name Substitution `PropertySourcesPlaceholderConfigurer`

**개발자는 표준 Java `Properties` 형식을 활용하여 별도의 파일의 Bean 정의로부터 property 값을 외부화하기 위해서 `PropertySourcesPlaceholderConfigurer`를 사용할 수 있다.** 이렇게 하는 것은 어플리케이션을 배포하려는 사람이 특정한 환경의 properties를  주요 XML 정의 파일 또는 Container를 위한 파일을 수정하는 위험성과 복잡성 없이 주문제작할 수 있다. (ex. database URL, passwords)

다음의 XML 기반의 configuration metadata 부분(placeholder 값으로 정의된 Datasource)을 고려해보자.

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
	<property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close" class="org.apache.commons.dbcp.BasicDataSource">
   <property name="driverClassName" value="${jdbc.driverClassName}"/>
   <property name="url" value="${jdbc.url}"/>
   <property name="username" value="${jdbc.username}"/>
   <property name="password" value="${jdbc.password}"/>
</bean>
```

이 예제는 외부 `Properties` file로 부터 설정된 properties를 보여준다. 런타임시에, `PropertySourcesPlaceholderConfigurer`는 Datasource의 몇가지의 properties를 교체한 metadata가 적용된다. 교체된 값은 placeholder 폼으로 명시되어 진다. 이러한 폼은 다음과 같다. `${property-name}`, 이러한 형태의 폼은 Ant, log4j, JSP EL 스타일을 따릅니다.

표준 Java `Properties` 형식의 다른 파일로부터 온 실제 값은 다음과 같다.

```
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

그러므로, `${jdbc.username}` String은 런타임시에 'sa' 값으로 교체되어지고, properties 파일의 키와 일치하는 다른 placeholder values에도 동일하게 적용됩니다. `PropertySourcesPlaceholderConfigurer`는 Bean 정의의 대부분의 properties 와 속성에서 placeholder를 점검합니다. 게다가, 개발자는 placeholder를 prefix 와 suffix로 설정할 수 있습니다.

`context` namespace가 Spring 2.5에서 소개되어 지면서, 개발자는 property placeholder를 특정(dedicated) 설정 요소를 사용해서 설정할 수 있습니다. 개발자는 `location` 속성에서 comma로써 구분된 리스트로서 하나 이상의 locations를 제공할 수 있습니다. 다음 예제를 참고해 보자.

```xml
<context:property-placeholder location="classpath:com/something/jdbc.properties"/>
```

`PropertySourcesPlaceholderConfigurer` 는 정의한 `Properties` 파일에서 properties를 찾을 뿐만 아니라, 기본적으로 특정한 properties files안에서 property를 찾지 못하면, Spring `Environment` properties 와 기본적인 자바 `System` properties를 점검한다.

> `PropertySourcesPlaceholderConfigurer`를 사용하여 클래스 이름을 대체할 수 있습니다. 이는 런타임시 특정 클래스를 선택해야 할 때 유용합니다. 다음의 예는 어떻게 사용하는지에 대해서 보여준다.
>
> ```xml
> <bean class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">
> 	<property name="locations">
>    	<value>classpath:com/something.strategy.properties</value>
>    </property>
>    <property name="properties">
>    	<value>custom.strategy.class=com.something.DefaultStrategy</value>
>    </property>
> </bean>
> 
> <bean id="serviceStrategy" class="${custom.strategy.class}"/>
> ```
>
> 클래스가 런타임시에 유효한 클래스를 확인할 수 없다면, Bean의 확인은 Bean이 생성되어질 때 실패합니다. 이러한 경우는   non-lazy-init Bean을 위한 `ApplicationContext`의 `preInstantiateSingletons()`단계 동안에 발생됩니다.

#### Example: The `PropertyOverrideConfigurer`

`PropertyOverrideConfigurer`(또 다른 Bean factory post-processor)는 `PropertySourcesPlaceholderConfigurer` 과 비슷합니다. 하지만 후자와는 다르게, 본래의 정의는 모든 Bean의 Properties에 대하여 기본적인 값 또는 값을 가지지 않습니다. 만약, 오버라이딩 한 `Properties` 파일이 특정 Bean property에 대한 항목이 없는 경우, 기본적인 context 정의가 사용됩니다.

Bean의 정의는 override 되어지는 것을 인식하지 못하는 것을 주목해라! 그래서 오버라이드 설정이 사용되어지는 XML 기반의 정의는 명백하지 않습니다. 같은 Bean property에 대한 다른 값을 정의하는 다양한 PropertyOverrideConfigurer 인스턴스의 경우에, 오버라이드 메커니즘 때문에 마지막 값이 승리한다.

Properties 파일 구성 행은 다음 형식을 따릅니다.

```
beanName.property=value
```

다음 리스트는 형식의 예를 보여줍니다.

```
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```

위의 예제 파일은 `driver`과 `url` properties를 가진 `dataSource`를 호출하는 Bean을 포함하는 Container 정의에 사용됩니다.

복합적(Compound) property names 또한 지원되어진다. 오버라이드 되어지는 최종 property를 제외한 경로의 모든 구성요소가 null 이 아닌 한( 아마(presumably) 생성자들로부터 초기화 되어진다. ) 다음 예제에서, tom Bean의 fred property의 bob property의 bob propertydml sammy property는 스칼라 값 123으로 세트 되어진다.

```
tom.fred.bob.sammy=123
```

> 명시된 오버라이드 값은 항상 문자 그대로의 값이다. 그것들은 Bean 참조에 의해서 번역 되어지지 않는다. 이러한 규칙은 XML Bean 정의에서 본래의 값이 Bean reference를 명시할 때 적용된다.

spring 2.5에서 소개된 `context` namespace를 통해, property 오버라이드를 별도의 configuration element로 설정할 수 있게 되었다. 다음 예제가 보여준다.

```xml
<context:property-override location="classpath:override.properties"/>
```

### 1.8.3 Customizing Instantiation Logic with a `FactoryBean`

개발자는 팩토리인 객체를 위해서 `org.springframework.beans.factory.FactoryBean` 인터페이스를 구현할 수 있다.

**`FactoryBean` 인터페이스는 Spring IoC Container 인스턴스화 로직에 연결하는 연결지점 입니다. 만약 장황한(verbose) XML과는 반대로 자바에서 잘 표현되는 복잡한 초기화 코드를 가지고 있다면, 개발자는 자신의 `FactoryBean`을 생성할 수 있고 클래스 내에서 복잡한 초기화 코드를 작성하고 그리고 Container에 사용자 `FactoryBean` 연결할 수 있다.**

`FactoryBean` 인터페이스는 3가지의 메소드를 지원한다.

- `Object getObject()` : 이 팩토리가 생성하는 객체의 인스턴스를 리턴해준다. 이 인스턴스는 공유되어질 수 있고, 이 팩토리가 Singleton을 리턴하는지 또는 Prototypes를 리턴하는지에 의존한다.
- `boolean isSingleton()` :  `FactoryBean`이 Singleton을 리턴하면 `true`를 리턴하고 그렇지 않으면 `false`를 리턴한다.
- `Class getObjectType()` : `getObject()` 메소드에 의해 리턴되어지는 객체 타입을 리턴하거나, 미리 타입이 알려지지 않앗다면 `null`을 리턴한다.

`FactoryBean` 개념과 인터페이스는 Spring Framework의 많은 곳에서 사용되어지고 있다. `FactoryBean` 인터페이스의 50개 이상의 구현은 Spring 자체와 함께 제공(ship with)됩니다.

**개발자가 실제 FactoryBean 인스턴스를 Bean 대신에 Container에게 요청할 때, &기호를 `ApplicationContext`의 `getBean()` 메소드가 호출될 때 머리맡에 붙여야(preface)한다.** 따라서 `myBean` 이 `id`인 주어진 `FactoryBean`에 대해, 컨테이너에서 `getBean("myBean")`을 호출하는 것은 `FactoryBean`에 의해 생산된 Bean이 리턴되고, 반면에 `getBean("&myBean")`을 호출하는 것은 `FactoryBean` 인스턴스 자체를 리턴한다.



























