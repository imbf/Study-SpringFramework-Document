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

> ##### **What is The Stream?**
>
> 영어 단어로 **스트림(Stream)**이란 흐르는 시냇물을 뜻하며, **컴퓨터 공학에서 스트림은 연속적인 데이터의 흐름 혹은 데이터를 전송하는 소프트웨어 모듈을 일컫는다.** 시냇물에 띄어진 종이배가 순서대로 흘러가듯이, 컴퓨터에서 스트림은 도착한 순서대로 데이터를 흘러 보낸다.
>
> 자바에서 **입출력 스트림**은 응용프로그램과 입출력 장치를 연결하는 소프트웨어 모듈이다. 응용프로그램은 입력 스트림과 연결하며, 입력 스트림은 키보드 장치를 제어하여 사용자의 키 입력을 받아 응용프로그램에게 전달한다. 또한 응용 프로그램은 출력 스트림에 연결하고 출력 스트림에 출력하면, 출력 스트림은 다른 끝단에 연결된 출력 장치를 제어하여 출력을 완성한다. 
>
> **스트림 입출력 방식에서, 자바 응용프로그램은 입출력 장치를 직접 제어하는 대신, 입출력 스트림 객체와 연결하여 쉽게 데이터 입출력을 실행한다.** 
>
> ##### **입출력 스트림의 특징**
>
> - **스트림의 양끝에는 입출력 장치와 자바 응용프로그램이 연결된다.**
>   - 자바 응용프로그램은 입력 스트림과 출력 스트림과만 연결하고, 입출력 스트림이 입출력 장치를 제어하고 실질적인 입출력을 담당한다.
> - **스트림은 단방향이다.**
>   - 입력 스트림은 입력 장치에서 응용프로그램으로 데이터를 전송하며, 출력 스트림은 응용프로그램으로부터 받은 데이터를 출력 장치로 전송한다. <u>두 가지 기능을 모두 가진 스트림은 없다.</u>
> - 스트림을 통해 흘러가는 기본 단위는 바이트나 문자이다.
>   - 자바의 스트림 객체는 바이트를 단위로 입출력하는 **바이트 스트림**과 문자 단위로 입출력하는 **문자 스트림**으로 나뉜다.
> - 스트림은 선입선출, 즉 FIFO 구조이다.
>   - 입력 스트림에 먼저 들어온 데이터가 응용프로그램에 먼저 전달되고, 출력 스트림은 응용프로그램이 출력한 순서대로 출력 장치에 보낸다.
>
> ##### 바이트 스트림과 문자 스트림
>
> 자바에서 입출력 스트림은 **문자 스트림(character stream)**과 **바이트 스트림(byte stream)**의 2종류로 나눈다. 문자 스트림은 문자만 다루기 때문에, 문자가 아닌 데이터기 출력되면 보이지 않거나 엉뚱한 기호가 출력되며, 문자가 아닌 정보가 입력되면 응용프로그램에게 엉뚱한 문자가 전달되는 오류가 발생한다. 참고로 자바에서 char 타입, 즉 문자 하나의 크기는 2바이트이다.
>
> 한편, 바이트 스트림은 바이트를 단위로 나누는 스트림으로, 스트림에 들어오고 나가는 정보를 **단순 바이너르(비트들)**로 다루기 때문에, 문자이든 이미지 바이트든 상관없이 흘려 보낸다.
>
> <img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200217195546653.png" alt="image-20200217195546653" style="zoom:50%;" />
>
> <img src="/Users/baejongjin/Library/Application Support/typora-user-images/image-20200217200850317.png" alt="image-20200217200850317" style="zoom:50%;" />
>
> 자바 플랫폼은 위의 그림과 같이 응용프로그램에서 바이트 단위나 문자 단위로 스트림 입출력을 할 수 있는 다양한 클래스를 제공한다. 바이트 스트림을 다루는 클래스는 공통적으로 이름 뒤에 **Stream**을 붙이고, 문자 스트림을 다루는 클래스는 **Reader/Writer**를 붙여 구분한다.
>
> 메모장으로 작성된 텍스트 파일이나 자바 소스 파일 같이 문자들로만 이루어진 파일을 읽고 쓰는 경우, 문자 스트림 클래스(FileReader, FileWriter)나 바이트 스트림 클래스(FileInputStream, FileOutStream) 모두 사용 가능하지만, 이미지나 오디오/비디오 파일의 데이터는 문자가 아닌 바이너리 정보들이므로, 이들을 읽거나 쓰는 경우 반드시 바이트 스트림 클래스(FileInputStream, FileOutputStream)를 사용해야 한다.
>
> ##### 스트림 연결
>
> **스트림은 서로 연결될 수 있다.** 자바 응용프로그램에서 바이트 스트림과 문자 스트림을 연결하여 사용해보자. <u>다음은 키보드로부터 문자를 입력받기 위해 표준 입력 스트림인 **System.in**과 **InputStreamReader** 스트림 객체를 연결하는 코드이다.</u>
>
> ```java
> InputStreamReader rd = new InputStreamReader(System.in);
> ```
>
> 이 코드는 문자 스트림 rd를 생성하고, 키보드와 연결된 표준 입력 스트림인 System.in을 연결한다. System.in은 InputStream 타입으로 바이트 입력 스트림이다. 이렇게 두 스트림이 연결되면 System.in은 사용자의 키 입력을 받아 바이트 스트림으로 내보내며, rd는 들어오는 바이트 스트림을 문자로 구성하여 응용프로그램에게 전달한다. 자바 응용 프로그램은 다음과 같이 rd.read()를 통해 사용자가 입력한 문자를 읽을 수 있다.
>
> ```java
> int c = rd.read();	// 입력 스트림으로부터 키 입력. c는 입력된 키의 문자 값
> ```
>
> 





`Resource` 인터페이스로부터의 몇가지의 가장 중요한 메소드는 다음과 같다.

- `getInputStream()` : 리소스를 찾아서 열고, 리소스에서 읽기 위한 `InputStream`을 반환한다. 각각의 호출은 새로운 `InputStream`을 리턴합니다. stream을 닫는건 호출자의 책임입니다.

- `exists()` : 





























