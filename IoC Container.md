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
> 2)활용성
> 가나다라마바사
>





















