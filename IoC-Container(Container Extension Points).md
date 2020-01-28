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

> `BeanPostProcessor`와 마찬가지로(As with),  개발자는 `BeanFactoryPostProcessor` 늦게 초기화 하도록 설정하는 것을 원하지 않을 것이다.













































