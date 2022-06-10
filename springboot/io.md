# IO 처리

프로젝트에서 IO처리를 위해 다양한 라이브러리를 사용하게 된다. 이런 경우에 너무 많은 의존성이 생겨서 나중에 유지관리하기가 어려워 질수 있다. 가급적 라이브러리 의존성은 적게 유지하도록 한다. 기본적인 파일 및 디렉터리에 대한 처리는 아래의 라이브러리 또는 API만 사용하도록 한다

* [apache commons-io](https://www.baeldung.com/apache-commons-io)
* spring의 FileCopyUtils
* spring resource
* java.nio.file

그런데 위의 라이브러리를 각각의 소스 파일에서 사용하다보면 의존성이 각각의 java 클래스에 있게 되어서 유지관리하기가 역시 쉽지 않다. 파일 IO처리가 필요하다면 공통으로 사용할 수 있는 Utitliy 클래스르 하나 생성하여 그것을 이용하는 것이 좋다.

Apache Common IO를 사용하기 위해 pom.xml 파일에 의존성을 추가한다.

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.8.0</version>
</dependency>
```

우선 각각의 라이브러리를 사용하는 방법을 살펴보자.

## Resource Interface

Spring의 Resource 인터페이스는 Resource에 대한 접근을 추상화하기 위한 인터페이스이다.

```java
public interface Resource extends InputStreamSource {
    // 리소스가 있는지 확인
    boolean exists(); 

	 // Stream이 열렸는지 확인
    boolean isOpen(); 

	 //해당 리소스의 url을 가져온다
    URL getURL() throws IOException; 

	  //해당 리소스의 파일을 가져온다
    File getFile() throws IOException; 

	  //상대 경로로 다른 리소스를 가져옴
    Resource createRelative(String relativePath) throws IOException; 

	  //파일 이름을 가져온다
    String getFilename(); 

	  //설명을 가져온다
    String getDescription(); 
}
```

### Resource로부터 파일 정보 얻기

ClassPathResource, FileSystemResource 등 네트웍이 아닌 접근할 수 있는 경로에 있는 파일은 getFile() 메소드를 사용하여 File 객체를 구할 수 있다. getFilename()으로 파일명을 구할 수 있다.

아래는 ClassPathResource를 사용한 코드이다.

```java
 Resource resource  = new ClassPathResource("/public/images/gominsi.png");
 System.out.println(resource.getFilename());
 File f = resource.getFile();
 System.out.println(f.getAbsolutePath());
```

아래는 FileSystemResource를 사용한 코드이다.

```java
String path = "G:\\images\\gominsi.png";
Resource resource = new FileSystemResource(path);
System.out.println(resource.getFile().getAbsolutePath());
```

### Resource로부터 Stream 얻기

UrlResource를 사용하는 경우 Stream을 구하여 파일을 읽을 수 있다.

```java
Resource resource  = new UrlResource("http://image.c1.com/aaa.jpg");
InputStream is = resource.getInputStream();
byte[] contents = is.readAllBytes();
```

UrlResource를 사용하는 경우 파일명을 구할 수 있다.

```java
 String filename = resource.getFilename();
```

그러나 File 객체를 구할 수는 없다. 아래코드는 오류가 발생한다.

```java
File f = resource.getFile();
```

## UrlResource

UrlResource를 사용하는 경우 접두사를 쓸 수 있다. 그러나 classpath는 JUnit을 사용하는 경우 MalformedException이 발생했다.

* **classpath:** classpath:com/myapp/config.xml Loaded from the classpath.
* **file:** file:/data/config.xml Loaded as a URL, from the filesystem. \[1]
* **http:** http://myserver/logo.png Loaded as a URL.
* **(none)** /data/config.xml Depends on the underlying ApplicationContext.

```java
Resource resource  = new UrlResource("http://image.c1.com/aaa.jpg");
```

## 파일복사

파일을 복사할 때는 FielCopyUtils 클래스를 사용한다.

**copy(byte\[] in, File out)**\
Copy the contents of the given byte array to the given output File.

**copy(byte\[] in, OutputStream out)** Copy the contents of the given byte array to the given OutputStream.

**copy(File in, File out)** Copy the contents of the given input File to the given output File.

**copy(InputStream in, OutputStream out)** Copy the contents of the given InputStream to the given OutputStream.

**copy(Reader in, Writer out)** Copy the contents of the given Reader to the given Writer.

**copy(String in, Writer out)** Copy the contents of the given String to the given Writer.

**copyToByteArray(File in)** Copy the contents of the given input File into a new byte array.

**copyToByteArray(InputStream in)** Copy the contents of the given InputStream into a new byte array.

**copyToString(Reader in)** Copy the contents of the given Reader into a String.

## Temp Directory 구하기

FileUtils를 사용하면 Temporary directory를 구할 수 있다.

```java
File tempDir = FileUtils.getTempDirectory();
```

## 파일정보 구하기

FilenameUtils 클래스를 사용하면 경로 및 파일에 대한 정보를 얻을 수 있다.

```java
String fullPath = FilenameUtils.getFullPath(path);
String extension = FilenameUtils.getExtension(path);
String baseName = FilenameUtils.getBaseName(path);
```

[apache-commons-io](https://www.baeldung.com/apache-commons-io)를 참고한다.

## 파일 경로 처리

파일 경로에 대한 처리를 하거나 정보를 구할 때에는 java.nio.file.Paths와 java.ni.file.Path를 사용한다.

```java
 Path path = Paths.get("c\\resources\\public\\images");
 System.out.println( path.getFileName()); 
 System.out.println(path.getParent().getFileName());
 System.out.println(path.toString());
 URI uri = path.toUri();
 System.out.println(uri.toString());
 System.out.println(path.toAbsolutePath().toString());
 // 두개의 경로 조합하기 
 String dirpath = "c:/temp";
 String fileName = "aaa.jpg";
 Path path1 = Paths.get(dirpath); 
 Path path2 = path1.resolve(fileName);
 System.out.println(path2.toString());
```

## 공통 IO 처리 Utiltiy 클래스

앞에서 언급한 것처럼 최소한의 라이브러를 사용한다고 소스파일에서 라이브러리들의 API를 직접사용하다보면 유지관리하기 어려워질 있다. 공통 클래스를 만들자.

com.sogood.core.util 패키지에 IoUtils.java 를 생성한다. 이제 개발하면서 IO처리를 할 일이 있으면 이 클래스에 메소드를 추가하고 메소드를 이용하도록 한다.
