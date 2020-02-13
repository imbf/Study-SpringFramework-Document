# 1.16 The `BeanFactory`

`BeanFactory` API는 Spring의 IoC 기능을 위한 기초를 제공한다. `BeanFactory`의 특정 계약은 Spring의 다른 부분 및 다른 관련 프레임워크와의 통합에 사용되며, 이것의 `DefaultListableBeanFactory` 구현은 상위 레벨 `GenericApplicationContext` 컨테이너 내의 주요 대표자 입니다.

`BeanFactory` 와 관련된 인터페이스들(`BeanFactoryAware`, `InitializingBean`, `DisposableBean`)은 다른 프레임 구성요소를 위한 주요한 통합 지점이다. 어떠한 애노테이션 또는 어떠한 리플렉션이 필요하지 않음으로, 컨테이너와 이것의 구성요소간에 매우 효율적인 상호작용이 가능합니다. 어플리케이션 레벨의 Beans는 같은 콜백 인터페이스를 사용하지만 일반적으로 선언적인(어노테이션 또는 프로그램적인 설정을 통한) 의존성 주입을 선호합니다. 

핵심 `BeanFactory` API 또는 해당 `DefaultListableBeanFactory` 구현은 사용될 설정 형식 또는 모든 구성 요소 어노테이션에 대해 가정을 하지 않습니다. 이러한 모든 특징(flavors)은 확장을 통해서 제공되어지며 공유 `BeanDefinition` 객체에서 core metadata representation으로써 작동합니다. 이러한 것들은 Spring Container가 유연하고 확장성 있다는 것을 보여주는 본질입니다.

---

### 1.16.1 `BeanFactory` or `ApplicationContext`?

> **What is the bootstrapping**
>
> 일반적으로 부트 스트랩은 외부 입력없이 진행되는 자체 시작 프로세스를 의미한다.
>
> 컴퓨터 기술에서는 전원을 켜거나, 일반 재설정 한 후 기본 소프트웨어를 컴퓨터의 메모리에 로드하는 프로세스, 특히 필요한 경우 다른 소포트웨어를 로드하는 운영 체제를 나타냅니다.

이번 섹션은 `BeanFactory` 와 `ApplicationContext` Container 계층과 bootstrapping에 대한 영향 사이의 차이에 관해서 설명한다.

개발자는 `GenericApplicationContext` 및 해당 서브클래스인 `AnnotationConfigApplicationContext`를 사용자 정의 bootstrapping을 위한 일반적인 구현으로 사용하지 않는 이유가 없는 한 `ApplicationContext`를 사용해야만 합니다. 모든 일반적인 목적(Configuration 파일 로드, classpath scan 유발, 어노테이션이 붙여진 클래스와 Bean 정의 등록을 위한 프로그래밍 적인 방법, 함수적인 Bean 등록(spring 5.0부터))을 위한 Spring의 핵심 Container의 가장 상위 주입지점이 있다.

**`ApplicationContext`는 `BeanFactory`의 모든 기능을 포함하기 때문에, Bean 처리에 대한 모든 제어가 필요한 시나리오를 제외하고 일반적으로 기본 `BeanFactory` 보다 추천된다.** `ApplicationContext` ( `GenericApplicationContext` 구현과 같은) 내에서, 여러 종류의 Bean들은 규칙에 의해서 탐색되어진다. 반면에 `DefaultListablebeanFactory` 는 모든 특별한 Beans에 대해서 알 수 없다.

많은 확장된 Container 기능의 경우(ex. annotation processing and AOP proxing), `BeanPostProcessor` 확장 지점은 필수적이다. 만약 기본적인 `DefaultListableBeanFactory`만 사용한다면 이러한 post-processors는 기본적으로 활성화 되지 못하고 탐색되어 지지도 못합니다. 어떠한 Bean정의 대해서 실제로 틀린게 없기 때문에 이러한 상황은 혼돈을 일으킬 수 있습니다. 오히려 이러한 시나리오에서는 추가 설정을 통해서 컨테이너를 완전히 bootstrapped 해야한다.

다음 테이블은 `BeanFactory` 와 `ApplicationContext` 인터페이스에 의해서 제공되어지는 기능을 나타냅니다.

**Table 9. Feature Matrix**

| Feature                                                 | `BeanFactory` | `ApplicationContext` |
| ------------------------------------------------------- | ------------- | -------------------- |
| Bean instantiation/wiring                               | Yes           | Yes                  |
| Integrated lifecycle management                         | No            | Yes                  |
| Automatic `BeanPostProcessor` registration              | No            | Yes                  |
| Automatic `BeanFactoryPostProcessor` registration       | No            | Yes                  |
| Convenient `MessageSource` access (for internalization) | No            | Yes                  |
| Built-in `ApplicationEvent` publication mechanism       | No            | Yes                  |

명확하게 Bean post-processor를 `DefaultListableBeanFactory`에 등록하고 싶다면, 개발자는 프로그래밍적으로 `addBeanPostProcessor`를 다음 예제처럼 호출할 필요가 있다.

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// Bean 정의와 함께 factory 생산

// 필요한 BeanPostProcessor 인스턴스를 등록
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// factory 사용
```

`BeanFactoryPostProcessor`를 기본 `DefaultListablebeanFactory`에 등록하기 위해서, 개발자는 해당 `postProcessBeanFactory` 메소드를 다음 예제와 같이 사용할 필요가 있다. 

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// Properties 파일로부터 몇가지의 property 값을 가져옴
PropertySourcesPlaceholderConfigurer cfg = new PropertySourcesPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// 실제로 교체를 수행
cfg.postProcessBeanFactory(factory);
```

위의 2가지 케이스에서, 명백히 등록하는 단계는 불편하다. 전형적인 기업형 설정에서 확장된 Container 기능을 위한 `BeanFactoryPostProcessor` 또는 `BeanPostProcessor`에 의존할 때, Spring-backed 어플리케이션에서 기본 `DefaultListableBeanFactory` 보다 다양한 `ApplicationContext`의 변형이 더 선호되어진다.

> `AnnotationConfigApplicationContext`는 post-proccessor가 등록된 모든 공통 애노테이션을 가지고 있으며 설정 annotation(ex. `@EnableTransactionManagement`)을 통해 cover 아래(underneath) 추가적인 processor를 가지고 올 수 있습니다. Spring의 애노테이션 기반의 설정 모델의 추상화 계층에서, Bean post-processor의 개념은 단순히(mere) 내부 Container 세부사항이 됩니다.























