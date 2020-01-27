# Spring Framework Core Technologies

이 레퍼런스는 스프링 프레임워크의 절대적으로 핵심적인(integral) 모든 기술에 대해서 다루는 문서 입니다.

이러한 핵심적인 기술 가운데 가장 중요한(Foremost) 기술은 Spring Framework의 **IoC(Inversion of Control) Container**입니다. Spring IoC 컨테이너를 철처히(thorough) 처리한 후 **Spring AOP(Aspect-Oriented Programming)** 기술에 대한 포괄적인(comprehensive) 내용을 다룹니다. Spring Framework는 개념적으로 쉽게 이해할 수 있고, Java Enterprise Programming에서 요구하는 AOP 요구 사항의 80%정도를 성공적으로 해결할 수 있는 자체의 AOP Framework를 가지고 있습니다.

AspectJ(현재 기능 측면에서 가장 풍부하고 자바 엔터프라이즈 영역에서 가장 성숙한 AOP 구현체) 와 Spring과의 통합도 제공합니다.

---

# 1. The IoC(Inversion of Control) Container

지금부터의 챕터는 Spring IoC Container에 대해서 다룰 것 입니다.

---

## 1.1 Introduction to the Spring IoC Container and Beans

**IoC는 의존성 주입(DI)이라고 알려져 있습니다. 객체가 생성자의 arguments, 팩토리 메서드에 대한 arguments, 팩토리 메서드로부터 리턴되거나 생성된 후의 객체 인스턴스에 설정된 특성을 통해서 의존성을 정의하는것을 의미한다.** IoC Conatiner는 Bean을 생성할 때 의존성을 주입합니다. 이 과정은 Service Loator pattern에 의한 방법이나, class들의 생성자를 직접적으로 사용함으로써 의존성의 인스턴스화 또는 위치를 제어하는 Bean 자체의 근본적인 반전 입니다.

***팩토리 메서드** 

> 1) 의도
> 객체를 생성하기 위해 인터페이스를 정의하지만, 어떤 클래스의 인스턴스를 생성할지에 대한 결정은 서브클래스가 내리도록 한다. 이처럼 구체적인 개체를 생성하는 부분을 분리하면 추상 팩토리 클래스에서는 어떠한 개체를 생성할 것인지에 대한 고민은 뒤로 미루고 개체를 사용하는 부분을 구현할 수 있다. '**팩토리 메서드란 추상 팩토리 클래스에 약속된 개체를 생성하는 메서드이다.**' 팩토리 메서드는 다른 이름으로 가상 생성자 (Virtual Constructor)라 한다.
>
> 2)예제
> 예제로 쓸 소재는 "로봇"입니다. 여러 종류의 로봇을 생산하는 "공장"도 있어야 할 것입니다. 따라서 로봇공장을 만들어서 예제를 한번 돌려 봅시다. 
>
> 1. **구조**
>    Robot(abstract class)
>
>    - SuperRobot
>    - PowerRobot
>
>    RobotFactory(abstract class)
>
>    - SuperRobotFactory
>    - ModifiedSuperRobotFactory
>
> 2. **Robot** 
>
>    - Robot(abstract class)
>
>       ```java
>       package pattern.factory;
>       
>       public abstract class Robot{
>         public abstract String getName();
>       }
>       ```
>
>    - SuperRobot
>
>       ```java
>       pakcage pattern.factory;
>       
>       public class SuperRobot extends Robot{
>       	@Override
>         public String getName(){
>           return "SuperRobot"
>         }
>       }
>       ```
>
>    - PowerRobot
>
>       ```java
>       package pattern.factory;
>       
>       public class PowerRobot extends Robot{
>         @Override
>         public String getName(){
>           return "PowerRobot";
>         }
>       }
>       ```
>
> 3. **RobotFactory**
>
>    - RobotFactory(abstact class)  : 기본 팩토리 클래스
>
>       ```java
>       package pattern.factory;
>       
>       public abstract class RobotFactory{
>         abstract Robot createRobot(String name);
>       }
>       ```
>
>    - SuperRobotFactory : 기본 팩토리 클래스를 상속 받아 실제 로직을 구현한 팩토리
>
>       ```java
>       package pattern.factory;
>       
>       public class SuperRobotFactory extends RobotFactory{
>         @Override
>         Robot createRobot(String name){
>           switch( name ){
>             case "super" : return new SuperRobot();
>             case "power" : return new PowerRobot();
>           }
>         }
>       }
>       ```
>
>    - ModifiedSuperRobotFactory : 로봇 클래스의 이름을 String 인자로 받아서 직접 인스턴스를 만들어 낸다.
>
>       **다시한번 JAVA API 문서를 통해서 이해해보자!!!! 매우 중요한 포인트이다!!!!**
>
>       ```java
>       package pattern.factory;
>       
>       public class ModifiedSuperRobotFactory extends RobotFactory{
>         @Override
>         Robot createRobot(String name){
>           try{
>             Class<?> cls = Class.forName(name); //클래스 정보를 담은 Class 클래스의 인스턴스를 얻어 온다.
>             Object obj = cls.newInstance();	// 클래스의 인스턴스를 생성한 다음 기본 생성자를 호출해 인스턴스를 초기화하고, 생성된 인스턴스를 Object 객체에 대입한다.
>             return (Robot)obj;	// obj객체를 다운 캐스팅(down casating)
>           }
>         }
>       }
>       ```
>
>       Class 클래스는 JVM에서 동작할 클래스의 정보를 묘사하기 위한 일종의 메타 클래스(Meta-Class) 입니다. 즉, JVM에 로드될 각 클래스들의 정보를 담고 있는 클래스입니다.
>
>       Class 클래스의 정적 메소드 forName()에 의해 레퍼런스 변수 cls는 some.package.SomeClass 클래스의 정보를 담은 Class 클래스의 인스턴스를 얻어옵니다.
>
>       이제 newInstance() 메소드를 호출하게 되면, 이 메소드는 some.pakcage.SomeClass의 인스턴스를 생성한 다음 기본 생성자를 호출하여 인스턴스를 초기화하고, 생성된 인스턴스를 리턴합니다.
>
>       **FactoryMain : 메인 프로그램**
>
>       ```java
>       package pattern.factory;
>       
>       public class FactoryMain{
>         public static void main(String[] args) {
>           RobotFactory rf = new SuperRobotFactory();
>           Robot r = rf.createRobot("super");
>           Robot r2 = rf.createRobot("Power");
>           
>           System.out.println(r.getName());	//SuperRobot
>           System.out.println(r2.getName());	//PowerRobot
>           
>           RobotFactory mrf = new ModifiedSuperRobotFactory();
>           Robot r3 = mrf.createRobot("pattern.factory.SuperRobot");
>           Robot r4 = mrf.createRobot("pattern.factory.PowerRobot");
>           
>           System.out.println(r3.getName());	//SuperRobot
>           System.out.println(r4.getName());	//PowerRobot
>         }
>       }
>       ```

**org.springframework.beans** 와 **org.springframework.context** 패키지는 SpringFramework의 IoC container의 기본 패키지이다. **BeanFactory** 인터페이스는 어느 타입의 객체라도 다룰 수 있는 향상된 설정 방법을 제공합니다. **ApplicationContext는 BeanFacotry를 상속받아 다양한 기능을 추가한 서브인터페이스이다.**

추가된 기능

- Spring의 AOP 기능과의 쉬운 통합
- 메세지 리소스 핸들링 (internationalization에 사용하기 위해)
- 이벤트 publication
- 응용 계층 별 contexts (ex. WebApplicationContext, ...)

짧게 말해서 **BeanFactory** 인터페이스는 configuration 프레임워크와 기본 기능을 제공한다. 그리고 **ApplicationContext** 인터페이스는 기업적으로 특화된 기술을 추가하였다. ApplicationContext가 Spring's IoC Container를 설명하기위해서 독점적으로 사용 될 것이다.

스프링 어플리케이션에서 중요한 역할을 수행하는 객체는 Spring IoC Container에 의해서 관리되어지고 **Bean**이라고 불린다.
**Bean** 은 Spring IoC Container에 의해 인스턴스화, 조립 및 관리되는 객체를 의미한다. **Bean**은 응용프로그램에서 많은 객체중 하나를 의미한다. **Bean**과 Bean들간의 종속성은 Container가 사용하는 configuration meatadata에 반영되어진다.

---

## 1.2 Container Overview

**org.springframework.context.ApplicationContext** 인터페이스는 Spring IoC Container를 나타내고, **Bean** 들을 조립, 인스턴스화, 구성한다. Container는 설정 메타데이터를 읽음으로써 인스턴스화, 구성, 조립할 객체에 대한 명령어를 가져옵니다. Container의 configuration metadata는 XML, Java annotations, Java Code에 의해서 작성되고,  이러한 configuration Metadata를 활용해서 어플리케이션을 구성하는 객체와, 객체간의 의존성을 표현할 수 있습니다.

ApplicationContext 인터페이스의 여러 구현은 Spring과 함께 제공되어 집니다. 독립형(stand-alone) 어플리케이션에서, ApplicationContext는 **ClassPathXmlApplication** 또는 **FileSystemXmlApplicationContext** 클래스의 인스턴스를 생성한다. configuration Metadata를 정의하기 위한 가장 전통적인 방법은 XML을 이용하는 것이다. 이 대신 Java Annotation 또는 metadata 형태의 코드를 사용할 수 있다. 

<img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200127123502518.png" alt="image-20200127123502518" style="zoom:50%;" />

위에 그려진 다이어그램은 스프링이 어떻게 동작하는지 hight-level의 뷰를 제공합니다. 어플리케이션의 클래스들은 configuration metadata와 결합하여 ApplicationContext가 생성 및 초기화 된 후 완전히 구성되고 실행가능한 시스템 또는 어플리케이션을 가질 수 있습니다.

configuration metadata는 일반적으로 간단하고 이해하기 쉬운 XML 형태로 제공 되어지며, Spring IoC Container의 기능과 중요한 개념을 전달하기위해 이번 쳅터에서 XML 형태의 configuration meatadata를 사용할 것입니다.

> XML 기반의 메타 데이터만 configuration Metadata로써 사용되는 것이 아닙니다.
> Spring Ioc container는 configuration metadata가 실제로 작성된 형식과는 분리(decoupled)되어 있습니다.
> 요즘날에 많은 자바 개발자들은 스프링 어플리케이션을 위해서 Java 기반의 configuraiton을 사용하고 있습니다.

Spring Container에서 사용하는 meta데이터의 다양한 형태

- **Annotation-based configuration** : spring 2.5 에서 소개되었다.
- **Java-based configuration** : Spring 3.0에서 시작되었고, Spring JavaConfig Project에서 제공되는 다양한 기능들이 이제는자바 SpringFramework의 핵심 부분중의 하나가 되었다. 게다가 어플리케이션 클래스 외부에 있는(external to) 빈들을 정의할 수 있습니다. 이러한 새로운 기능들을 사용하기위해 @Configruation, @Bean, @Import, @DependsOn과 같은 다양한 Annotation을 사용할 수 있습니다.

Spring Configuration은 Container가 관리할 수 있는 하나 이상의 Bean을 정의해야한다. XML기반의 configuration metadata는 \<beans>\</beans> 태그안에다가 bean들을 설정한다. Java Configuration은 일반적으로 @Bean 이라고 명명된 anntation메소드를 @Configuration이 붙은 Annotation 클래스 내에 사용한다.

이러한 bean의 정의는 어플리케이션을 구성하는 실제 객체와 일치한다. 일반적으로 서비스 계층의 객체, data access objects(DAOs), presentation 객체, Hibernamte의 sessionFactories와 같은 객체, JMS queues, 등등의 객체등을 정의한다. 일반적으로는 컨테이너에 세분화된(fine-grained) 도메인의 객체를 설정하지 않습니다. 이것은 보통 DAO의 역할과, 도메인 객체를 생성 및 로드할 business logic의 역할이기 떄문입니다. 

XML기반의 configruation metadata의 기본 구조를 참고해 봅시다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

1. **id 속성 :** 개개의 bean을 식별하는 문자열이다.
2. **class 속성 :** Bean의 타입을 정의하고 완전히 검증된 classname을 사용합니다.

### 1.2.2 Instantiating a Container

ApplicationContext 생성자로 제공되는 위치경로는 Container가 다양한 외부 리소스로부터 configruration metadata를 로드할 수 있는 리소스이다.

```java
ApplicationContext context = new ClassPathXmlApplicationContext("service.xml", "daos.xml");
```

Spring's IoC Container에 대해서 배우고 난 후, Spring's Resource 개념에 대해서 더 많이 배워야할 것이다.

**service.xml** : 서비스 계층의 객체들의 설정파일을 의미한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

**daos.xml** : data access objects 파일의 예시를 보여준다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

property 태그의 **name** 속성은 JavaBean property의 이름을 나타내는 요소이고, **ref** 요소는 또다른 Bean을 정의한 이름을 나타내는 요소이다. **id와 ref사이의 연결은 수집한 객체간의 의존성을 표현한다.**

import 태그를 사용해서 bean을 정의하는 파일들을 로드할 수 있다. (추천하지 않는다.)

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

#### The Groovy Bean Definition DSL

외부화된 configuration meatadta를 위한 예시, Bean을 정의하는 것은 또한 Grail Framework로부터 알려진 Spring's Groovy Bean Definition DSL 에 의해서 표현되어진다. 전형적으로 이러한 configruation은 .groovy 파일로부터 만들어지고 아래와 같은 구조를 갖는다.

```groovy
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

importBeans를 통해서 직접적으로 XML bean 정의 파일을 importing 할 수 있습니다.

### 1.2.3. Using the Container

ApplicationContext는 서로 다른 Bean의 registry와 그들의 종속성을 유지할 수 있는 향상된 팩토리 인터페이스이다.
**T getBean(String name, Class\<T> requiredType>)**메서드를 사용함으로써 Bean의 인스턴스를 검색할 수 있습니다.

ApplicationContext는 Bean의 definitions와 Bean을 접근할 수 있도록 합니다.

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

이러한 메소드들은 대부분 쓸일이 없음으로 개발자들은 metadata를 통해서 특정한 Bean의 의존성을 설정해주기만 하면된다.

---

## 1.3 Bean Overview

Spring IoC Container는 하나이상의 Bean들을 관리한다. 이러한 Bean들은 Container에게 제공한 configuration metadata와 함께 생성되어진다.

컨테이너 자체 내에서, 이러한 Bean들의 정의는 BeanDefinition 객체에 의해서 표현되어진다. BeanDefinition 객체는 다음의 metadata를 포함하고 있다.

- A package-qualified class name : Bean이 정의되어질 실제로 구현될 클래스를 의미한다.
- Bean behavioral configuration elements : 어떻게 Bean이 Container내에서 행동할지에 대한 metadata를 의미한다. (scope, lifecycle callbacks, ...)
- Bean이 작업을 수행하기 위해서 필요한 또 다른 Bean의 참조를 의미하는 metadata. 이러한 참조는 collaborators 또는 dependencies를 의미한다.
- 새롭게 생성된 객체에서 설정하기위한 configuration settings metadata를 포함한다.

이러한 메타 데이터는 각 Bean 정의를 구성하는 특정 세트로 변환됩니다. 다음 표는 이러한 속성을 설명합니다.

<img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200127123545398.png" alt="image-20200127123545398" style="zoom:50%;" />

특정 Bean을 작성하는 방법에 대한 정보가 포함된 Bean 정의 외에도, ApplicationContext 구현은 외부의 Container에서 생성된 객체의 등록을 허용한다. 이러한 것들은 ApplicationContext의 BeanFactory에 **getBeanFactory()** 메소드를 활용해서 접근하면 할 수 있다. getBeanFactory() 는 BeanFactory의 **DefaultListableBeanFactory** 구현을 리턴한다. DefaultListableBeanFactory 는 **registersingleton(..)** 과 **registerBeanDefinition(..)**을 통해서 이러한 Bean 등록을 지원해준다. 그러나 전형적인 어플리케이션은 기본적인 bean 설정 메타데이터를 통해 정의된 bean과 작업을 진행한다.

> autowiring 또는 다른 introspection 단계 동안 적절하게 추론(reason)하기 위해서 Bean 메타데이터와 수동적으로(manually) 제공되어야하는 singleton 인스턴스는 가능한 빨리 등록되어야 한다.
>
> 존재하는 메타데이터와 singleton instance에 대한 오버라이딩은 어느정도 지원이 되는 반면에, 런타임(팩토리에 실시간 access와 동시에)에 새로운 bean의 등록은 공식적으로 지원되지 않고 동시 접근에러를 발생시킬 것이다. Bean 컨테이너의 상태가 일치하지 않거나 둘 다 일 것이다.

### 1.3.1 Naming Beans

모든 빈은 하나이상의 식별자를 갖는다. 이러한 식별자는 Bean을 관리하는 container에서 오직 하나여야 한다. Bean은 보통 오직 하나의 식별자를 갖는다. 그러나 만약 식별자가 하나 이상으로 요구된다면, 이러한 추가적인 식별자는 별칭(aliases)로 불릴 수도 있습니다.

XML 기반의 configuration metadata에서는 Bean의 식별자를 명시하기 위해서 id 속성과, name 속성 또는 둘다를 사용해야 합니다. **id** 속성은 정확히 하나의 아이디만 명시하게 끔 한다. 관례적으로, 이러한 이름은 alphanumeric('myBean', 'someService', ...)이지만 특수문자도 포함할 수 있습니다. 만약 Bean을 위해서 또 다른 별칭을 소개하고 싶다면, **name 속성**을 사용할 수 있습니다. 

만약 name과 id 속성 명쾌하게(explicitly) 제공하지 않았다면, 그러한 Bean에게 특별한 name을 생성해 줄 것입니다.ref 요소 또는 Service Locatior 스타일 조회를 사용하여 이름으로 해당 Bean을 참조하고 싶은 경우, 무조건 이름을 제공해야 합니다. 
이름을 제공하지 않는 이유에서는 내부 Bean과 autowiring collaborators와 관련있습니다.

> **Bean Naming Conventions**
>
> Bean의 이름을 지을 때 인스턴스 필드 이름을 위한 Java Convention 표준이 사용되어 집니다. 즉, Bean의 이름은 소문자로 시작하고 그로부터 camel-cased로 표시되어진다. (ex. accountmanager, accountService, userDao, loginController, ...)

> classpath에서 component를 스캔하면, Spring은 이름이 없는 components에 bean의 이름을 생성합니다.
> 스프링이 사용하는 naming 규칙은 **java.beans.Introspector.decapitalize**에 정의되어 있다.

#### Aliasing a Bean outside the Bean Definition

Bean의 정의에서, Bean에게 id 속성과 name속성을 사용해서 하나 이상의 이름을 줄 수 있습니다. 이러한 이름은 같은 Bean과 동 동일한 별명일 수 있으며, 일부 상황에서 매우 유용하게 사용됩니다. 예를 들어, 응용 프로그램의 각 구성 요소가 해당 구성 요소 자체에 고유한 Bean 이름을 사용하여 공통 종속성을 참조하게 합니다.

Bean이 실제로 정의되어 있는 곳에 모든 별명이 명시되어 있다면 이것은 항상 적절한 상황이 아닙니다. 그러나 다른 곳에(elsewhere) 정의된 Bean의 별명을 도입하는 것이 때때로 바람직합니다. 이러한 경우는 configuration이 각각의 객체를 정의한 서브시스템에 분리되어 있는 큰 시스템의 경우에 해당합니다. XML 기반의 configuration metadata에서 \<alias> 태그를 사용해서 이러한 프로세스를 운영할 수 있습니다.

```xml
<alias name="fromName" alias="toName"/>
```

subsystem A를 위한 configuration metadata가 subsystemA-dataSource 라는 이름에 의해서 DataSource를 참조하고 있다면. 이러한 subsystem을 사용하는 메인 어플리케이션을 구성할 때 메인 어플리케이션은 myApp-dataSource에 의해서 DataSource를 참조할 것이다. 3가지의 이름이 모두 같은 객체를 가리키도록 하려면, configuration metadata에 다음과 같이 별명을 정의해야 한다.

```java
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

> **Java-Configuration**
>
> Javaconfiguration을 사용한다면, @Bean 애노테이션은 별명을 제공할 수 있습니다.

### 1.3.2 Instantiating Beans

Bean definition은 본질적으로 하나 이상의 객체를 생성하기 위한 레시피입니다. Container는 요청 될 때 명명된 Bean의 레시피를 보고, 해당 Bean definition에 의해 캡슐화된 configuration metadata를 사용해서 실제 객체를 생성한다.

만약 XML 기반의 configuration metadata를 사용한다면, bean의 요소에 class 속성에 인스턴스화 된 객체의 타입을 명시할 수 있다.  이 **class 속성**은 보통 의무적으로 작성해야한다.

아래와 같은 2가지의 방법으로 **Class property**를 사용할 수 있다.

- 일반적으로 컨테이너 자체가 Bean의 생성자를 호출함으로써 Bean을 직접적으로 생성하는 경우 구성할 Bean 클래스를 지정합니다.
- 객체를 생성하기위해 사용되는 정적 팩토리 메소드를 포함하는 실제 클래스를 명시해야한다.
   컨테이너가 클래스에서 정적 팩토리 메소드를 호출하여 Bean을 생성하는 경우에 실제 클래스를 명시해야 한다.

> **Inner Class names**
>
> 정적 중첩 클래스에 대한 Bean 정의를 설정하기 위해선 중첩 클래스의 binary name을 사용해야만 한다.
>
> 만약 com.example 패키지에 someThing이라 불리는 class가 있다고 가정하자, 그리고 이 SomeThing 클래스는 OtherThin이라 불리는 중첩 클래스를 가지고 있으면, Bean을 정의하기위한 class 속성의 값은 com.example.SomeThing$OtherThin이 되어야 한다.
>
> 이름에서 $ 문자는 외부의 클래스 이름으로부터 중첩된 클래스 이름을 구분하기 위해서 사용된다.

#### Instantiation with a Constructor

**생성자적 접근에의해서 Bean을 생성한다면, 모든 일반적인 클래스들은 Spring에서 사용가능하고 Spring과 호환됩니다. 즉, 개발중인 클래스를 특정한 인터페이스로 구현하거나, 특정한 방식(fashion)으로 코딩할 필요가 없다.** Bean 클래스를 간단히 명시해주는 것만으로도 충분하다. 그러나 **특정한 bean을 위해서 사용하는 IoC 타입에 의존한다면, default 생성자가 필요할 것이다.**

Spring IoC Container는 사실상 관리하고자 하는 모든 클래스들을 관리할 수 있습니다. JavaBean에만 국한되지 않습니다.
대부분의 Spring을 사용하는 개발자들은 기본 생성자와 컨테이너의 속성을 모델로한 적절한 setter 및 getter 만 사용하는 실제 JavaBean을 선호합니다. 하지만 여러분들이 사용하는 container는 bean의 특성을 가지지 않는 이국적인 class들도 가질 수 있습니다.

예로들어 개발자들이 JavaBean 스펙을 지키지 않는 legacy connection pool 을 사용해야 한다면 스프링은 이것 또한 관리해 줄 수 있습니다.

XML 기반의 configuration metadata 에서는 다음과 같이 bean class를 설정할 수 있습니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anthoerExample" class="examples.ExampleBeanTwo"/>
```

생성자에 인수를 제공하고, 객체를 생성 한 후 객체 인스턴스 속성을 설정하는 매커니즘에 대한 자세한 내용은 
**Injecting Dependency** 을 참고해보아라!

#### Instantiation with a Static Factory Method 

 정적인 팩토리 메서드로 생성한 Bean을 정의할 때, 팩토리 메소드 자신의 이름을 명시하기 위한 **factory-method 속성**과 정적인 팩토리 메소드를 포함하는 클래스를 명시하기 위해서 **class 속성**을 사용해라. 개발자는 이 메소드를 호출할 수 있고 객체를 리턴받을 수 있다. (이것은 생성자를 통해서 객체가 생성되는 것 처럼 다루어진다.) 이러한 Bean 정의의 한가지 용도는 레거시 코드에서 정적 팩토리를 호출하는 것입니다.

다음의 Bean 정의는 팩토리 메서드를 호출함으로써 생성된 빈을 명시하기위한 정의이다. 이 정의는 리턴되는 객체의 클래스 타입을 명시할 뿐만 아니라, 팩토리 매소드가 포함하는 클래스까지 명시하고 있다. 아래의 예제에서 createInstance() 매서드는 정적인 메소드 여야한다. 다음 예제는 어떻게 정적인 메소드를 정의하는지에 대한 예제를 보여준다.

```xml
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
```

  다음 예제는 이전 Bean 정의에서 작동하는 클래스를 보여줍니다.

```java
public class ClientService{
  private static ClientService clientService = new ClientService();
  private ClientService() {}
  
  public static ClientService createInstance() {
    return clientService;
  }
}
```

선택적인 인자를 팩토리 메소드에 제공하고 팩토리 메소드로부터 객체가 리턴된 이후에 객체 인스턴스의 속성을 setting하는 메커니즘에 대한 구체적인 방법론은 **Dependencies and Configuration in Detail** 을 보길 바란다.

#### Instantiation by Using an Instance Factory Method

**정적인 팩토리 메소드를 통해 인스턴스화 하는 것과 비슷하게, 인스턴스 팩토리 메소드의 인스턴스화는 Container로부터 존재하는 Bean의 non-static method를 호출해서 새로운 Bean을 만드는 방법이다.** 이러한 방법론을 사용하기 위해서는, class 속성은 비어있어야하고, **factory-bean 속성**에서 객체를 생성하기 위해 호출 할 인스턴스 메소드가 포함 된 현재(부모 또는 조상) 컨테이너의 Bean 이름을 지정하여야한다. **factory-method 속성**에 팩토리 메소드의 이름을 설정해야한다.

다음 예는 이러한 Bean들을 어떻게 설정하는지에 관한 예이다.

```xml
<!-- The Factory Bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
	<!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
      factory-bean="serviceLocator"
      factory-method="createClientServiceInstance"/>
```

위의 bean 설정과 일치하는 예제 클래스이다.

```java
public class DefaultServiceLocator{
  private static ClientService clientService = new ClientServerImpl();
  
  public ClientService createClientServiceIntstance(){
    return clientService;
  }
}
```

하나의 팩토리 클래스는 하나 이상의 팩토리 메소드를 가질 수 있다. 아래는 예시를 참고하자

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
	<!-- Inject any dependencies required by this locator bean-->
</bean>

<bean id="ClientService"
      factory-bean="serviceLocator"
      factory-method="createClientServiceInstance"/>

<bean id="accountService"
      factory-bean="serviceLocator"
      factory-method="createAccountServiceInstance"/>
```

아래의 코드는 위의 configuration metadata와 일치하는 클래스의 예시이다.

```java
public class DefaultServiceLocator{
  
  private static ClientService clientService = new ClientServerImpl();
  
  private static AccountService accountService = new AccountServiceImpl();
  
  public ClientService createClientServiceIntstance(){
    return clientService;
  }
  public AccountService createAccountServiceInstance(){
    return accountService;
  }
}
```

이러한 접근방법은 의존성 주입을 통해서 factory bean이 관리되어지고 설정되어진다는 것을 보여준다.
**Dependencies and Configuration in Detail** 에 대해서 봐보자

> Spring Documentation에서 "factory bean"은 스프링 컨테이너에서 설정되어지고 인스턴스나, 팩토리 메소드를 통해서 객체를 생성할 수 있는 Bean을 의미한다. 대조적으로 FactoryBean( 대문자 표기 ) 는 스프링의 특정한 FactoryBean을 의미한다.















































