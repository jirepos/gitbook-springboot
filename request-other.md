# 기타 응답처리

'요청과 응답처리' 글에서 서비스를 호출할 때 JSON으로 데이터를 주고 받도록 표준을 정했다. 그러나 실제 앱을 개발하다보면 외부 라이브러리들을 사용하는 경우가 많은데 이때는 라이브러리만이 요구하는 데이터 형식이 있는 경우가 대부분이다. 모든 서비스 호출은 JSON을 사용하는 것이 원칙이지만 라이브러를 사용할 때는 예외로 한다.

## Text를 반환하기

라이브러리가 데이터로 text를 원할 때가 있다. 일단 스프링에서 어떻게 Text를 반환하는지 메소드를 살펴보자.

```java
@GetMapping(value="/getText",  produces="text/plain;charset=utf-8")
public @ResponseBody String getText(@RequstParam String docId) {
    return service.getText(docId); 
}
```

produces 속성을 사용하여 반환하는 미디어 타입이 application/text라는 것을 알려 주어야 한다. @ResponseBody 어노테이션을 사용하여 반환타입을 String으로 설정한다. 요청 파라미터를 처리하는 경우에는 @RequestParam 어노테이션을 사용할 수 있지만 가급적 @RequestBody를 사용한다. 그러나 개발자가 서비스 요청을 직접 정의할 수 @RequestBody JSONReqMessage를 사용한다.

## XML 반환하기

서버에 XML 요청을 할 때에는 Header에 다음과 같이 Content-Type을 설정해야 한다.

```
Content-Type: text/html; charset=utf-8
```

XML을 반환하고자 한다면 produces 속성에 "text/xml"값을 설정한다.

```java
@GetMapping(value = "/getXml", produces ="text/xml;charset=utf-8")
public @ResponseBody String getXml(HttpServletRequest request, HttpServletResponse response) 
	String xml = "<item>사과</item>";
	return xml;
}// :
```

## 이미지 반환하기

### 특정 타입의 이미지 요청

png 파일을 서버에 요청할 때에는 이미지 타입을 명확히 해야 한다. 요청할 수 있는 이미지 타입은 세가지다.

* png
* jpeg
* gif

```
Content-Type: image/png
```

produces 속성에 'image/png'를 설정한다. 바이너리의 반환타입은 byte\[]로 선언한다.

```java
@GetMapping(value = "/getPNG", produces = "image/png")
public @ResponseBody byte[] getPNG(HttpServletRequest request, HttpServletResponse response) hrows IOException {
	InputStream in = getClass().getResourceAsStream("/public/images/gominsi.png");
	return IOUtils.toByteArray(in);
}// :
```

### octet-stream으로 이미지 요청

특정 이미지 타입을 지정하지 않고 바이너리로 다운로드 받으려면 다음과 같이 Content-Type 헤더를 설정한다.

```
Content-Type: application/octet-stream
```

pdocues 속성도 Content-Type과 동일하게 설정한다.

```java
@GetMapping(value = "/getImage", produces = "application/octet-stream")
public @ResponseBody byte[] getImage(HttpServletRequest request, HttpServletResponse esponse) throws IOException {
	InputStream in = getClass().getResourceAsStream("/public/images/gominsi.png");
	return IOUtils.toByteArray(in);
}// :
```

## 파일 다운로드

파일을 다운로드 하는 요청의 Content-Type은 다음과 같이 설정한다.

```
Content-Type: application/octet-stream
```

### 크기가 작은 파일의 다운로드 처리

produces 속성도 Content-Type과 동일하게 설정한다. 파일다운로드도 이미지와 마찬가지로 byte\[]을 반환한다. 파일 다운로드를 처리할 때에는 응답헤더에 Content-Disposition 헤더를 추가해야 한다. 크기가 크던 작던 통일된 방식으로 다운로드를 처리하는 것이 유지관리하기에 좋다. 이 방법은 사용하지 않는다. 참고만 한다.

```java
@GetMapping(value = "/getFile", produces = "application/octet-stream")
public @ResponseBody byte[] getFile(HttpServletRequest request, HttpServletResponse esponse) throws IOException {
	response.setHeader("Content-Disposition", "attachment;filename=" + "gominsi.png");
	InputStream in = getClass().getResourceAsStream("/public/images/gominsi.png");
	return IOUtils.toByteArray(in);
}// :
```

### 크기가 큰 파일의 다운로드

파일 다운로드는 공통으로 처리해야 하는 코드가 많기 때문에 별도의 Utility 클래스를 만들어서 구현한다. 또한 위의 코드는 파일을 모두 읽어서 다운로드를 처리하기 때문에 큰 파일을 다운로드할 때에는 메모리 문제가 발생할 수 있다.

com.sogood.core.util 패키지에 FileDownloader.java 를 생성하고 다음과 같이 다운로드 메소드를 작성한다.

```java
public class FileDownloader {
  public static void download(HttpServletRequest request, HttpServletResponse response, File file, String toName)
      throws IOException {

    response.setContentType(ResMediaType.OCTET_STREAM);
    response.setHeader("Content-Disposition", "attachment;filename=" + toName);
    response.setHeader("Content-Transfer-Encoding", "binary");
    response.setContentLength((int) file.length());
    OutputStream out = response.getOutputStream();
    FileInputStream fis = null;
    try {
      fis = new FileInputStream(file);
      FileCopyUtils.copy(fis, out);
    } catch (FileNotFoundException e) {
      e.printStackTrace();
      throw e;
    } finally {
      if (fis != null) {
        try {
          fis.close();
        } catch (Exception e) {
        }
      }
    }
    out.flush();
  }// :
}///~  
```

다음과 같이 다운로드 메소드를 구현한다. 반환값이 없다는 것에 주의한다.

```java
@GetMapping(value = "/getFileDownload", produces = ResMediaType.OCTET_STREAM)
public void getFileDownload(HttpServletRequest request, HttpServletResponse response) throws IOException {
	File file = new ClassPathResource("/public/images/gominsi.png").getFile();
	FileDownloader.download(request, response, file, "gominsi3.png");
}// :
```

## Partial Content

HTML5에서는 video 태그를 지원한다. video 태그를 사용하면 동영상을 시청할 수 있다. 브라우저는 미디어 데이터를 내려 받을 때 파일 전체를 내려 받지는 않는다. 재생될지 안될지 모르는 페이지가 열린 시점에서 파일 전체를 내려 받는 것은 서버 네트워크, 브라우저 모두에게 부담이 된다. 따라서 당장 재생하는 것이 아니라면 필요할 것 같은 데이터만을 내려받는다.

하지만 브라우저는 미디어 리소스의 파일의 앞부분만을 내려받는 것은 아니다. 실제로는 미디어 파일의 맨 처음 부분과 마지막 부분을 내려 받는다. 이것은 해당 미디어 리소스 길이 정보를 미리 얻어둬야 하기 때문이다.

브라우저는 미디어 리소스를 서버에 요청할 때 HTTP 헤더에 Range 헤더를 첨부한다. Range 헤더에는 얻고 싶은 바이트의 범위를 지정한다.

```
...
Range: bytes=21010-
...
```

위의 코드는 브라우저가 서버에 21,010 바이트 이후의 데이터를 요구한 것이다. 이 요청에 대해서 서버는 다음과 같은 응답 헤더를 반환한다.

```
HTTP/1.1 206 Partial Content
Date: Wed, 15 Nov 2015 06:25:24 GMT
Last-Modified: Wed, 15 Nov 2015 04:58:08 GMT
Content-Range: bytes 21010-47021/47022
Content-Length: 26012
Content-Type: image/gif

... 26012 bytes of partial image data ..
```

일반적인 파일을 요청한 경우의 응답코드는 "200 OK"이지만, 바이트 범위를 한정지어 요청하면 응답코드는 "206 Partial Content"이다.

브라우저는 미디어 리소스 하나을 재생하기 전의 준비단계에서 이러한 요청을 몇 번이고 서버에 보낸다.

브라우저는 먼저 범위를 한정하지 않고 서버에 요청을 보낸다. 하지만 모든 미디어 데이터를 내려받는 것은 아니다. 어느 정도의 데이터를 얻으면 수신을 취소하고 나서 바로 파일의 마지막 부분을 요청한다. 그 후 맨 처음 에 내려받은 미디어 데이터의 이어지는 부분을 내려받는다.

또한 재생 중에도 한꺼번에 남은 미디어 데이터를 내려받지는 안흔다. 몇번에 걸쳐 부분적으로 미디어 데이터를 내려 받을 때도 있다.

부분적으로 데이터를 내려 주기 위해서 FileDownloader.java에 resourceRegion()를 다음과 같이 구현한다.

```java
private static ResourceRegion resourceRegion(UrlResource video, HttpHeaders headers) throws IOException 
	final long chunkSize = 1000000L;
	long contentLength = video.contentLength();
	HttpRange httpRange = headers.getRange().stream().findFirst().get();
	if (httpRange != null) {
		long start = httpRange.getRangeStart(contentLength);
		long end = httpRange.getRangeEnd(contentLength);
		long rangeLength = Long.min(chunkSize, end - start + 1);
		return new ResourceRegion(video, start, rangeLength);
	} else {
		long rangeLength = Long.min(chunkSize, contentLength);
		return new ResourceRegion(video, 0, rangeLength);
	}
}
```

ResponseEntity를 반환하는 메소드를 FileDownloader.java에 작성한다.

```java
public static ResponseEntity<ResourceRegion>  partialDownload(HttpServletRequest request, tpServletResponse response, HttpHeaders headers, UrlResource urlResource) throws IOException {
  //UrlResource video = new UrlResource("classpath:/public/media/" + fileName);
ResourceRegion region = resourceRegion(urlResource, headers);
return ResponseEntity.status(HttpStatus.PARTIAL_CONTENT)
		.contentType(MediaTypeFactory.getMediaType(urlResource).orElse(MediaType.APPLICATION_OCTET_STREAM).ody(region);
}
```

Controller 클래스에 ResponseEntity\<ResourceRegion> 을 반환하는 메소드를 작성한다.

```java
@GetMapping(value = "/getPartialVideo/{fileName}")
public ResponseEntity<ResourceRegion> getPartialVideo(HttpServletRequest request, ttpServletResponse response,
		@PathVariable("fileName") String fileName, @RequestHeader HttpHeaders headers) throws xception {
	UrlResource video = new UrlResource("classpath:/public/media/" + fileName);
	return FileDownloader.partialDownload(request, response, headers, video);
}// :
```

## produces의 MIME 설정

Controller의 메소드에 produces 속성을 이용하여 반환하는 Media Type을 설정해햐 하는데 여기에 charset을 추가해야 한다. 간편하게 처리하기 위해서 com.sogood.core.constatns 패키지에 ResMediaType.java 클래스를 생성하고 다음과 같이 작성한다.

```java
package com.sogood.core.constants;

import org.springframework.http.MediaType;

public class ResMediaType {
  private static final String charSet = ";charset=utf-8";
  public static final String TEXT = MediaType.TEXT_PLAIN_VALUE + charSet; 
  public static final String XML = MediaType.TEXT_XML_VALUE + charSet; 
  public static final String HTML = MediaType.TEXT_HTML_VALUE + charSet; 
  public static final String JSON = MediaType.APPLICATION_JSON_VALUE + charSet; 

  // image
  public static final String IMAGE_JPEG = MediaType.IMAGE_JPEG_VALUE; 
  public static final String IMAGE_PNG = MediaType.IMAGE_PNG_VALUE;
  public static final String IMAGE_GIF = MediaType.IMAGE_GIF_VALUE; 
  // binary 
  public static final String OCTET_STREAM = MediaType.APPLICATION_OCTET_STREAM_VALUE;
}///~
```

Controller 클래스에서 다음과 같이 사용할 수 있다.

```java
@GetMapping(value = "/getFile", produces = ResMediaType.OCTET_STREAM)
public @ResponseBody byte[] getFile(HttpServletRequest request, HttpServletResponse response) hrows IOException {
	response.setHeader("Content-Disposition", "attachment;filename=" + "gominsi.png");
	InputStream in = getClass().getResourceAsStream("/public/images/gominsi.png");
	return IOUtils.toByteArray(in);
}// :
```
