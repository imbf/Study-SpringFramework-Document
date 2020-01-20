# IoC(Inversion of Control) Container

## 1.4 Dependencies

기업형 어플리케이션은 한개의 객체로써 구성되어 있지 않다. 심지어는 가장 간단한 어플리케이션 조차도 일관된(coherent) 응용 프로그램으로써 간주할 수 있게끔 보여주기 위해 적은 수의 객체들도 서로 관련되어 일을한다. 다음 섹션에서는 
독립적인(stand-alone) 여러 Bean의 정의에서 목표를 달성하기 위해 여러 객체가 협업하는 완전히 완성된 어플리케이션까지 가는 방법에 대해서 설명하겠다.

### 1.4.1 Dependency Injection

**의존성 주입은(DI) 객체가 생성자 인자, 팩토리 매소드의 인자, 팩토리 메소드로부터 객체가 리턴되고 생성된 후의 객체 인스턴스의 properties를 통하여 의존성(즉, 객체가 함께 일하는 또 다른 객체)을 정의하는 프로세스를 가리킨다.** Container는 빈을 생성할 때 의존성을 주입한다. 이러한 프로세스는 클래스의 생성자를 직접 사용 또는 Service Locator Pattern을 사용함으로써 자신의 종속성 인스턴스화 또는 위치를 제어하는 Bean 자체의 역을 의미합니다.

코드는 의존성 원칙과 함께 더 간결해지고, 분리도는 객체가 자신의 의존성을 제공받을 때 더 효과적이다. 객체는 해당 의존성을 찾지 않고, 의존성의 위치나 클래스를 알지 못한다. 결과적으로 클래스는 테스트하기에 더 쉬워지고 특히 의존성이 인터페이스나 추상 클래스에 있을 때 더 쉬워진다. 이렇게 DI를 사용하면 **stub**나 **mock**을 구현해 단위 테스트를 할 수 있다.

의존성 주입의 두가지 변형

- **Constructor-based dependency injection**
- **Setter-based dependency Injection**

### Constructor-based Dependency Injection

 **Constructor argument resolution matching** 인자의 타입을 사용함으로써 발생한다. Bean을 정의하는 생성자 인수에 잠재적인 모호성(ambiguity)이 존재하지 않는 경우, Bean definition에서 생성자 인수가 정의되는 순서는 Bean이 인스턴스화 될 때 해당 인수가 적절한 생성자에 제공되는 순서입니다. 다음의 클래스를 고려해 봅시다.

```java
package x.y;

public class ThingOne {
  public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
    //...
  }
}
```

**Thingtwo와 ThingTree 클래스가 상속에 관련이 없는 클래스라고 가정하면, 어떤 잠재적인 모호함(ambiguity)도 존재하지 않는다. 게다가 다음의 configuration이 잘 동작하므로, 생성자 인수의 인덱스 또는 타입을 \<constructor-arg/> 요소에 명시할 필요가 없습니다.**

```xml
<beans>
	<bean id="beanOne" class="x.y.ThingOne">
  	<constructor-arg ref="beanTwo"/>
    <constructor-arg ref="beanThree"/>
  </bean>
  
  <bean id="beanTwo" class="x.y.ThingTwo"/>
  
  <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

또 다른 Bean이 참조될 때, 타입을 알고 있다면, matching은 발생할 수 있다. (앞의 예제 에서와 같이) **\<value>true\</value>**와 같이 간단한 타입이 사용된다면, Spring은 도움없이 타입을 match할 수 없을 것이다.
다음의 클래스를 고려해 보자.

```java
package examples;

public class ExampleBean{
  
  // Number of years to calculate the Ultimate Answer
  private int years;
  
  // The Answer to Life, the Universe, and Everything
  private String ultimateAnswer;
  
  public ExampleBean(int years, String ultimateAnswer){
    this.year = years;
    this.ultimateAnswer = ultimateAnswer;
  }
}
```

**Constructor argument type matching**

이전 시나리오에서, Configuration Metadata에 **type 속성**을 사용함으로써 생성자 인자의 타입을 명시한다면  Container는 간단한 타입의 타입 매칭을 할 수 있습니다. 다음 예에서 알아봅시다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
  <constructor-arg type="int" value="7500000"/>
  <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

**Constructor argument index**

개발자는 **index 속성**을 사용해서 생성자 인수의 index를 명시할 수 있습니다. 다음 예를 참고해 봅시다.

```java
<bean id="exampleBean" class="examples.ExampleBean">
  <constructor-arg index="0" value="7500000"/>
  <constructor-arg index="1" value="42"/>
</bean>
```

**constructor argument name**

개발자는 명확함(disambiguation)을 위해서 생성자 인자의 name을 사용한다, 다음 예를 참고 해 보자.

```xml
<bean id="exampleBean" calss="examples.ExampleBan">
	<constructor-arg name="years" value="7500000"/>
  <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

명심해라, 이 작업을 즉시 수행하려면(to make this work out of the box), Spring이 생성자로부터 인자의 이름을 찾을 수 있게끔 하기위해서 **디버그 플래그를 활성화**하여 코드를 컴파일해야 합니다. 만약 디버그 플래그를 활성화하여 코드를 컴파일 하기 원치 않거나 할 수 없는 경우에 생성자의 인자의 name을 **@ConstructorProperties** JDK annotation을 이용해서 작성하여라. 

샘플 클래스는 다음과 같다.

```java
package examples;

public class ExampleBean{
  
  // Fields omitted
  
  @ConstructorProperties({"years", "ultimateAnswer"})
  public ExampleBean(int years, String ultimateAnswer){
    this.year = years;
    this.ultimateAnswer = ultimateAnswer;
  }
}
```

### Setter-based Dependency Injection

**Setter 기반의 Dependency Injection 이란 Bean을 인스턴스화 하기위해 인자가 없는 생성자 또는 정적 팩토리 메소드를 호출한 후 생성된 Bean의 setter 메소드를 호출하는 Container에 의해 이루어진다.** 

다음의 예제는 setter injection을 사용함으로써 의존성이 주입된 클래스를 보여준다. 이 클래스는 자바의 문법을 따르고, 컨테이너에서 특정한 인터페이스, 클래스, annotation과 의존이 없는 POJO(Plain Old Java Object) 이다.

```java
public class SimpleMovieLister{
  
  // SimpleMovieLister 클래스는 MovieFinder의 의존성을 가지고 있다.
  private MovieFinder movieFinder;
  
  // Spring Container가 Movie Finder를 주입하기위한 setter method이다.
  public void setMovieFinder(MovieFinder movieFinder){
    this.movieFinder = movieFInder;
  }
  
  // 실제로 의존성이 주입된 MovieFinder 객체를 사용하는 비즈니스 로직은 생략되었다.
}
```

**ApplicationContext는 자신이 관리하는 빈을 위해 생성자 기반의 의존성 주입과 생성자 기반의 의존성 주입을 지원한다.** 
생성자를 통해서 의존성들이 이미 주입된 후에도, ApplicationContexts는 생성자 기반의 의존성 주입을 지원한다.  PropertyEditor 인스턴승와 함께 사용해서 한 형식에서 다른 형식으로 properties를 변환하는 BeanDefinition 형식으로 종속성을 구성하면 됩니다. **대부분의 Spring 사용자는 이러한 클래스를 직접(프로그래밍 방식으로) 사용하지 않고 XML Bean 정의, Annotated 컴포넌트(@Component, @Controller 등과 같은 어노테이션이 붙여진 클래스) 또는 @Configuration 클래스의 @Bean 메소드를 사용한다.** 이러한 source들은 내부적으로 BeanDefinition 인스턴스로 변환되어 전체 Spring IoC Container의 인스턴스를 로드하는 데 사용됩니다.

> #### **Constructor-based or setter-based DI?**
>
> 생성자 기반 또는 setter 기반의 의존성을 혼합할 수 있으므로, 선택적인 의존성들을 위해서 configuration 메소드 또는 setter 메소드를 사용하고, 의무적인 의존성들을 위해서 생성자를 사용하는 것은 좋은 규칙이다. setter 메소에서 @Required 어노테이션을 사용해서 property를 필수 의존성으로 만들 수 있습니다. 그러나 프로그래밍 방식으로 프로그래밍 방식으로 인수를 검증하는 생성자 삽입이 더 바람직합니다.
>
> Spring 팀은 일반적으로 생성자 삽입을 지지합니다(advocates). 생성자 삽입을 사용하면 어플리케이션 구성요소를 불변의(immutable) 객체로서, 필요한 의존성이 null이 아니라는 것을 확실히 해줍니다. 게다가, 생성자로 삽입한 의존성은 완전히 초기화된 상태로 항상 리턴해줍니다. 부수적으로(As a side note), 생성자 인수가 많으면 효율적이지 못한 코드로 진행될 가능성이 높음으로 클래스가 너무 많은 책임을 가질 가능성이 있으며, 적절한 분리 문제를 해결하기 위해 리팩토링 해야한다는 것을 암시합니다.
>
> Setter를 사용하는 의존성 주입은 클래스내에서 합당한 기본값이 할당된 선택적 의존성을 위해서 주로 사용된다. 그렇지 않으면 의존성을 사용하는 모든 곳에서 not-null check를 수행 해야만 한다. setter를 사용한 injection의 한가지 장점은 Setter 메소드가 해당 클래스의 오브젝트를 나중에 재구성 하거나 다시 주입할 수 있다는 것 입니다.
> **JMX MBeans**를 통한 관리는 setter injection을 위한 강력한(Compelling) 사용사례이다.
>
> **특정한 클래스에 가장 적합한 의존성 주입 방법을 사용하십시요** 때떄로 코드를 가지고 있지 않는 외부의 클래스를 사용할 때 의존성 주입 방법을 선택하십시요. 예로들어 만약 어느 setter method도 외부 클래스에 존재하지 않는다면 생성자를 활용한 의존성 주입이 DI를 형성하는데 가능한 방법일 것입니다.

### Dependency Resolution Process

**Container는 다음과 같이 Bean의 의존성 확인(Resolution)한다.**

- 모든 Bean들을 설정하는 configuration metadata와 함께 ApplicationContext는 생성되어지고 초기화 되어진다.
   Configuration metadata는 XML, Java Code, Annotation에 의해서 명시되어진다.
- 각 Bean을 위해서 의존성들은 properties, 생성자 인자, 정적 팩토리 메소드의 인자의 형태로 표현이 되어질 것이다. Bean이 실제로 생성되어질 때 이러한 의존성들이 Bean에게 제공되어질 것이다.
- 각각의 property 또는 생성자의 인자는 설정할 값의 실제 정의 또는 컨테이너의 다른 Bean에 대한 참조입니다.
- 각각의 property 또는 생성자의 인수의 값은 특정한 형태에서 해당 속성 또는 생성자 인수의 실제 유형으로 변환됩니다.
   기본적으로 Spring은 String 형태로 제공된 값을 내장 타입(int, long, String, boolean, ..) 으로 변환할 수 있습니다.

Spring Container는 컨테이너가 생성될 때 각 Bean의 구성을 확인합니다. 그러나 **Bean의 속성이 생성될 때까지 설정되지 않는다면, singleton-scoped 이고 사전에 인스턴스화 되도록(Default) Container가 생성될 때 Bean이 생성될 것이다.** 
**그렇지 않으면 Bean은 요청받을 때 생성이 될것이다.** Bean의 생성은 잠재적으로 Bean의 의존성 그리고 Bean의 의존성의 의존성이 생성되고 할당되어질 때 생성된 Bean의 그래프를 생성합니다. 이러한 종속성간의 불일치 확인이 늦게 나타날 수 있습니다.

> #### **Circular Dependencies**
>
> 만약 의존성 주입으로 생성자 주입을 대부분 사용한다면, 해결할 수 없는 순환 의존성 시나리오를 발생시킬 수도 있습니다.
>
> 예로들어 : 만약 클래스 A가 생성자 주입을 통해 클래스 B의 인스턴스를 요구하고, 클래스 B가 클래스 A의 인스턴스를 생성자 주입을 통해 요구한다. 만약 개발자가 클래스 A와 B가 서로 의존성이 주입되어지게 Bean을 설정한다면 Spring IoC Container는 JVM 실행시간동안 순환참조를 발견하고 **BeanCurrentlyInCreationException** 예외를 던질것이다.
>
> 이러한 문제를 해결하는 한가지 방법으로는 생성자 대신에 setter 메소드가 설정할 일부 클래스 코드를 수정하는 것이다. 또는 생성자 주입을 피하고 Setter 주입만을 사용하는 것이다. 이러한 방법들이 추천되지는 않지만, 개발자들은 순환 의존성들을 setter를 이용한 의존성 주입으로써 설정할 수 있다.
>
> 전형적인 케이스(순환 의존성이 없는 경우)와는 다르게, Bean A와 B 사이의 순환 의존성으로 인해 Bean 중 하나가 완전히 초기화 되기전에 하나의 Bean이 다른 Bean으로 주입됩니다. (전형적인 chickent-and-egg 시나리오 이다.) 

일반적으로 개발자들은 스프링이 올바른 일은 한다고 믿는다. Spring은 Container가 load되는 시간에 순환 의존성 그리고 존재하지 않는 Bean을 참조하는 것과 같은 설정 문제를 찾을 수 있다. Spring은 Bean이 실제로 생성될 때 속성을 설정하고 가능한 늦게 의존성을 해결합니다. **이러한 것들은 올바르게 로딩되어진 Spring Container가 객체나 의존성을 생성하는데 문제가 있다면 객체를 요청할때서야 예외를 발생한다는 것을 의미합니다.** 일부 설정 이슈에 대한 잠재적으로 지연된 가시성은 기본적으로 미리 인스턴스화된 singleton Bean들에 의해서 ApplcationContext가 구현되는 이유이다. 이러한 Bean이 실제로 필요하기 전에 생성하는데 약간의 선행(upfront)시간과 메모리가 필요하므로 나중에 ApplicationContext가 생성될 때 설정 문제를 발견할 수 있다. 개발자들은 singleton Bean들을 미리 인스턴스화 시키는 것 보다도 늦게 초기화 하기 위해서 이러한 default method를 오버라이드 할 수 있다.

**만약 순환 의존성이 존재하지 않는다면,  하나 이상의 collaborating Bean이 dependent Bean에 주입이 되어질 때 각각의 collaborating Bean은 dependent Bean에 주입 되어지기 전에 완전히 구성 되어집니다.** 이러한 사실은, 만약 Bean A가 Bean B의 의존성을 가지고 있다면, Spring IoC Container는 Bean A 에서 setter method가 호출되기 이전에 Bean B 를 구성해 놓습니다. 다른말로 Bean이 인스턴화되고(미리 인스턴화 된 singleton이 아닌 경우), 종속성이 설정되고, 관련된 라이프사이클 메소드 ( 구성된 init 또는 InitializingBean 콜백 메소드) 가 호출된다.

### Examples of Dependency Injection

setter 기반의 의존성 주입을 위한 XML 기반의 configuration metadata를 사용한 예제이다.

```xml
<beans>
	<bean id="exampleBean" class="examples.ExampleBean">

    <!-- 중첩된 ref 요소를 사용한 setter 주입 -->
    <property name="beanOne">
    	<ref bean="anotherExampleBean"/>
    </property>
    
    <!-- 적절한 ref 속성을 사용한 setter 주입 -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
  </bean>
  
  <bean id="anotherExampleBean" class="examples.AnotherBean"/>
  <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
  
</beans>
```

위의 예제와 일치하는 ExampleBean 클래스

```java
public class ExampleBean{
  
  private AnotherBean beanOne;
  
  private YetAnotherBean beanTwo;
  
  private int i;
  
  public void setBeanOne(AnotherBean beanOne){
    this.beanOne = beanOne;
  }
  
  public void setBeanTwo(YetAnotherBean beanTwo){
    this.beanTwo = beanTwo;
  }
  
  public void setIntegerProperty(int i){
    this.i = i;
  }
}
```

위의 예제에서 setter은 XML 파일에서 설정된 properties와 대응하여 match하기 위해 선언되어진다.

다음 예제는 생성자 기반의 의존성 주입을 위한 XML configuration Metadata 예제이다.

```xml
<beans>
  <bean id="exampleBean" class="examples.ExampleBean">
    <!--중첩된 ref 요소를 사용하여 생성자 주입-->
		<constructor-arg>
			<ref bean="anotherExampleBean"/>
    </constructor-arg>
    
    <!-- 적절한 ref 속성을 사용하여 생성자 주입 -->
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg type="int" value="1"/>
	</bean>
  
  <bean id="anotherExampleBean" class="examples.AnotherBean"/>
  <bean id="yetAnotherBean" class="examples.YetAnotherBean"
</beans>

```

다음 예는 XML-Based Configuration Metadata와 일치하는 ExampleBean 클래스를 보여준다.

```java
public class ExampleBean{
  
  private Another beanOne;
  
  private YetAnotherBean beantwo;
  
  private int i;
  
  public ExampleBean(AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i){
    this.beanOne = anotherBean;
    this.beanTwo = yetAnotherBean;
    this.i = i;
  }
}
```

Bean definition에서 명시된 생성자 인자는 ExampleBean 클래스 생성자의 인자로써 사용된다.

지금부터는 이러한 예제의 변형에 대해서 참고해보자. 생성자를 사용하는 대신, Spring은 객체의 인스턴스를 리턴하기 위해서 정적 팩토리 메소드를 리턴한다.

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
	<constructor-arg ref="anotherExampleBean"/>
  <constructor-arg ref="yetAnotherBean"/>
  <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

 다음의 예제는 ExampleBean 클래스와 일치하는 예제이다.

```java
public class ExampleBean{
  
  // private 생성자
  private ExampleBean(...){
    //...
  }
  
  // 정적인 팩토리 메소드
  // 이 메소드의 인자들은 어떻게 그들이 실제로 사용되어지는 것과 상관없이 리턴되는 Bean의 의존성으로 여겨진다.
  public static ExampleBean createInstance(AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
    ExampleBean eb = new ExampleBean(...);
    // some other operations...
    return eb;
  }
}
```

정적인 메소드의 인자는 **\<constructor-args/>** 요소에 의해서 제공되어진다. 생성자로써 사용하는 것처럼 같다. 팩토리 메소드에 의해서 리턴되는 클래스의 타입은 정적 팩토리 메소드를 포함하는 클래스와 동일한 유형일 필요는 없습니다. (위의 예제와는 달리). 정적이지 않는 팩토리 메소드의 인스턴스는 본질적으로 동일한 방법으로 사용되어진다. (하지만 class 속성 대신에 **factory-bean 속성을 사용한다.**)

### 1.4.2 Dependencies and Configuration in Detail

이전 섹션에서 언급 한것처럼. 개발자들은 bean의 특성 및 생성자 인자를 관리되는 Bean에 대한 참조 또는 인라인으로 정의 된 값으로 정의할 수 있습니다. Spring의 XML 기반의 configuration metadata는 이를 위해 \<property/> 와 \<constructor-arg/> 요소에서 하위 요소 유형을 지원합니다.

### Straight Values (Primitives, String, and so on)

**\<property/>**요소의 **value 속성**은 사람이 읽을 수 있는 문자열 표현으로 생성자 인수나, 속성을 정의합니다. Spring의 변환(conversion) 서비스는 이러한 String의 값들을 실제 property나 인자의 값으로 바꾸어준다.
다음 예제는 다양한 value의 값이 설정되어지는 것을 보여준다.

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destory-method="close">
  
  <!-- setDriverClassName 호출의 결과 -->
  <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
  <property name="username" value="root"/>
  <property name="password" value="masterkaoli"/>
  
</bean>
```

다음은 더 간결한 XML Configuration을 위한 p-namespace를 사용한 예제이다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframewokr.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/chema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>
  
</beans>
```

위의 XML파일이 보다 더 간단하다. 하지만, Bean을 정의할 때 자동 property를 완성해 주는 기능을 지원해주는 IDE(intellij와 spring Tool Suite)를 사용하지 않으면 런타임시에 오타(typo)가 발견될 수도 있습니다. 이러한 IDE의 도움을 매우 추천합니다.

개발자들은 **java.util.Properties** 인스턴스를 다음과 같이 설정할 수 있습니다.

```xml
<bean id="mappings" class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
	
  <!-- typed as a java.util.Properties -->
  <property name="properties">
  	<value>
    	jdbc.driver.classname=com.mysql.jdbc.Driver
      jdbc.url=jdbc:mysql://localhost:3306/mydb
    </value>
  </property>
</bean>
```

Spring Container는 **\<value/> 요소**내의 텍스트를 **java.util.Properties**의 인스턴스로 JavaBeans **PropertyEdiotr** 방법을 사용해서 변환합니다. 이러한 방법은 매우 간결하고 스프링팀이 value 속성 스타일에 중첩된 **\<value/>**요소를 사용하는 것을 선호하는 부분 중 하나입니다.

### The idref element

**idref 요소는 Container에 있는 다른 Bean의 Id(문자열 값-참조가 아님)를 \<constructor-arg/> 또는 \<property/> 요소에전달하는 오류 방지 방법입니다.**

**다음의 예는 어떻게 idref 요소를 사용하는지에 대해서 알려줍니다. (추천 하는 방법)**

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
	<property name="targetName">
  	<idref bean="theTargetBean"/>
  </property>
</bean>
```

이전 Bean의 정의는 런타임시 다음의 Bean 정의 예제와 정확히 동일하다.

```xml
<bean id="theTargetBean" class="..."/>

<bean id="client" class="...">
	<property name="targetName" value="theTargetBean"/>
</bean>
```

idea 태그를 사용하면 배포시 컨테이너가 참조된 bean이 실제로 존재하는지를 검증(validate) 할 수 있음으로 첫 번째 예제가 두번째 예제보다 선호됩니다. 두 번째 변형된 예제에서, Client Bean의 targetName 특성에 전달 된 값에 대해 검증을 수행하지 않기 때문에 1번 예제의 idref 태그르 사용할 것을 추천합니다. 오류는 Client Bean이 실제로 인스턴스화 될때 이상한 결과와 함께 오타가 발견됩니다. 만약 Client Bean이 prototype bean이라면, 이러한 오타와 예외는 container가 배포된 후에 발견이 되어질 것입니다.

> **idref 요소의 local 속성** 은 4.0 Beans XSD 에서 더이상 지원이 되지 않습니다. 더이상 일반적인 Bean의 참조보다 Value를 제공하지 않기 때문입니다. 4.0 버전을 업그레이드 할 때 존재하는 idef local 참조를 **idref bean**으로 바꾸십시요.

Value를 가지고 오는 \<idref/> 요소의 일반적인 위치는 ProxyFactoryBean Bean definition에서 AOP interceptors의 configuration에 위치한다. 개발자가 interceptor name을 \<idef/> 요소를 사용해서 명시한다면 interceptor ID를 틀리는 것을 방지해 줍니다.

### References to other Beans (Collaborators)

**ref 요소**는 \<constructor-arg/> 또는 \<property/> 요소 안에 들어 있는 마지막 요소이다. 이러한 요소에서 Bean에 지정된 특정한 property의 값을 Cotainer가 관리하는 Bean(Collaborator)에 대한 reference가 되도록 설정한다. 참조가 되어지는 Bean은 property가 설정되어 있고, property가 설정되기 전에 요구에 의해 초기화 되어져 있는 Bean의 의존성을 가진다.
(만약 collaborator가 singleton bean이라면, 이미 Container에 의해 초기화 되어져 있을 것이다. ) **모든 참조는 궁극적으로 다른 객체에 대한 참조 입니다. Bean의 Scoping 과 Validation은 bean, local, parent 속성을 통해 객체의 ID 또는 name을 명시 했는지 안했는지에 따라서 다릅니다.**

**\<ref/> 태그의 bean 속성**을 통해서 target Bean을 명시하는 것은 가장 일반적인 형태이고, 같은 XML 파일에 존재 하는 것과는 관계없이 부모 Container 또는 같은 Container에서 모든 Bean의 참조를 허용합니다. **bean속성의 값은** target Bean의 id 속성과 또는 target Bean의 name 속성과 같습니다. 

다음 예를 통해서 어떻게 ref 요소를 사용하는지 알아봅시다.

```xml
<ref bean="someBean"/>
```

**parent 속성**을 통해 target Bean을 명시하는 것은 현재  Container의 부모 Container 안에서 reference를 생성하는 것이다. **parent 속성**의 값은 아마 target Bean의 id 속성 또는 target Bean의 name 속성에서의 값중의 하나와 같을 것입니다. target Bean은 현재 Container의 부모 Container에 있어야 합니다. 컨테이너 계층구조가 있고, Parent Bean과 이름이 동일한 프록시를 사용하여 존재하는 Bean을 부모 Container에 감쌀려는 경우에는 개발자는 Bean reference 변형을 사용해야 합니다.

다음의 2가지 예는 어떻게 parent 속성을 사용하는가에 대해서 알려줍니다.

```xml
<!-- parent context-->
<bean id="accountService" class="com.something.SimpleAccountService">
	<!-- 필요에 따라 의존성 삽입 -->
</bean>
```

```xml
<!-- child context -->
<!-- Bean 이름이 parent Bean과 동일하다. -->
<bean id="accountService" class="org.springframework.app.framework.ProxyFactoryBean">
  <property name="target">
    <ref parent="accountservice"/> <!-- 부모 Bean을 어떻게 참조하는지에 주목하십시요. -->
  </property>
  <!-- 필요한 설정 및 의존성을 삽입하세요. -->
</bean>
```

> **ref 요소의 local 속성**은 더이상 4.0 Beans XSD 에서 지원하지 않습니다. 일반적인 bean의 참조보다 값을 제공하지 않기 때문입니다. 4.0 스키마를 업그레이드할 때 이미 존재하는 ref local 참조를 ref bean으로 바꾸세요

### Inner Beans

**\<property/> 또는 \<constructor-arg/> 요소 내부의 \<bean/> 요소**는 inner Bean을 정의한다. 다음 예제를 참고해 보자.

```xml
<bean id="outer" class="...">
	<!-- target Bean의 참조를 사용하는 것 대신에, 간단히 target Bean inline을 정의하였다. -->
  <property name="target">
  	<bean class="com.example.Person">
    	<property name="name" value="Fiona Apple"/>
      <property name="age" value="25"/>
    </bean>
  </property>
</bean>
```

내부 Bean의 정의는 ID 또는 name을 필요로 하지 않습니다. 만약 명시되어 있다해도, Container는 이러한 값들을 식별자로써 사용하지 않을 것입니다. **Container는 Inner Bean 생성시 Scope flag도 무시합니다.** inner Bean은 항상 익명적이고, outer Bean과 함께 항상 생성되어 있어져야하기 때문입니다. Inner Bean에 독립적으로 접근하는 것과 Inner Bean을 가지고 있는 Bean 이외에 다른 Bean에 주입하는 것은 불가능합니다.

특별한 상황에서는, custom scope 로부터 destruction callbacks를 받을 수 있다. - 예로들어, singleton Bean에 포함된 request-scoped inner Bean의 경우. Inner Bean 인스턴스의 생성은 Inner Bean을 포함하는 Bean에 연결되어 있지만
<u>**but destruction callbacks let it participate in the request scope's lifecycle**</u> 

이러한 것들은 일반적인 시나리오가 아니며, Inner Bean들은 전형적으로 자신을 포함하는 Bean의 범위를 공유한다.

### Collections

**\<list/>, \<set/>, \<map/>, \<props/> 요소는** Java Collection type의 List, Set, Map, Properties의 인자와 properties를 각각 설정해준다.

다음 예는 이러한 요소들을 어떻게 사용하는지에 대해서 보여준다.

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
  
  <!-- setAdminEmails(java.util.Properties) 호출 결과 -->
  <property name="adminEamils">
  	<props>
    	<prop key="adminstrator">adminstrator@example.org</prop>
      <prop key="support">support@example.org</prop>
      <prop key="development">development@example.org</prop>
    </props>
  </property>
  
  <!-- setSomeList(java.util.List) 호출 결과 -->
  <property name="someList">
  	<list>
    	<value>a list element followed by a reference</value>
      <ref bean="myDataSource"/>
    </list>
  </property>
  
  <!-- setSomeMap(java.util.Map) 호출 결과 -->
  <property name="someMap">
  	<map>
    	<entry key="an entry" value="just some string"/>
    	<entry key="a ref" value="myDataSource"/>
    </map>
  </property>
  
  <!-- setSomeSet(java.util.Set) 호출 결과 -->
  <property name="someSet">
  	<set>
    	<value>just some string</value>
      <ref bean="myDataSource"/>
    </set>
  </property>
</bean>
```

key 또는 value 또는 set value의 값은 다음 요소중 어떤것이라도 될 수 있다.

```XML
bean | ref | idrf | list | set | map | props | value | null
```

### Collection Merging

Spring Container 또한 merging collection을 지원한다. 어플리케이션 개발자는 부모의 \<list/>, \<map/>, \<set/>, \<props/> 요소를 정의할 수 있도록 그리고 자손의 \<list/>, \<map/>, \<set/>, \<props/> 요소들이 부모 collection으로부터의 값을 상속하고 오버라이드 할 수 있도록 할 수 있다. 즉, 자손 collection의 값은 부모와 자손 collection의 요소를 병합한 값과 같다. 부모 collection에서 정의된 값을 오버라이딩 할 수 있습니다.

이번 섹션인 marging 섹션은 부모와 자식의 bean 메커니즘에 대해서 논의하고, 부모와 자식 Bean 설정에 익숙하지 않는 분은 relevant section을 시작하기전에 읽으실 것을 권유합니다.

다음은 collection merging을 입증하는 예제이다.

```xml
<beans>
	<bean id="parent" abstract="true" class="example.ComplexObject">
  	<property name="adminEmails">
    	<props>
      	<prop key="adminstrator">adminstrator@example.com</prop>
      	<prop key="support">support@example.com</prop>
      </props>
    </property>
  </bean>
  <bean id="child" parent="parent">
  	<property name="adminEmails">

    </property>
  </bean>
</beans>
```































