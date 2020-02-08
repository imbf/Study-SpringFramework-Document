## 1.14 Registering a `LoadTimeWeaver`

**`LoadTimeWeaber`는 JVM에 클래스들이 로드 되어질 때 동적으로 클래스들을 변환하기 위해 Spring에 의해서 사용되어진다.**

load-time weaving을 활성화 하기 위해서, 개발자는 다음 예제에서 보여주는 것 처럼 `@Configuration` 클래스들 중 하나에 `@EnableLoadTimeWeaving` 애노테이션을 추가 해 줘야 한다.

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
   
}
```

대안으로, XML 설정을 위해서 개발자들은 `context:load-time-weaver` 요소를 사용할 수 있습니다.

```xml
<beans>
   <context:load-time-weaver/>
</beans>
```

`ApplicationContext`를 위해서 한번 설정 되어지면, 해당 `ApplicationContext`내에 존재하는 모든 Bean은 `LoadTimeWeaverAware`를 구현 할 것이다. 그럼으로(thereby) load-time weaver 인스턴스에 참조를 얻을 수 있다.
이러한 것들은 특히 Spring's JPA support와 결합하여 매우 유용하다. Spring's JPA support에서 JPA class transformation을 위해서 load-time weaving은 아마 거의 필수적이다. 더 자세한 내용을 참고 할려면 `LocalContainerEntitiyManagerFactoryBean` java document를 참조해라. 더 많은 AspectJ load-time weaving에 대한 정보를 원한다면 Load-time Weaving with AspectJ in the Spring framework(https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/core.html#aop-aj-ltw)를 참조해라.