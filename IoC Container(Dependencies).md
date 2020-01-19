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

Thingtwo와 ThingTree 클래스가 상속에 관련이 없는 클래스라고 가정하면, 어떤 잠재적인 모호함(ambiguity)도 존재하지 않는다. 게다가 다음의 configuration이 잘 동작하므로, 생성자 인수의 인덱스 또는 타입을 **\<constructor-arg/>** 요소에 명시할 필요가 없습니다.

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

























