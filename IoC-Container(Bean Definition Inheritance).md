## 1.7 Bean Definition Inheritance

Bean의 정의에는 생성자 인수, property 값, Container의 특정 정보(ex. 초기화 메서드, 정적인 팩토리 메소드 이름, ...)등 많은 설정 정보를 포함할 수 있다. 자손 Bean의 정의는 설정 데이터를 부모 Bean의 정의로부터 상속할 수 있다. 자손 Bean의 정의에서는 몇개의 값 또는 다른 필요한 configuration information을 오버라이드 및 추가할 수 있다. 부모와 자손의 Bean 정의를 사용하는것은 typing을 절약하는데 매우 효과적입니다. 사실상(effectively), 이것은 템플릿의 한 형태입니다.

프로그래밍 방식으로 `ApplicationContext` 인터페이스를 사용하는 경우에, 자손 Bean 정의는 `ChildBeanDefinition` 클래스로 표시됩니다. 대부분 사용자는 이 수준에서 작업을 하지 않습니다. 대신에, `ClassPathXmlApplicationContext`와 같은 클래스에서 Bean 정의를 선언적으로 구성합니다. XML 기반의 configuration metadata를 사용할 때, 개발자는 `parent` 속성을 사용함으로써 자손 Bean의 정의를 나타낼 수 있습니다. parent Bean을 Parent 속성의 값으로써 명시하면 사용가능하다.

```xml
<bean id="inheritedTestBean" abstract="true" class="org.springframework.beans.TestBean">
	<property name="name" value="parent"/>
   <proeprty name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass" class="org.springframework.beans.DerivedTestBean" parent="inheritedTestBean" init-method="initialize">
	<property name="name" value="override"/>
   <!-- 값이 1인 age property는 부모 Bean으로부터 상속되었다. -->
</bean>
```

자손 Bean 정의는 지정되지 않는 경우 상위 Bean 정의의 클래스를 사용하지만 오버라이드할 수도 있습니다. 이러한 경우에는, 자손 Bean 클래스는 부모 Bean 클래스와 무조건 호환되어야 한다.(즉, 부모 property 값들을 수용할 수 있어야 한다.)

**하위 Bean 정의는 범위(scope), 생성자 인자 값, property 값, 새로운 값을 추가할 수 있는 메소드를 사용하여 부모의 method를 오버라이드 할 수 있도록 상속한다. 모든 범위, 초기화 메소드, 소멸 메소드, 명시된 정적인 팩토리 메소드는 해당하는 부모 Bean 의 설정을 오버라이드 합니다.**

**나머지 설정은 항상 자손 Bean 정의로부터 가져옵니다. (depends on, autowire mode, dependency check, singleton, lazy init)**

이전 예제에서 `abstract` 속성을 사용함으로써 부모 Bean 정의를 abstract 로써 명백히 지정했다. 만약 부모 Bean의 정의에서 명확하게 class를 명시하지 않았다면, 부모 Bean 정의를 `abstract`로써 명백히 명시하는것은 필요합니다.

다음 예제를 참고 해 봅시다.

```xml
<bean id="inheritedTestBeanWithoutClass" abstarct="true">
	<property name="name" value="parent"/>
   <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean" parent="inheritedTestBeanWithoutClass" init-method="initialize">
	<property name="name" value="override"/>
   <!-- name이 age이고 value가 1인 property 요소는 부모 Bean정의로부터 상속한다. -->
</bean>
```

부모 Bean은 불완전하고, `abstract` 라고 명시되어 있음으로 인스턴스화 되지 않는다. **Bean 정의에서 `abstract` 라고 명시되어 있을 때, 자손 정의를 위한 부모 정의로써 도와주는 template Bean definition으로만 오직 사용가능하다.** 이러한 `abstract` 부모 Bean을 다른 Bean의 property로 참조하거나 부모 Bean ID로 명백히 getBean()을 호출하여 상위 빈을 사용하려고 하면 오류를 리턴합니다. 이와 비슷하게, Container의  내부 `preInstantiateSingletons()` 메소드는 abstract로써 명시된 Bean 정의를 무시합니다.

> `ApplicationContext` 는 기본적으로 모든 Singleton Bean들을 사전 인스턴스화 합니다. 그러므로, 개발자가 부모 Bean 정의를 템플릿으로 사용할지, 특정한 클래스를 정의하는 definition으로 사용할지 abstract속성의 값을 true로 설정함으로써 확실히 해주는 것이 중요하다. 그렇지 않으면 application context가 실제로 `abstract ` Bean을 사전 인스턴스화 할것이다.



























