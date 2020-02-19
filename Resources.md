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

> #### **스트림(컴퓨팅)** - Wiki
>
> 컴퓨터 처리 환경에서 **스트림(stream)**은 시간이 지남에 따라 사용할 수 있게 되는 <u>일련의 데이터 요소를 가리키는 수 많은 방식</u>에서 쓰인다.
>
> - C 프로그래밍 언어에 기반을 둔 <u>유닉스 관련 시스템에서 스트림은 **개별 바이트나 문자열인 데이터의 원천**</u>이다.
>   스트림들은 파일을 읽거나 쓸 때, 네트워크 소켓을 거쳐 통신할 때 쓰이는 추상적인 개념이다. 표준 스트림들은 모든 프로그램에 이용할 수 있는 세 개의 스트림을  말한다.
> - 파이프라인(한 데이터 처리 단계의 출력이 다음 단계의 입력으로 이어지는 형태로 연결된 구조를 가리킨다.)은 장치에 **삽입된 제한이 없는 정보**뿐 아니라 스트림으로 이해할 수 있다.
> - 스킴 프로그래밍 언어 등에서 스트림은 느긋하게 계산하거나 지연 처리된 **일련의 데이터 요소**를 말한다. 스트림은 리스트와 유사하게 사용되지만 나눙에 이 요소들은 필요할 때에만 계산한다.
> - **스트림 프로세싱 - 병렬 컴퓨팅에서**, 특히 그래픽 처리에서 스트림이라는 용어는 소프트웨어뿐 아니라 하드웨어에도 적용된다.
>
> **java의 File Stream 또한 Stream 개념을 이용해서 만들어진 것 중 하나고 Java 8의 Stream api 또한 Stream 개념을 이용해서 만들어진 것 중 하나이다.(구분해서 생각하지 말자!!)**

> #### Stream - 11번가 신입사원 교육
>
> 1. **Stream이란?**
>    - **youTube 스트리밍과 비슷하게 생각**... 다 다운받고 영상으로 보는 것이 아닌, 내려받는대로 재생한다는 느낌!!
>    - **데이터 반복문을 간단하게 처리** - 개인적으로 이 기능이 우선적으로 와닿았음
>    - 멀티스레드 코드를 개발자가 구현하지 않아도 **데이터 병렬로 처리**할 수 있다.
>    - 사전적 용어 : 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소
>      - 연속된 요소 : Collection과 비슷하게 특정 요소 형식으로 연속된 값 집합(다른점은, Collection은 데이터, Stream은 계산 중점)
>      - 소스 : Collection, Array, File등의 데이터 제공 소스를 소비
>      - 데이터 처리 연산 : DB와 비슷한 연산, Functional Language지원 연산을 지원한다
> 2. **Stream과 Collection**
>    - Stream은 단 한번만 탐색 가능 -> Consume되면 끝
>    - Collection은 사용자가 직접 요소를 반복(명시적 개발자 의지)
> 3. **Stream 연산**
>    - 중간연산 : filter나 sorted와 같이 다른 Stream을 반환하는 연산 (return 타입이 그냥 Stream 이라고 생각하면 된다.)
>    - Stream 파이프라인에서 결과를 도출하는 연산(return 타입이 그냥 Stream이 아닌것이라도 생각하면 될  듯)
>
> #### Stream 활용
>
> 1. 스트림
>
>    - 스트림 개념
>
>    > **계산을 수행하기 위해, 스트림 작업은 스트림 파이프라인으로 구성됩니다. 스트림 파이프 라인은 소스(배열, 컬렉션, 함수 생성기, I/O 채널, ...), 0 개 이상의 중간 작업(스트림을 필터와 같은 다른 스트림으로 변환), 종료 작업(count 또는 forEach(Consumer)과 같은 결과 또는 부작용을 생성함)으로 구성되어 있다.** 스트림은 게으르다. 소스데이터에 대한 계산은 종료 작업이 시작될 때만 수행되며 소스 요소는 필요한 경우에만 사용되어집니다.
>
>    - Stateful operation & Stateless opertaion
>
>    > 중간 연산은 Stateless 와 Stateful 연산으로 나뉩니다. **filter 와 map과 같은 Stateless 연산은 새 요소를 처리 할 때 이전에 보았던 객체로부터 어떠한 상태도 유지하지 않습니다.** --즉, 각 요소는 다른 요소의 연산과 독립적으로 처리되어질 수 있습니다. **distinct, sorted와 같은 Stateful 연산은 새로운 요소를 처리할 때 이전에 보았던 객체로 부터의 상태를 포함합니다(incorporate).**
>    >
>    > Stateful 연산은 결과를 제공하기 전에 전체 input을 처리할 필요가 있습니다. 예로들어, 스트림의 모든 요소를 볼 때까지 스트림의 정렬 결과를 생성할 수 없습니다. 결과적으로, 병렬 프로그래밍에서, **stateful 중간 연산을 포함하는 몇몇의 파이프라인들은 데이터를 여러번 전달해야 하거나 중요한 데이터를 버퍼링하는게 필요할 수도 있습니다. 독점적으로 stateless 중간 연산을 보함하는 파이프 라인은 최소한의 데이터 버퍼링으로 병렬이든 순차적이든 단일 패스로 처리할 수 있습니다.**

> #### What is The Stream? - 블로깅
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

`Resource` 인터페이스로부터의 몇가지의 가장 중요한 메소드는 다음과 같다.

- `getInputStream()` : 리소스를 찾아서 열고, 리소스를 읽기 위한 `InputStream`을 반환한다. 각각의 호출은 새로운 `InputStream`을 리턴합니다. stream을 닫는건 호출자의 역할입니다.
- `exists()` : 물리적인 형태에서 이러한 리소스가 실제로 존재하는지 가리키는 `boolean`을 리턴합니다.
- `isOpen()` : 리소스가 열린 스트림을 가진 핸들을 나타내는지 여부를 가리키는 `boolean`을 리턴합니다. 만약 `true` 라면, `InputStream`은 여러번 읽혀지지 못하고 무조건 오직 한번만 읽혀질 것이다. 그리고 리소스 유출을 피하기 위해 닫혀질 것이다. `InputStreamResource`를 제외한 모든 일반적인 리소스 구현에 `false`를 리턴 한다.
- `getDescription()` :  이 리소스에 대한 설명을 리턴하고 리소스를 작성할 때 에러 발생을 위해서 사용되어질 것입니다. 이것은 종종 완전한 qualifed 파일 이름 또는 리소스의 실제 URL 입니다.

**다른 메소드들은 개발자가 리소스를 나타내는 `File` 객체 또는 실제 `URL`을 얻게 한다.**(만약 기본적인 구현이 호환가능하고 해당 기능을 지원한다면)

Spring자체는 `Resource` 추상화를 광범위하게 사용한다. 리소스가 필요할 때 많은 메소드 시그니쳐(메소드 이름 + 인자)에서 argument type으로써 Spring은 Resource 추상화를 광범위하게 사용한다. Spring API에서 다른 메소드들(다양한 `ApplicationContext` 구현에 대한 생성자)는 `String` 을 가져옵니다. 간단한 형식의 `String` 또는 간단한 폼은 해당 context 구현에 적합한 리소스를 생성하는데 사용 되거나 또는 문자열 경로의 접두사를 통해서 호출자가 특정 자원 구현을 생성하고 사용하도록 지정할 수 있습니다. (매우 어려우니까 다시 한번 복습하자!!!)

**`Resource` 인터페이스는 Spring에 의해서 또는 Spring과 함께 많이 사용되지만, 코드가 Spring의 다른 모든 부분에 대해 알지 못하거나 신경쓰지 않을 때 조차도 리소스에 접근을 위해서 자체 코드에서 일반적인 유틸리티 클래스로써 자체적으로 사용하는 것이 매우 유용합니다.** 이것은 코드를 Spring에 연결 짓는 반면에, 이것은 실제로 유틸리티 클래스의 작은 세트에만 결합합니다. 이 유틸리티 세트는 URL을 위한 유능한 대체제를 제공하며 이러한 목적으로 사용하는 다른 라이브러리와 동등한 것으로 간주될 수 있습니다.

> `Resource` 추상화는 기능을 바꾸지 않습니다. 가능하면 이를 감쌉니다. 예로들어, `UrlResource`는 URL을 감싸고 이러한 작업을 하기 위해서 강싸진 URL을 사용합니다.

---

## 2.3 Built-in Resource Implementations

Spring은 다음의 `Resource` 구현을 포함합니다.

- UrlResource
- ClassPathResource
- FileSystemResource
- ServletContextResource
- InputStreamResource
- ByteArrayResource

### 2.3.1 `UrlResource`

**`UrlResource`는 `java.net.URL`을 감싸고 일반적으로 files, HTTP target, FTP target 등의 URL로 일반적으로 접근할 수 있는 모든 객체에 접근하기 위해 사용되어집니다.** 모든 URL들은 표준화된 `String` 표현식을 가지고 있습니다. 적절히 표준화된 접두사는 다른 것들로부터 하나의 URL 타입을 가르키기 위해 사용되어진다. 이러한 것들은 파일시스템 경로를 접근하기 위한 `file:`, HTTP 프로토콜을 통해 리소스에 접근하기위한 `http:`, FTP를 통해 레소스에 접근하기 위한 `ftp:` 등을 포함한다.

**`UrlResource`는 명쾌하게 `UrlResource` 생성자를 사용하기 위해 Java code에 의해서 생성되지만 경로를 나타내는 문자열 인수를 가지는 API(`PropertyEditor`) 메소드가 호출 할 때 종종 암시적으로 생성됩니다.** 후자의 경우, JavaBeans `PropertyEditor`는 궁극적으로 어떠한 타입의 `Resource`를 생성할지 결정합니다. 만약 문자열 경로가 잘 알려진 접두사(ex. `classpath:`)를 포함한다면, 해당 접두사에 의해 적절하게 특화된 `Resource`를 생성합니다. 그러나 만약 접두사를 인지하지 못한다면, `PropertyEditor`는 String이 표준 URL 문자열이라고 가정하고 `UrlResource`를 생성합니다.

### 2.3.2. `ClassPathResource`

**`ClassPathResource` 클래스는 classpath로부터 얻어야하는 리소스를 나타낸다. `ClasspathResource`는 thread context class loader, 지정된 class loader, 지정된 클래스를 사용하여 자원을 로드한다.**

이 `Resource` 구현은 클래스 경로 리소스가 파일 시스템에 있지만(reside in) jar에 상주하고(서블릿 엔진 또는 환경이 무엇이든) 파일 시스템으로 확장되지 않은 클래스 경로 리소스가 아닌 경우 `java.io.File`로써 해결을 지원합니다. 이를 해결하기 위해 다양한 `Resource` 구현은 항상 `java.net.URL`로써 해결책을 지원합니다.

`ClassPathResource`는 `ClassPathResource` 생성자를 명시적으로 사용함으로써 자바 코드에 의해서 생성되지만 종종 경로를 나타내기 위해 `String` 인자를 취하는 API 메소드를 호출할 때 암묵적으로 생성되기도 합니다. 후자의 경우에, JavaBeans `PropertyEditor`는 문자열 경로에 있는 특별한 접두사인 `classpath:`를 인지하고 이러한 경우에  `ClassPathResource`를 생성합니다.

### 2.3.3 `FileSystemResource`

`FileSystemResource`는 `java.io.File`과 `java.nio.file.Path`를 처리하기 위한  `Resource`  구현 입니다. `FilesystemResource`는 `File` 또는 `URL`로써 분석을 지원합니다.

### 2.3.4 `ServletContextResource`

적절한 웹 어플리케이션의 root 디렉토리 내의 상대적인 경로를 해석하는 `ServletContext` 리소스들을 위한 `Resourece` 구현입니다.

`ServletContextResourece`는 스트림 접근과 URL 접근을 항상 지원하지만 웹 어플리케이션 아카이브가 확장되고 자원이 실제로 파일 시스템에 있는 경우에만 `java.io.File` 액세스를 허용합니다. 파일 시스템에서의 확장 여부 또는 데이터베이스와 같은 다른곳에서 또는 JAR로부터에 직접 접근은 Servlet Container에 실제로 의존합니다.

### 2.3.5 `InputStreamResource`

`InputStreamResource`는 주어진 `InputStream`을 위한 `Resource` 구현입니다. 이것은 어떤 특정한 `Resource` 구현이 적용되지 않는 경우에만 사용가능 해야한다. 특히, 가능한 경우 `ByteArrayResource` 또는 파일 기반의 `Resource` 구현을 선호하세요.

다른 `Resource` 구현과 대조적으로, `InputStreamResource`는 이미 열린 리소스에대한 서술자 입니다. 그러므로 `isOpen()` 으로부터 `true`를 리턴합니다. 만약 리소스 서술자를 어디에서나 유지할 필요가 있는 경우 또는 Stream을 여러번 읽어야할 필요가 있는 경우에 `InputStreamResource`를 사용하지 마세요.

### 2.3.6 `ByteArrayResource`

`ByteArrayResource`는 주어진 바이트 배열을 위한 `Resource` 구현입니다. `ByteArrayResource`는 주어진 바이트 배열을 위해서 `ByteArrayInputStream`을 생성합니다.

`InputStreamResource`의 단일 사용에 의지하는(resort to) 것 없이 주어진 바이트 배열로부터의 content를 로딩하기 위해 유용하다.























