## 1.11 Using JSR 330 Standard Annotations

Spring 3.0에서 시작으로, Spring은 JSR-330 표준 애노테이션(의존성 주입)을 지원합니다. 이러한 애노테이션은 Spring Annoptation으로써 같은 방법으로 스캔되어진다. 이러한 애노테이션을 사용하기 위해서 개발자는 자신의 classpath에 관련된 jar파일을 추가해야 할 필요가 있다.

> 만약 Maven을 사용한다면, `javax.inject` 아티팩트는 표준 Maven repository에서 이용가능하다. 개발자는 다음의 의존성을 자신의 pom.xml 파일에 추가할 수 있다.
>
> ```xml
> <dependency>
> 	<groupId>javax.inject</groupId>
>    <artifactId>javax.inject</artifactId>
>    <version>1</version>
> </dependency>
> ```

---

### 1.11.1 Dependency Injection with `@Inject` and `@Named`

`@Autowired` 대신에 개발자는 다음과 같이 `@javax.inject.Inject` 애노테이션을 사용할 수 있다.

```java
import javax.inject.Inject;

public class SimpleMovieLister {
   
   private MovieFinder movieFinder;
   
   @Inject
   public void setMovieFinder(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
   }
   
   public void listMovies() {
      this.movieFinder.findMovies(...);
      // ...
   }
}
```

`@Autowired` 애노테이션 처럼, 개발자는 `@Inject` 애노테이션을 필드 레벨, 메소드 레벨, 생성자 인자 레벨에서 사용할 수 있다. **게다가, 개발자는 주입 지점(Injection Point)을 `Provider` 로써 선언해 `Provider.get()` 호출을 통해 범위가 짧은 Beans에 대한 주문형(on-demand) 액세스 또는 다른 Beans에 대한 지연(lazy) 액세스를 허용 할 수 있다.**

다음의 예제는 이전 예제의 변형을 제공해준다.

```java
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {
   
   private Provider<MovieFinder> movieFinder;
   
   @Inject	// 주입 지점 : movieFinder
   public void setMovieFinder(Provider<MovieFinder> movieFinder){
      this.movieFinder = movieFinder;
   }
   
   public void listMovies() {
      this.movieFinder.get().findMovies(...);
      // ...
   }
   
}
```

**만약 주입되어져야 할 의존성에 대해 qualified name을 사용하고 싶다면, 개발자는 `@Named` 애노테이션을 사용해야한다.**
다음 예제를 참고 해 보자.

```java
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {
   
   private MovieFinder movieFinder;
   
   @Inject
   public void setMovieFinder(@Named("main") MovieFinder movieFinder){
   	this.movieFinder = movieFinder;
   }
   
   // ...
}

```

`@Autowired`와 마찬가지로, `@Inject` 애노테이션은 `java.util.Optional` 또는 `@Nullable` 애노테이션과 함께 사용되어질 수 있다. `@Inject`는 `required` 속성을 가지지 않기 때문에 이러한 것들은 `@Inject` 에서 더 적용되어질 수 있다. 다음 예제에서는 어떻게 `@Inject` 와 `@Nullable` 이 사용되어지는지에 대한 예제이다.

```java
public class SimpleMovieLister {
   
   @Inject
   public void setMovieFinder(Optional<MovieFinder> movieFinder){
      // ...
   }
}
```

```java
public class SimpleMovieLister {
   
   @Inject
   public void setMovieFinder(@Nullable MovieFinder movieFinder){
      // ...
   }
}
```

---

### 1.11.2 `@Named` and `@ManagedBean` : Standard Equivalents to the `@Component` Annotation

`@Component` 대신에, 개발자는 `@javax.inject.Named` 또는 `javax.annotation.ManagedBean` 을 사용할 수 있습니다.
다음예제에서 이러한 것들의 사용방법을 보여줍니다. (**사용자가 작성한 Bean 정의 방법**)

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")	// @ManagedBean("movieListener") Annotation 또한 사용가능하다.
public class SimpleMovieLister {
   
   private MovieFinder movieFinder;
   
   @Inject
   public void setMovieFinder(MovieFinder movieFinder){
      this.movieFinder = movieFinder;
   }
}
```

Component를 위한 name을 명시하지 않고 `@Component` 애노테이션을 사용하는 것은 매우 일반적인 방법이다. `@Named` 애노테이션 또한 이러한 비슷한 방법으로 사용 가능하다. 다음 에제에서 보여준다.

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {
   
   private MovieFinder movieFinder;
   
   @Inject
   public void setMovieFinder(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
   }
   
   // ...
}
```

`@Named` 또는 `@ManagedBean`을 사용할 때, 개발자는 Spring annotation을 사용할 때와 같은 방법으로 Component Scanning을 사용할 수 있다., 다음 예제에서 보여준다.

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig {
   // ...
}
```

**`@Component` 애노테이션과 대조적으로, JSR-330 `@Named`와 JSR-250 `@ManagedBean` 애노테이션들은 구성할 수 없다.**
**개발자는 사용자 component 애노테이션을 만드는데 Spring의 stereotype model을 사용해야만 한다.**

---

### 1.11.3 Limitations of JSR-330 Standard Annotations

표준 Annotation과 작업 시에, 개발자는 사용하지 못하는 몇가지의 중요한 기능에 대해서 알아야 한다. 다음 테이블에서 보여준다.

**Table 6. Spring component model elements versus JSR-330 variants**

| Spring              | javax.inject.*        | javax.inject.restrictions / comments                         |
| ------------------- | --------------------- | ------------------------------------------------------------ |
| @Autowired          | @Inject               | `@Inject` 'required' 속성을 가지지 않는다. 대신 Java 8의 `Optional` 을 함께 사용 할 수 있습니다. |
| @Component          | @Named / @ManagedBean | JSR-330은 구성가능한 모델을 제공하지 않습니다. 오직 명명된 component만을 식별할 수 있는 방법을 제공합니다. |
| @Scope("singleton") | @Singleton            | JSR-330의 기본 범위는 Spring의 `prototype`과 같습니다. 그러나, Spring의 일반적인 default와 함께 일관성(consistent)를 유지하기 위해서, 기본적으로 Spring Container내에 선언된 JSR-330 Bean은 `singleton` 입니다. `Singleton` 이외의 범위를 사용하려면, 개발자는 Spring의 `@Scope` 애노테이션을 사용해야 합니다. javax.inject 는 `@Scope` 애노테이션을 제공합니다. 그럼에도 불구하고 이 주석은 사용자 주석을 만드는 의도로만 사용됩니다. |
| @Qualifier          | @Qualifier / @Named   | `javax.inject.Qualifier` 은 단지 사용자 qualifier를 만들기 위한 meta-annotation 입니다. 구체적인 `String` qualifier(Spring의 값을 가지는 `@Qualifier` 처럼) `javax.inject.Named` 를 통해서 관련되어질 수 있습니다. |
| @Value              |                       | no equivalent                                                |
| @Required           |                       | no equivalent                                                |
| @Lazy               |                       | no equivalent                                                |
| ObjectFactory       |                       | `javax.inject.Provider`는 Spring의 `ObjectFactory`의 직접적인 대안으로 `get()` 메소드 name이 더 짧습니다. 이러한 것들은 Spring의 `@Autowired` 또는 어노테이션이 없는 생성자 와 setter 메소드와 결합하여 사용가능하다. |



























