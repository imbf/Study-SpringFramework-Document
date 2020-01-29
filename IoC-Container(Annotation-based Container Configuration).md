## 1.9 Annotation-based Container Configuration

> #### Spring 설정에 XML 보다 annotation이 더 좋나?
>
> annotation-based configuration 의 소개에서는 XML 보다 Annotation 기반의 configuration이 더 좋느냐 하는 질물을 일으켰습니다. 우리는 "때에 따라 다르지라고" 짧게 대답할 수 있습니다. 좀 더 길게 답하자면 각각의 접근법은 장점과 단점을 보유하고 있습니다. 보통, 어떠한 전략이 더 적합한지 결정하는 것은 개발자에게 달려(be up to)있습니다. **개발자가 정의하는 방식에 따라서, annotation들은 선언에 많은 수의 context를 제공하며 짧고 간결한 설정을 제공합니다.** **반면에 XML은 소스코드를 조작하거나 다시 컴파일 할 필요없이 components 간에 연결짓는데 더 훌륭합니다.** <u>일부 개발자들은 소스에 가까운 wiring을 선호하는 반면에 다른 사람들은 주석이 달린 클래스가 더이상 POJO가 아니며, 구성이 분산되어 제어하기 더 어렵다고 주장합니다.</u>
>
> 어느 선택을 하던간, Spring은 두가지의 스타일과 심지어는 같이 섞어논 스타일까지도 수용할(accommodate) 수 있습니다. Spring의 JavaConfig option 을 통해서 지적할만한 가치가 있습니다. Spring은 annotations이 비침투 방법으로 target 요소 소스코드를 건드리지 않고 tooling 측면에서 사용되어질 수 있도록 한다. 모든 설정 스타일은 Spring Tool Suite로부터 지원되어진다.

**XML 설정의 대안은 annotation-based 설정에 의해서 제공되어진다. annotation-based configuration은 angle-bracket 선언 대신에 구성요소를 wiring하기 위해 bytecode metadata에 의존한다.** **Bean wiring을 설명하기위해 XML을 사용하는 것 대신에, 개발자는 관련된 클래스나, 메소드, 필드 선언에서 annotation을 사용함으로써 component class 자체에서 설정을 한다.** Example: The `RequiredAnnotationBeanPostProcessor` 에서 언급했듯이, `BeanPostProcessor`을 annotation과 함께 사용하는 것은 Spring IoC Container를 확장하는 가장 일반적인 방법이다. 예로들어, Spring 2.0에서는 `@Required` 에노테이션과 함께 필요한 properties을 시행할 수 있는 가능성이 도입되었습니다. Spring 2.5 에서는 스프링 의존성 주입을 운영하기 위해서 동일한 일반적인 접근방식을 따를 수 있도록 하였습니다. 근본적으로, `@Autowired` annotation은 Autowiring Collaborator에 설명된것처럼 같은 기능을 제공하고 더 섬세한 제어와 더 넓은 적용성을 가집니다. Spring 2.5는 또한 `@PostConstruct` 와 `@PreDestory` 같은 JSR-250 annotation을 지원을 추가하였습니다. Spring 3.0에서는 `@Inject` 와 `@Named` 같은 `javax.inject` 패키지에 포함된 JSR-330 annotations을 추가 지원합니다. 이러한 annotation에 관한 더 자세한 정보는 relevant section 에서  찾을 수 있습니다.

> **Annotation injection은 XML injection 이전에 수행되어진다. 그러므로, XML configuration은 양쪽 접근을 통해 wired 된 properties에 대한 annotation을 오버라이드 합니다.**

언제나 그렇듯이, 개별 Bean 정의로 Annotation을 등록할 수 있지만, 다음 태그를 포함하여 암시적으로 등록 할 수도 있습니다.(`context` namespace 포함에 주목해야 한다.)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

암시적으로 등록되어진 post-processors는 `AutowiredAnnotationBeanPostProcessor`, `CommonAnnotationBeanPostProcessor`, `PersistenceAnnotationBeanPostProcessor`, `RequiredAnnotationBeanPostProcessor`들을 포함한다.

> `<context:annotation-config/>` 는 오직 정의된 같은 applicationcontext 내의 Bean에서 annotation만 찾습니다. 즉, `DispatcherServlet`의 `WebApplicationConext`에 `<context:annotation-config/>`를 넣으면  서비스가 아닌 컨트롤러의 `@Autowired` Bean을 검사합니다. 자세한 정보는 DispatcherServlet을 참조하십시요.







































