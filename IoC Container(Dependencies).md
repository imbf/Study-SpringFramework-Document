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

지금부터는 이러한 예제의 변형에 대해서 참고해보자. 

































