## 1.13 Environment Abstraction (환경 추상화)

**`Environment` 인터페이스는 애플리케이션 환경의 두 가지 주요 측면인 `profiles` 와 `properties` 를 모델링하는 Container의 통합된 추상화 입니다.**

<u>**profile은 지정된 profile이 활성화된 경우에만 Container에 등록될 명명된 논리 Bean 정의 그룹입니다.**</u> Bean들은 XML로 정의된 Profile 또는 annotations가 붙여진 profile에 할당되어질 것입니다. **profile과 관련있는 `Environment` 객체의 역할은 어떠한 profiles이 현재 활성화 되어 있고, 어떠한 profiles이 기본적으로 활성화 되어야 하는지 결정합니다.**

**Properties는 대부분의 모든 어플리케이션에서 중요한 역할을 수행하고 다양한 소스로부터 비롯될 수 있다**
다양한 소스 : properties.files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc `Properties` objects, `Map` objects, ...

**properties와 관련있는 `Environment` 객체의 역할은 Property source를 설정하고 그들(property source)로 부터 properties를 추출할 수 있게 편리한 service 인터페이스를 사용자에게 제공해준다.**

### 1.13.1 Bean Definition Profiles

