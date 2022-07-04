# IO 처리 

IoUtils.java의 메소드들의 사용방법을 설명한다. 

## 파일 확장자 구하기

```java
  /** 파일이름에서 확장자만 돌려준다.  */
  public static String getFilenameExtension(String path) { 
    return  FilenameUtils.getExtension(path);
  }//:
```
```java
  /** 파일 확장자 구하기 */
  @Test
  public void testFileNameExtension() {
    String fileName = "aaa.txt";
    String ext = IoUtils.getFilenameExtension(fileName);
    System.out.println(ext); // 파일 확장자 출력 
    // txt
  }// :
```

## 디렉토리 관련 
### 디렉토리 생성 
```java

  /**
   * 경로에 있는 디렉토리를 생성한다. 
   * @param strPath   생성할 디렉토리 경로  "d:\\aaa\\bbb\\ccc"
   * @throws IOException
   *    생성 실패시 예외를 던진다. 
   */
  public static void createDirectories(String strPath) throws IOException  { 
    Path path = Paths.get(strPath);
    Files.createDirectories(path);
  }//:
```
```java
  /** Direcotires 생성 테스트  */
  @Test
  public void testCreateDirectories() throws Exception {
    String path = "d:\\aaa\\bbb\\ccc";
    IoUtils.createDirectories(path);
  }//:
```




## 파일관련
## File 객체 
### 클래스 패스 파일 생성  
```java


  /**
   * classpath에 파일을 읽어 반환한다.  경로는 'com/sogood/aaa.txt'와 같이 작성한다. 
   * @param path clasdspath 경로 파일명
   * @return
   *    File
   */
  public static File getFileInClasspath(String path) {
    try {
      ClassPathResource resource = new ClassPathResource(path);
      return resource.getFile();  
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }//:
```
```java
  /** 클래스패스에 파일을 찾고 File 객체를 반환한다. */
  @Test 
  public void testCreateFileInClasspath() {
    String classPath = "com/sogood";
    String fileName = "aaa.txt";
    File file = IoUtils.createFileInClasspath(classPath, fileName);
    if(file.exists()) {
      System.out.println("The file exists.");
    }else {
      System.out.println("The file doesn't exist.");
    }
    System.out.println(file.getName());
  }//:
```

### 클래스 패스의 파일 객체 반환 
```java

  /**
   * classpath에 파일을 읽어 반환한다.  경로는 'com/sogood'와 같이 작성한다. 
   * @param path clasdspath 경로
   * @param fileName 파일명 
   * @return
   *    File
   */
  public static File createFileInClasspath(String path, String fileName) {
    try {
      ClassPathResource resource = new ClassPathResource(path);
      return  new File(resource.getURI() + fileName);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }//:
```

```java
  /* 클래스패스에 파일을 찾고 File 객체를 반환한다 */ 
  @Test 
  public void testGetFileInClasspath() {
    String path = "aaa/bbb/ccc/aaa.txt"; 
    File f = null; 
    try {
      f = IoUtils.getFileInClasspath(path);  
    } catch (Exception e) {
      // 파일이 없으면 예외 발생 
      e.printStackTrace();; 
    }
  }//:
```

## 파일 읽기 
### byte[] 배열로 반환 
```java
  /**
   * 파일을 읽어 byte[]로 반환한다.  
   * @param f 읽을 파일
   * @return
   *    파일 내용의 byte[] 배열 
   */
  public static byte[] readFileToByteArray(File f)  { 
    try {
      return FileUtils.readFileToByteArray(f);  
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }//:
```
```java
  /** 클래스패스의 파일을 읽고 byte[] 로 반환  */
  @Test 
  public void testReadFileToByteArray() throws Exception  {
    String path = "public/index.html";
    File f = IoUtils.getFileInClasspath(path);  
    byte[] bytes = IoUtils.readFileToByteArray(f);
    String html = new String(bytes);
    System.out.println(html);

  }//:
```

### 라인 읽기 
```java

  /**
   * 파일을 읽어서 List<Sring>으로 반환한다. 
   * @param f 파일 객체 
   * @param charset  문자셋 
   * @return
   *    파일에서 읽은 문자열 
   */
  public static List<String> readLines(File f, String charset) {
    try {
      return FileUtils.readLines(f, charset);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }//:

```

```java

  /** 파일의 라인들을 읽어 List<String>으로 반환한다.  */
  @Test
  public void testReadLines() {
    String path = "public/index.html";
    File f = IoUtils.getFileInClasspath(path);  
    List<String> lines = IoUtils.readLines(f, "utf-8");
    lines.forEach( line -> System.out.println(line));
  }//:
```


### 파일내용 문자열로 반환 
```java
  /**
   * 파일을 끝까지 읽고 문자열로 반환한다.
   * @param f 파일 
   * @param charset  문자셋 
   * @return
   *    파일 내용 
   */
  public static String readFileToString(File f, String charset) {
    try {
      return FileUtils.readFileToString(f, charset);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }//:
```
```java
  /** 파일을 끝까지 읽어 문자열로 반환한다.  */
  @Test
  public void testReadFileToString(){ 
    String path = "public/index.html";
    File f = IoUtils.getFileInClasspath(path);  
    String s =  IoUtils.readFileToString(f, "utf-8");
    System.out.println(s);
  }
```




### Classpath 파일읽기 
#### 문자열로 읽기 
```java

  /**
   * 클래스 패스의 파일을 읽는다. 
   * @param pathAndFile 클래스경로와 파일이름. 예) com/sogood/aaa.txt 
   * @param charset 파일의 charset  
   * @return
   *  파일 내용
   */
  public static String readFileClasspathToString(String pathAndFile, String charset) {
    try {
      //ClassPathResource resource = new ClassPathResource(pathAndFile);
      // jar 파일에 묶여 있을 경우에는 getFile()이 동작하지 않는다. 
      //System.out.println(resource.getPath());
      //System.out.println(resource.getURI().getPath().
      //return FileUtils.readFileToString(ResourceUtils.getFile("classpath:" + pathAndFile), charset);

      InputStream is =  IoUtils.class.getResourceAsStream("/" + pathAndFile);
      InputStreamReader isr = new InputStreamReader(is);
      BufferedReader br = new BufferedReader(isr);
      StringBuilder sb = new StringBuilder();
      String line = "";
      while((line = br.readLine()) != null) {
        sb.append(line);
      }
      isr.close();
      br.close();
      return sb.toString();
    } catch (Exception e) {
      throw new RuntimeException(e);
    }

  }//:
```
```java
  /** Classpath의 파일의 내용을 문자열로 반환한다. */
  @Test 
  public void testReadFileClasspathToString() {
    String path = "public/index.html";
    String contents = IoUtils.readFileClasspathToString(path, "UTF-8"); 
    System.out.println(contents);
  }//:
```






## 파일 쓰기 

```java
  /**
   * 파일에 내용을 쓴다. 
   * @param f 쓸 파일 
   * @param data  쓸 내용 
   * @param charset 문자셋 
   */
  public static void writeStringToFile(File f, String data, String charset) {
    try {
      FileUtils.writeStringToFile(f, data, charset);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }//:
```
```java
  /** 파일에 내용을 쓴다.  */
  @Test 
  public void testWriteStringToFile() {
    String path = "d:\\aaa\\bbb\\ccc";
    File f = new File(path + "\\test.txt");
    String data = "hello"; 
    IoUtils.writeStringToFile(f, data, "utf-8");
  }
```


