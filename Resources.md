# 2. Resources

이번 챕터에서는 Spring이 어떻게 리소스를 다루는지 그리고 개발자는 Spring에서 리소스를 사용해서 어떻게 작업하는지에 대해서 다룰 것입니다. 이러한 것들은 다음과 같은 topic들을 포함합니다.

- Introduction
- The Resource Interface
- Built-in Resource Implementations
- the `ResourceLoader`
- The `ResourceLoaderAware` interface
- Resources as Dependencies
- Application Contexts and Resource Paths

---

## 2.1 Introduction

**불행히도 다양한 URL 접두사에 대한 Java의 표준 `Java.net.URL` 클래스와 표준 핸들러는 낮은 레벨의 리소스에 모두 접근하기 위해 적절하지 않습니다.** 예로들어, classpath 로부터 또는 `ServletContext`와 관련하여 확보해야하는 자원에 액세스하는 데 사용되어지는 표준화된 `URL` 구현이 없습니다. 반면에 특화된 `URL` 접두사를 위해서 새로운 핸들러를 등록하는 것은 가능하다(`http:`와 같은 접두사를 위한 기존 핸들러와 비슷하게), 이러한 것들은 일반적으로 꽤 복잡하고, `URL` 인터페이스는 여전히 바람직한 기능들(가리키는 자원의 존재를 확인하는 메소드와 같은)이 없습니다.

---

## 2.2 The Resource Interface

**Spring의 `Resource` 인터페이스는 저수준의 리소스에 접근을 추상화 하기위한 보다 더 유용한 인터페이스를 의미합니다.** 다음의 리스트는 `Resource` 인터페이스 정의를 보여준다.

```java
public interface Resource extends InputStreamSource {
   
   boolean exists();
   
   boolean isOpen();
   
   URL getURL() throws IOException;
   
   File getFile() throws IOException;
   
   Resource createRelative(String relativePath) throws IOException;
   
   String getFilename();
   
   String getDescription();
}
```

`Resource` 인터페이스의 정의에서 보여준 것처럼, 위 인터페이스는 `InputStreamSource` 인터페이스를 상속한다. 다음의 예제는 `InputStreamSource` 인터페이스의 정의를 보여준다.

```java
public interface InputStreamSource {

   InputStream getInputStream() throws IOException;
}
```

> **What is The Stream?**
>
> - 일반적으로 데이터, 패킷, 비트 등의 일련의 연속성을 갖는 흐름을 의미
>   - 음성, 영상, 데이터 등의 작은 조각들이 하나의 줄기를 이루며 전송되는 데이터 열(列)
>     - 호스트 상호간 또는 동일 호스트 내 프로세스 상호간 통신에서 큐에 의한 메세지 전달방식을 이용한 가상 연결 통로를 의미하기도 함 

`Resource` 인터페이스로부터의 몇가지의 가장 중요한 메소드는 다음과 같다.

- `getInputStream()` : 리소스를 찾아서 열고, 리소스에서 읽기 위한 `InputStream`을 반환한다. 각각의 호출은 새로운 `InputStream`을 리턴합니다. stream을 닫는건 호출자의 책임입니다.
- `exists()` : 
- `isOpen()` : 
- `getDescription()` : 



























