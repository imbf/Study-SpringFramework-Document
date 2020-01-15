# Spring Framework Overview

**Spring은 Java Enterprise applications를 쉽게 개발할 수 있도록 만들어준다.** Spring은 enterprise 환경에서 Java 언어가 제공하는 모든 것들을 지원한다. JVM에서의 대체 언어로써 Groovy 와 Kotlin을 지원한다. Spring은 애플리케이션의 요구에 따라 다양한 종류의 아키텍처를 생성할 수 있는 유연성을 가진다. Spring Framework 5.1부터 Spring은 JDK 8 이상을 필요로하며 JDK 11까지 지원합니다. Java SE 8 update 60은 Java 8의 minimum patch release로써 추천되어지지만 일반적으로 가장 최근의 patch release를 사용하기를 추천한다.

Spring은 다양한 범위의 Application scenario를 지원한다. 기업에서 애플리케이션은 종종 오랫동안 실행되어야 하고, 개발자가 제어할 수 없는(is beyond) JDK 및 애플리케이션 서버에서 실행되어야 한다. 다른 어플리케이션들은 클라우드 환경에서 서버가 내장된 단일 jar파일로 실행할 수 있다. 또 다른 어플리케이션은 서버가 필요없는 독립형(standalone) 응용 프로그램일 수도 있습니다.

**Spring은 오픈소스 입니다.** Spring은 실무에서의 다양한 사용사례를 기반으로 지속적인 피드백을 제공하는 크고 활발한 커뮤니티를 가지고 있습니다. 이러한 커뮤니티는 Spring이 매우 오랜 기간동안 성공적으로 발전할 수 있게 도와주었습니다.

---

## 스프링이란 무엇을 의미하는가?

Spring은 아주 다양한 의미를 가지고 있다. Spring Projects 자체로써 사용되어질 수도 있습니다. 시간이 지남에따라서(over time) Spring Projects들은 Spring Framework위에서 개발되어 졌습니다. 대부분의 사람들이 생각하는 Spring은 전체 프로젝트의 집합을 의미한다. 지금 보고있는 reference document는 기초 즉, Spring Framework 자체에 집중할 것입니다.

Spring Framework는 여러가지의 모듈로 나뉘는데, 어플리케이션 개발에 필요한 모듈을 사용해서 개발할 수 있습니다.
SpringFramework가 가지는 여러가지 모듈 중 가장 핵심 모듈은 configuration model 과 dependency injection mechanism을 포함하는 핵심 컨테이너 모듈입니다. 그 밖에도(beyond that) SpringFramework는 다른 어플리케이션을 위해서 messaging, transactional data and persistence, web 등의 기본적인 기능지원을 해준다. SpringFramework는 Servlet을 기반으로하는 Spring MVC Framework 과 동시에 Spring WebFlux reactive web Framework 또한 제공한다.

*transaction : 데이터베이스의 상태를 변화시키기 위해 수행하는 작업의 단위

모듈에 대한 참고사항(A note about modules) : SpringFramework jars JDK 9's는 JDK 9's 모듈 경로("Jigsaw")에 배포할 수 있습니다. Jigsaw-enable 어플리케이션을 사용하기 위해서, SpringFramework 5 jars 에는 "자동 모듈 이름"  Manifest 항목이 있습니다. Manifest 는 jar artifact names로부터 독립적인 stable language-level module names가 정의되어져 있습니다. 물론 Spring's Framework jars는 JDK8과 9이상의 classpath에서도 잘 작동합니다.

---

## Spring 과 Spring Framework의 역사

Spring은 초기의 J2EE specifications의 복잡성을 해결하기 위해서 2003년에 시작되었습니다. 일부는 Java EE와 Spring이 경쟁관계에 있다고 생각하지만, 사실 Spring은 Java EE를 보완하는 프레임워크 입니다. Spring 프로그래밍 모델 JAva EE 플랫폼 specification을 포함하지 않습니다. 오히려 자바 EE로부터 신중하게 선택된 개별 specification이 Spring에 반영됩니다.

- Servlet API
- WebSocket API
- Concurrency utilities
- JSON Binding API
- Bead Validation
- JPA
- JMS
- transaction을 위한 JTA/JCA 셋업

**Spring Framework는 의존성 주입 및 일반적인 Annotation을 지원한다.** 어플리케이션 개발자들은 Dependency Injection과 Annotaion을 SpringFramework에서 제공하는 Spring-specific mechanism 대신에 사용할 수 있다.

SpringFramework 5.0 에서 부턴 Spring은 Java EE7(Servlet 3.1+, JPA 2.1+)을 최소한으로 필요로 한다. 동시에 런타임시 Java EE8 레벨에서 (Servlet 4.0, Json Binding API) 최신 API와의 즉시 통합을 제공합니다.

시간이 지남에따라, 어플리케이션 개발에서 Java EE의 역할은 진화하였고. Java EE와 Spring 초기에 어플리케이션은 application server에 배포되어지게 개발되었다. Spring Boot의 도움으로 오늘날의 Applications은 devops 와 cloud 방법에서 제작되어진다. Servlet Container가 내장되어 있고, 사소한 것들이 바뀌었습니다. Spring Framework 5 에서 WebFlux Application은 더이상 Servlet API를 직접적으로 사용하지 않고 Netty와 같은 Servlet Container가 아닌 서버에서 실행될 수 있다.

Spring은 계속해서 발전하고 혁신을 이뤄나갈 것이며, Spring에는 Spring Boot, Spring Security, Spring Data, Spring Cloud, Spring Batch 등과 같은 다양한 프로젝트들이 존재한다. 각 프로젝트에는 자체 소스코드 저장소, 이슈 트래커, release cadence가 있다는 사실을 기억해야 합니다. Spring project의 전체(complete) 목록은 https://Spring.io/projects 를 참조하세요!!

---

## 3. 디자인 철학 (Design Philosophy)

프레임워크에 대해서 배울 때, 프레임워크에 대한 것 뿐만 아니라 프레임워크가 따르는 철학에 대해서 아는 것도 중요하다.
Spring Framework의 원칙에 대해서 소개하겠다.

- **Spring은 모든레벨에서 선택을 제공합니다.** Spring을 사용하면 Design 결정s을 가능한한 늦게 할 수 있습니다. 예로들어,
   개발자는 프로젝트의 코드를 바꾸는 것 없이 configuration을 통하여 persistence provider를 바꿀 수 있습니다.
   인프라와, 타사 API 통합에도 편리함을 제공한다.
- **Spring은 다양한 관점을 수용합니다.** Spring은 유연성을 수용하고, 어떻게 그것을 수행할 것인지에 대해서 의견을 고집하지 (opinionated) 않습니다. Spring은 다양한 종류의 어플리케이션의 다른 관점에서의 요구를 지원합니다.
- **Spring은 이전 버전과의 호환성을 유지합니다.** Spring's evolution은 버전간의 중요한 변경사항(breaking change)이 거의 없습니다. Spring은 신중하게 선택된 타사 라이브러리 및 JDK 버전을 지원하여 Spring에 의존하는 응용 프로그램 및 라이브러리의 유지관리를 용이하게 합니다.
- **Spring은 API Design에 대해 신중히 한다.** Spring 개발 팀은 직관적이고(intuitive) 많은 버전과 수년에 걸쳐 유지되는(hold up) API를 만들기 위해 많은 시간과 노력을 쏟습니다.
- **Spring은 Code 품질을 위해서 high standard(높은 기준)을 설정합니다.** Spring Framework는 의미있고, 현재의 정확한 javadoc에 중점을 둡니다. Spring은 패키지간에 순환 의존성이 없는 clean code를 요구할 수 있는 매우 드문 project입니다.

---

## Feedback and Contributions

이슈에 대해서 디버깅하고, 진단하고, 질문을 하고 싶다면 우리는 StackOverflowf를 사용할 것을 제안한다. 우리는 사용할 수 있는 태그를 나열한 questions page를 가지고 있다. 만약 SpringFramework에 문제를 가지고 있거나, 기능(feature)을 제안하고 싶은 경우 GitHub Issues를 사용하세요.

마음속에 솔루션을 가지고 있거나, 수정사항을 가지고 있는 경우 Github에 pull request를 보내면 됩니다. 명심하십시요 (keep in mind)  가장 사소한(trivial) 문제를 제외하고(But) 모든것에 대해서, 논의가 진행되고(take place) 나중에 참조할 수 있도록 기록이 남겨질 것입니다.

---

## Getting Started

스프링을 시작한다면, Spring Boot 기반의 어플리케이션을 만듬으로써 Spring Framework를 사용할 수 있습니다. **Spring Boot는 Spring 기반의 production 준비가 된 application을 만드는 가장 빠른 방법을 제공해준다.** Spring Boot는 Spring Framework, configuration 보다는 convention을 중요시 합니다. SpringBoot는 최대한 빨리 가동하고 실행하도록 설계되었습니다.

https://start.spring.io를 사용해서 기본적인 프로젝트를 생성할 수 있고, Getting Started Guides중의 하나를 따를 수 있다.
뿐만아니라, 빨리 Spring 에 대해서 터득(digest)할 수 있다. 이러한 Guide는 특정한 문제를 해결할 때 고려해야 할 Spring 포트폴리오의 다른 프로젝트들도 다룹니다.



