# Spring Framework Core Technologies

이 레퍼런스는 스프링 프레임워크의 절대적으로 핵심적인(integral) 모든 기술에 대해서 다루는 문서 입니다.

이러한 핵심적인 기술 가운데 가장 중요한(Foremost) 기술은 Spring Framework의 **IoC(Inversion of Control) Container**입니다. Spring IoC 컨테이너를 철처히(thorough) 처리한 후 **Spring AOP(Aspect-Oriented Programming)** 기술에 대한 포괄적인(comprehensive) 내용을 다룹니다. Spring Framework는 개념적으로 쉽게 이해할 수 있고, Java Enterprise Programming에서 요구하는 AOP 요구 사항의 80%정도를 성공적으로 해결할 수 있는 자체의 AOP Framework를 가지고 있습니다.

AspectJ(현재 기능 측면에서 가장 풍부하고 자바 엔터프라이즈 영역에서 가장 성숙한 AOP 구현체) 와 Spring과의 통합도 제공합니다.

---

# 1. The IoC(Inversion of Control) Container

지금부터의 챕터는 Spring IoC Container에 대해서 다룰 것 입니다.

---

## 1.1 Introduction to the Spring IoC Container and Beans

**IoC는 의존성 주입(DI)이라고 알려져 있습니다. 객체가 생성자의 arguments, 팩토리 메서드에 대한 arguments, 팩토리 메서드로부터 리턴되거나 생성된 후의 객체 인스턴스에 설정된 특성을 통해서 의존성을 정의하는것을 의미한다.**



> *팩토리 메서드 
>
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
>       FactoryMain : 메인 프로그램
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
>
>       







































