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
			<!-- 자식 Collection Definitiond에서 merge가 명시되어 있다. -->
      <props name="adminEmails">
      	<prop key="sales">sales@example.com</prop>
        <prop key="support">support@example.co.uk</prop>
      </props>
    </property>
  </bean>
</beans>
```

위 예제는 child Bean Definition의 adminEmails property의 porps 요소의 merge=true 속성을 사용하세요. Child **Bean이 Container에 의해서 분석되고 인스턴스화 될 때 인스턴스 결과는 자손의 adminEmails collection과 부모의 adminEmials collection을 병합한 결과를 포함한 adminEmails Properties collection을 가집니다.**

결과는 다음과 같다.

```
adminstrator=adminstrator@example.com
sales=sales@example.com
support=supprot@example.co.uk
```

**자식 Properties 컬렉션의 값 설정은 부모의 \<props/>요소에 있는 property의 모든 요소를 상속하고 support 값을 위해서 자식의 값이 부모 컬렉션에서의 값을 오버라이드 합니다.**

이러한 병합 과정은 \<list/>, \<map/>, \<set/> 컬렉션 타입에도 비슷하게 적용된다. \<list/> 요소의 특별한 경우에, List 컬렉션과 관련있는 의미(즉, value의 순서있는 컬렉션의 개념) 은 유지되어야 한다. 부모의 값들은 모든 자식의 list의 값보다 우선시 된다.
Map, Set, Properties 컬렉션 타입의 경우에는 어떠한 순서도 존재하지 않는다. 이러한 이유로(hence), 순서가 있지 않는 시멘틱들도 Container가 내부적으로 사용하는 관련된 Map, Set, Properties 구현 타입에 기초가되는 컬렉션 타입에 영향을 미친다.

### Limitations of Collection Merging

개발자는 다른 컬렉션 타입을 병합할 수 없다. 만약 이러한 시도를 한다면 적절한 예외가 던져질 것이다. **merge 속성은 하위의, 상속받는 자손 definition에 명시되어야 한다.** 부모 컬렉션 정의에 merge 속성을 명시하는 것은 쓸모없고, 바라는 병합 결과를 발생하지 않을것이다.

### Strongly-typed collection

Java 5에서 제네릭 타입이 소개되면서(with), 개발자는 강력한 타입의 collection을 사용할 수 있게 됬다. 즉, String 요소만 포함할 수 있도록 컬렉션 유형을 선언할 수 있다는 말이다. Spring을 사용하여 강력한 형식의 Collection을 Bean에 의존성 주입 하려는 경우, 개발자는 Spring의 타입변환 지원을 사용할 수 있다. 예로들어 강한타입의 Collection 인스턴스의 요소가 Collection 타입에 추가되기전에 적절한 타입으로 변환 되어질 것이다.

다음의 예제는 어떻게 이러한 것들을 정의하는지에 대해서 알려준다.

```java
public class SomeClass{
  private Map<String, Float> accounts;
  
  // setter를 통한 의존성 주입 방법
  public void setAccounts(Map<String, Float> accounts){
    this.account = accounts;
  }
}
```

```xml
<beans>
	<bean id="something" class="x.y.SomeClass">
  	<property name="accounts">
    	<map>
      	<entry key="one" value="9.99"/>
        <entry key="two" value="2.75"/>
        <entry key="six" value="3.99"/>
      </map>
    </property>
  </bean>
</beans>
```

 의존성 주입을 위한 something Bean의 accounts property가 준비가 되었을 때 강력한 형식의 Map\<String, Float> 의  요소 타입에 대한 제네릭 정보는 reflection에 의해서 사용할 수 있습니다. 게다가, Spring의 타입 변환 인프라는 다양한 요소의 값을 Float으로 인식한다. String 값(9.99, 2.75, 3.99)은 실제 Float 타입으로 변환이 되어집니다.

### Null and Empty String Values

Spring은 빈 문자열과 properties를 위해서 빈 인자를 처리합니다.

다음의 XML 기반의 configuration metadata는 email property에 빈 String 값을 설정하는 작은 정보를 포함한다.

```xml
<bean class="ExampleBean">
	<property name="email" value=""/>
</bean>
```

이전의 configuration metadata는 다음 Java Code와 대응한다.

```java
exampleBean.setEmail("");
```

\<null/>요소는 **null 값(null)**을 다룹니다.

다음은 \<null/>을 활용하는 방법에 대한 예제입니다.

```xml
<bean class="ExampleBean">
	<property name="email">
  	<null/>
  </property>
</bean>
```

이전의 XML 기반의 configuration Metadata는 다음의 Java Code와 대응된다.

```java
exampleBean.setEmail(null);
```

### XML Shortcut(간단한 방법) with the p-namesace

**p-namespace**는 개발자가 **bean 요소의 속성을** 사용하며 ( 중첩된 \<property/> 요소 대신 )  collaborating beans들의 값을 설정하는 방법이다.

Spring은 XML 스키마 정의에 기반한 namespace로 확장가능한 설정 형식을 제공해준다.이번 Chapter에서 논의될 **bean 설정 형식**은 XML 스키마 문서에 정의되어 있다. 그러나 p-namespace는 XSD 파일에 정의되어 있지 않고 오직 Spring 핵심 문서에만 저장되어 있다.

다음은 2가지의 같은 결과를 갖는 간단한 XML 형식을 보여준다. (한개는 표준 XML 형색 한개는 p-namespace를 사용한 XML 형식)

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean name="classic" class="com.example.ExampleBean">
  	<property name="eamil" value="someone@somewhere.com"/>
  </bean>
  
  <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
  
</beans>
```

위의 예제는 Bean 정의에서 email이라고 불리는 p-namespace의 속성에 대해서 보여주었다. 이러한 형식은 Spring에게 property 선언을 포함하도록 정의합니다.앞에서 언급했듯이 p-namespace에는 schema 정의가 없음으로 속성 이름을 property name으로 지정할 수 있습니다.

다음의 예는 다른 Bean을 참조하는 2가지  Bean의 정의를 포함하고 있습니다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean name="john-classic" class="com.example.Person">
    <property name="name" value="John Doe"/>
    <property name="spouse" ref="jane"/>
  </bean>
  
  <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>
  
  <bean name="jane" class="com.example.Person">
  	<property name="name" value="Jane Doe"/>
  </bean>

</beans>
```

위 예제는 p-namespace를 사용한 property value를 포함할 뿐"만 아니라, property references를 선언하기 위해 특별한 형식을 포함하고 있다. 첫 번째 Bean의 정의는 **\<property name="spouse" ref="jane"/>**를 사용해서 john으로부터 jane의 참조를 생성하지만, 두 번째 Bean 정의는 **p:spouse-ref="name"**을 사용함으로써 첫 번째 Bean의 정의와 똑같은 역할을 하는 Bean을 정의했다. 이러한 경우에 spouse는 property name이고, 반면에 -ref 부분은 단순한 값이 아니라 다른 Bean의 참조를 가리킨다.

> p-namespace는 표준 XML 형태처럼 유연하지 않습니다. 예로들어, property reference를 선언하는 형식은 ref로 끝나는 속성과 충돌하지만, 반면에(whereas) 표준 XML 형식은 그렇지 않습니다. 세 가지 접근 방식을 동시에 사용하는 XML 문서를 생성하지 않도록 접근 방식을 신중하게 선택하고 이를 팀 구성원에게 전달하는 것을 권장합니다.

### XML Shortcut with the c-namespace

p-namespace를 사용한 XML의 손쉬운 방법과 비슷하게, Spring 3.1에서 소개되었던 c-namespace는 중첩된 constructor-arg 요소 보다 생성자의 인자를 정의하기 위해서 inline 속성에 허용하는 방법이다.

다음의 예는 c-namespace를 사용해서 생성자 기반의 의존성 주입을 하는 예제를 보여준다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="beanTwo" class="x.y.ThingTwo"/>
  <bean id="beanThree" class="x.y.ThingThree"/>
  
  <!-- 선택적인 인자의 이름 전통적 선언 -->
  <bean id="beanOne" class="x.y.ThingOne">
  	<constructor-arg name="thingTwo" ref="beanTwo"/>
  	<constructor-arg name="thingThree" ref="beanThree"/>
  	<constructor-arg name="email" value="something@somewhere.com"/>
  </bean>
  
  <!-- c-namespace의 인자 이름 선언 -->
  <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>
</beans>
```

c-namespace는 이름으로 부터 생성자의 인자를 셋팅하기 위해 p-namespace와 같은 컨벤션(규칙)을 사용하고 있다.
비슷하게 c-namespace는 XML 파일로 선언되어야 한다. 비록 Spring core내에 존재하는 XSD schema에는 정의되어 있지 않지만.

생성자 인수의 이름을 사용할 수 없는 드문 경우(디버깅 정보 없이 bytecode가 컴파일 되었을 경우)에는, 다음과 같이 인수 인덱스에 fallback을 사용할 수 있습니다.

```xml
<!-- c-namespace 인덱스 선언 -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree" c:_2="something@somewhere.com"/>
```

> XML의 문법 때문에, index 정의에 선두 '_'가 필요하다. XML 속성 이름은 숫자로써 시작할 수 없다.( 허용하는 조금의 IDE가 있을 지라도 ) \<constructor-arg> 요소를 위한 대응하는 인덱스 표기법은 사용할 수 있지만, 기본의 선언 순서가 일반적으로 충분하기 때문에 사용하지 않는다.

실제로(In practice), 생성자 확인 메커니즘은 인수를 일치시키는데 매우 효율적이므로, 실제로 필요한 경우가 아니면(unless) configruation 전체에서 이름 표기법을 사용하는 것을 추천합니다.

### Compound(복합체) Property Names

개발자는 복합체 또는 중첩된 property 이름을 Bean properties를 설정할 때 사용합니다. 최종 property 이름을 제외한 경로의 모든 구성 요소가 null이 아닌 한.

다음의 Bean 정의에 관한 예제를 보아라

```xml
<bean id="something" class="things.ThingOne">
	<property name="fred.bob.sammy" value="123"/>
</bean>
```

 Something Bean은 sammy property를 가진 bob property를 가진 fred property를 가진다. 그리고 마지막 sammy property는 123이라는 값이 설정 되어진다. 이러한 property들이 작동하기 위해서는, Bean이 생성된 후 fred의 bob property와 something의 fred property가 무조건 null이 아니여야 한다. 그렇지 않으면 NullPointerException 예외를 발생할 것이다.

### 1.4.3 Using depends-on

만약 Bean이 또 다른 Bean의 의존성 이라면, 한 Bean은 다른 Bean의 property로써 설정되어진다고 의미할 수 있다. 전형적으로 XML 기반의 configuration metadata에서 \<ref/> 요소를 사용해서 의존성을 설정해야한다. 그러나 때때로 Bean들 간의 의존성은 직접적이지가 않다. 데이터베이스 드라이버 등록과 같이 클래스의 정적 초기화 프로그램을 트리거해야하는 경우를 예로들 수 있다. **depends-on 속성은 이러한 속성을 사용하는 Bean이 초기화 되기전에 하나 이상의 Bean을 초기화 시키는 속성이다.**

> 트리거(trigger)
>
> 특정 테이블에 INSERT, DELETE, UPDATE 같은 DML 문이 수행되었을 때 데이터베이스에서 자동으로 동작하도록 작성된 프로그램입니다.

다음 예는 dependes-on 속성을 사용해서 single Bean의 의존성을 표현하는 방법이다.

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean"/>
```

다수의 Bean에 의존성을 표현하기 위해, depends-on의 속성 값으로써 bean의 이름을 나열한다. (쉼표, 공백, 세미콜론은 유효한 구분기호(delimiters)이다.)

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager, accountDao">
	<property name="manager" ref="manager"/>
</bean>

<bean id="manager" class="ManagerBean"/>
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao"/>
```

depends-on 속성은 초기화 시간 의존성과 singleton Bean의 경우 소멸시간 의존성을 모두 지정할 수 있습니다. 지정된 Bean과의 의존 관계를 지정하는 의존 Bean은 지정된 Bean이 소멸되기전에  첫 번째로 소멸됩니다. 게다가 depends-on은 종료 순서를 제어할 수도 있습니다.

### 1.4.4 Lazy-initialized(게으르게 초기화되는) Beans

**기본적으로 ApplicationContext 구현은 초기화 프로세스의 일부로서(as) 모든 singleton Bean을 설정하고 생성합니다.** 일반적으로, 이러한 사전 인스턴스화는 바람직합니다. 환경설정 또는 주변의 환경 에러가 몇시간 또는 며칠 뒤가 아닌 즉시 발견되기 때문에 바람직합니다. 이러한것들이 바람직하게 동작하지 않을 때, Bean의 정의를 늦게 초기화하도록 함으로써 singleton의 사전 인스턴스화를 막을 수 있습니다. lazy-initialized bean은 IoC Container에게 Container가 초기화할 때 보다는 요청받을 때 bean의 인스턴스를 생성하도록 요청합니다.

XML에서 이러한 작업들은 \<bean/> 요소에 **lazy-init 속성**에 의해서 제어됩니다. 예제를 참고하세요.

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

이러한 configuration 들이 ApplicationContext에 의해서 처리될 때, lazy Bean은 ApplicationContext가 시작할 때 사전 인스턴스화가 되어지지 않고, 반면에 not.lazy Bean은 사전 인스턴스화가 된다.

lazy-initialized Bean이 lazy-initialized Bean이 아닌 singleton Bean에 의존성이라면, ApplicationContext는 lazy-initialized Bean을 시작시에 생성한다. singleton Bean의 의존성을 무조건 충족해야 하기 때문이다. lazy-initialized Bean은 not lazy-initialized Bean으로써 singleton Bean의 모든곳에 주입된다.

개발자는 **\<beans/> 요소의 default-lazy-init 속성**을 사용해서 Container 레벨에서 lazy-initialization을 제어할 수 있다.

다음의 예제 코드를 참고하자

```xml
<beans deafult-lazy-init="true">
	<!-- 어떠한 빈도 사전 인스턴스화 되지 않는다. -->
</beans>
```

### What is bean wiring?

- **Bean Wiring이란 Spring Container를 통해서 Bean들을 결합하는 과정이다.** 요구되어지는 Bean은 Container에게 알려져야하고, Bean을 wiring하는 시간에 어떻게 Container가 Bean을 연결짓기 위해 의존성을 삽입하는지 Container에게 알려져야한다.
- **Spring Container내의 Bean을 결합하는 것은 Bean wiring 또는 wiring으로 알려져 있다**. Spring이 Bean을 wiring할 때, 개발자는 Container에게 어떠한 Bean들이 필요하고,  Bean들을 묶기 위해 어떠한 의존성 주입이 필요한지에 대해서 알려줘야한다.

### Autowiring Collaborators

Spring Container는 수집한 Bean들간에 관계를 autowire할 수 있다. Spring은 ApplicationContext의 내용을 검증함으로써 자동적으로 Bean의 collaborators(other beans)를 확인할 수 있다.

**Autowiring이 가지는 장점**

- Autowiring은 생성자의 인자 또는 properties를 명시해줘야되는 필요성을 크게 감소시킨다.
  (이번 Chapter의 다른 곳에 논의 된 Bean Template과 같은 다른 메커니즘도 이와 관련되어 논의될 가치가 있습니다.)
- Autowiring은 객체가 변할 때 설정을 변경할 수 있습니다. 예로들어, 개발자가 어떠한 class의 의존성을 추가하고 싶다면 설정파일을 수정하지 않아도 자동적으로 의존성이 만족되어 집니다. autowiring은 개발 중에 Code base가 안정적 일 때 wiring하기 위한 변화 옵션을 무시하지 않고 특히 유용할 수 있습니다.

의존성 주입을 위해서 XML 기반의 configuration metadata를 사용할 때, 개발자는 \<bean>요소의 autowire 속성을 사용해서 bean 정의를 위해서 autowire mode를 명시할 수 있습니다. autowiring 기능은 4가지의 모드가 존재하고,  Bean마다 autowiring을 명시할 수 있습니다. 

다음은 4가지의 autowiring을 명시하기 위한 테이블 입니다.

**Autowiring modes**

- **no(default) :** autowiring 을 사용할 수 없다. Bean 참조는 ref 요소에 의해서 정의되어야 한다. 기본적인 setting의 변화는 큰 배포를 위해서 추천하지 않는다. collaborators를 명시하는 것은 제어를 더 좋게, 명확하게 할 수 있다.
  어느 정도 까지는 (To some extent), 시스템 구조를 문서화합니다.
- **byname : **property name에 의해서 Autowiring된다. Spring은 autowired가 필요한 property 로써 같은 name의 Bean을 찾습니다. 예로들어, 만약 Bean의 정의가 name에 의해서 autowire하기 위해 설정되어 지고 master property를(setMaster(..) method) 갖는다면, Spring은 master라는 이름이 붙여진 Bean 정의를 찾고 이를 사용하여 property를 설정합니다.
- **byType : **property 타입의 Bean 한개가(정확히 한개)  컨테이너에 있는 경우, property가 autowired되어 집니다. 하나 이상 존재한다면, Bean을 위해 byType autowiring을 사용할 수 없다는 치명적인(fatal) 예외가 던져질 것이다.  만약 어떠한 Bean도 match되지 않는다면, 어떠한 것도 일어나지 않을 것이다.
- **constructor :** byType과 유사하지만(Analogous) 생성자 인수를 이용하는 mode이다. 컨테이너에 생성자 인자 타입의 Bean이 정확히 하나가 아니면, 치명적인 에러가 발생되어질 것이다.

byType과 constructor autowiring 모드를 사용하면, 개발자는 배열과 타입이 정해진 collection을 연결지을 수 있다. 이러한 경우에서 type을 예측하는 container안에 모든 autowire 대상자는 의존성을 만족해야한다. 만약 예측된 key Type이 String 이라면 개발자는 강한 타입의 Map 인스턴스를 autowire할 수 있다. autowire된 Map 인스턴스의 값은 예측되는 유형을 만족하는 Bean Instance로 구성되어 있다. 그리고 Map 인스턴스의 key는 대응되는 Bean name을 포함한다.

### Limitations and Disadvantages of Autowiring

Autowiring은 프로젝트 전반에 걸쳐 꾸준히 사용되어질 때 가장 효과적이다. 만약 autowiring이 꾸준히 사용되지 않는다면, 개발자들은 한개 이상의 Bean을 정의하기 위해 autowiring을 사용하는 것에 혼동을 느낄 것이다.

**autowiring의 단점과 한계**

- property와 constructor-arg를 사용해서 의존성을 명확히 명시하면 항상 autowiring을 오버라이드 한다. 개발자는 간단한 primitives, Strings, Classes(간단한 properties의 배열)과 같은 properties를 autowire할 수 없다. 이러한 제한은 의도적으로 설계되어진 것입니다.
- Autowiring은 직접 wiring을 명시하는 것 보다 덜 정확하다. 앞의 표에서 언급했듯이, Spring은 예기치 않은 결과를 발생할 수 있는 모호한(ambiguity) 경우 추측을 피하기 위해 주의를 기울입니다. Spring이 관리하는 객체 사이의 관계는 더이상 명확히 문서화 되지 않습니다.
- Spring Container에서 문서를 생성할 수 있는 도구에는 wiring 정보가 제공되어지지 않을 수 있다.
- Container 내의 다중 Bean 정의에선 autowired될 setter 메소드 또는 생성자 인수로 지정된 타입과 일치할 수 있습니다. 배열, 컬렉션, 맵 인스턴스에서 이러한 것들이 필수적으로 문제는 아닙니다. 그러나 단일 값을 예측하는 의존성을 위해 이러한 모호성은 임의로 해결되면 안됩니다. 만약 사용할 수 있는 Bean의 정의가 유일하지 않다면, 예외가 발생할 것입니다.

후자의 시나리오에서, 개발자는 다양한 옵션을 갖습니다.

- 명시적 wiring을 위해서 autowiring을 포기(abandon)하십시요.
- **autowire-candidate 속성을 false**로 설정하는 것을 통해 Bean 정의에서 autowiring을 피하세요
- **\<bean/>요소의 primary 속성을 true**로 설정하는 것을 통해 single Bean 정의를 최우선 후보자로 지정하세요.
- annotation-based configuration 과 함 보다 세밀한 제어를 구현하세요.

### Excluding a Bean from Autowiring

각각의 Bean 마다, 개발자들은 autowiring으로부터의 Bean을 제외할 수 있다. XML 기반의 Spring에서, \<bean/> 요소의 autowire-candidate 속성을 false로 설정하면된다. Container는 특정한 Bean 정의를 autowiring infrastructure에서 사용할 수 없게 합니다.(@Autowired 애노테이션 포함)

> autowire-candidate 속성은 오직 type 기반의 autowiring에만 영향을 미친다. 명시된 Bean이 autowire candidate로 명시되어 있지 않음에도 불구하고 이름에 의한 reference에는 영향을 미치지 못한다. 만약 name이 일치하면 autowiring은 Bean을 주입할 것이다.

개발자는 Bean의 name에 대응해서 pattern-matching 기반의 autowire candidates를 제한할 수 있다. 가장 상위의 \<beans/> 요소는 default-autowire-candidates 속성내에 하나 이상의 패턴을 적용할 수 있다. 예로들어, Repository라는 마지막 이름을 가진 모든 Bean의 autowire 후보 상태를 제한하고자 한다면, *Repository라고 값을 제공하면 된다. 다중 패턴을 재공하고 싶다면, default-autowire-candidates 속성에 콤마로 구분하여서 패턴을 제공하면 된다. Bean 정의의 autowire-candidate 속성에 대한 명확한 true와 false의 값이 항상 우선합니다.(take precedence) 이러한 Bean에서는 패턴 매칭 규칙이 적용되지 않습니다.

개발자가 autowiring으로부터 다른 Bean이 주입되는 것을 완전히 막고 싶다면 이러한 기술은 매우 유용합니다. 제외된 Bean들이 autowiring을 사용함으로써 설정이 되어지지 않는다는 것을 의미하지 않는다. 이러한 Bean들 자체가 다른 Beane들을 autowiring 하기위한 후보자가 아니라는 것을 의미한다.

### 1.4.6 Method Injection

























































