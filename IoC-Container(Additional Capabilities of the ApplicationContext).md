## 1.15 Additional Capabilities of the `ApplicationContext`

챕터 소개에서 논의 되었던 것처럼, **`org.springframework.beans.factory` 패키지는 Beans을 관리하고 다루기 위해 프로그래밍 적인 방법들을 포함해 기본적인 기능들을 제공한다.** `org.springframework.context` 패키지는  `BeanFactory` 인터페이스를 상속하는 ApplicationContext 인터페이스를 추가하고, 이 외에도 어플리케이션 프레임워크를 지향하는 스타일로
추가적인 기능을 지원하기위해 다른 인터페이스들을 상속합니다. <u>**많은 사람들은 프로그래밍 방식으로 `ApplicationContext` 를 생성하지 않고 완전히 선언적인 방식으로 `ApplicationContext`를 사용하지만, 대신 `ContextLoader`와 같은 지원 클래스에 의존하여  Java EE web 어플리케이션의 일반적인 시작 프로세스의 부분으로써 ApplicationContext를 자동적으로 인스턴스화 한다.**</u> (ApplicationContext의 매우 중요한 내용!!)

보다 프레임워크 지향적인 스타일로 `BeanFactory` 기능을 향상시키기 위해, Context 패키지는 다음과 같은 기능을 제공해 준다.

- `MessageSource` 인터페이스를 통해서 i18n-style에서 메세지에 접근할수 있다.
- `ResourceLoader` 인터페이스를 통해서 URLs 와 files 같은 리소스에 접근할 수 있다.
- 이벤트 publication, 즉 `ApplicationEventPublisher` 인터페이스의 사용을 통해 `ApplicationListener` 인터페이스를 구현하는 Beans
- 여러 (계층적) contexts를 로딩하여, `HierarchicalBeanFactory` 인터페이스를 통하여 어플리케이션 웹 계층과 같은 하나의 특별한 계층에 각각의 contexts가 집중할 수 있게 만듭니다.

---

### 1.15.1 Internationalization Using `MessageSource`

