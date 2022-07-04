# ResponseBody 사용 


## Text 요청 
```jsx
      // Text 요청 테스트 
      let btnText = document.querySelector("#btnText");
      btnText.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-body/get-text' ,
          headers: {
            'Accept': 'text/plain', // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지 
            'Content-Type': 'text/plain',  // 서버에 어떤 형식의 데이터를 보내는지 알려줌
          }
        }
        ajax(options)
          .then((response) => {
            console.log(response);
            log(response);
          })
          .catch(error => {
          })
      }) //:

```

```java
    /** plain/text 반환 */
    @GetMapping(value = "/get-text", produces = MediaType.TEXT_PLAIN_VALUE + ";charset=utf-8")
    public @ResponseBody String getText(HttpServletRequest request, HttpServletResponse response) {
        String greeting = "반가워요";
        return greeting;
    }//:
```


## HTML 요청
```jsx
      // HTML 요청 테스트 
      let btnHtml = document.querySelector("#btnHtml");
      btnHtml.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-body/get-html' ,
          headers: {
            'Accept': 'text/html', // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지 
            'Content-Type': 'text/html',  // 서버에 어떤 형식의 데이터를 보내는지 알려줌
          }
        }
        ajax(options)
          .then((response) => {
            console.log(response);
            log(response);
          })
          .catch(error => {
          })
      }) //:
```

```java
    @GetMapping(value = "/get-html", produces = MediaType.TEXT_HTML_VALUE + ";charset=utf-8")
    public @ResponseBody String getHtml(HttpServletRequest request, HttpServletResponse response) {
        String html = "<strong>나는 강하다</strong>";
        return html;
    }//:
```


## JSON 요청
```jsx

      // JSON 요청 테스트 
      let btnJson = document.querySelector("#btnJson");
      btnJson.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-body/get-json' ,
          headers: {
            'Accept': 'application/json', // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지 
            'Content-Type': 'application/json',  // 서버에 어떤 형식의 데이터를 보내는지 알려줌
          }
        }
        ajax(options)
          .then((response) => {
            console.log(response);
            log(JSON.stringify(response));
          })
          .catch(error => {
          })
      }) //:
      
```

```java
    /** applicatin/json 반환 */
    @GetMapping(value="/get-json", produces = MediaType.APPLICATION_JSON_VALUE + ";charset=utf-8")
    public @ResponseBody DemoBean getJson(HttpServletRequest request, HttpServletResponse response) {
        DemoBean bean = new DemoBean();
        bean.setUserName("홍길동");
        bean.setAge(10);
        return bean;
    }//:

```

## PathVariable  
```jsx
      // @PathVariable 테스트
      let btnPath = document.querySelector("#btnPath");
      btnPath.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-body/path-variable/30/홍길동' ,
          headers: {
            'Accept': 'application/json', // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지 
            'Content-Type': 'application/json',  // 서버에 어떤 형식의 데이터를 보내는지 알려줌
          }
        }
        ajax(options)
          .then((response) => {
            console.log(response);
            log(JSON.stringify(response));
          })
          .catch(error => {
          })
      }) //:

```
```java
    /** @PathVariable 사용 */
    @GetMapping("/path-variable/{age}/{name}")
    public @ResponseBody DemoBean getJsonPath(HttpServletRequest request, HttpServletResponse response
            , @PathVariable int age, @PathVariable String name) {
        DemoBean bean = new DemoBean();
        bean.setUserName(name);
        bean.setAge(age);
        return bean;
    }//:

```


## RequestParam 
```jsx
      // @ReqeustParam 테스트
      let btnReqParam = document.querySelector("#btnReqParam");
      btnReqParam.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-body/request-param?age=30&name=홍길동' 
        }
        ajax(options)
          .then((response) => {
            console.log(response);
            log(JSON.stringify(response));
          })
          .catch(error => {
          })
      }) //:      
```


```java
    /** @RequestParam 사용 */
    @GetMapping("/request-param")
    public @ResponseBody DemoBean requestParam(HttpServletRequest request, HttpServletResponse response
            , @RequestParam("age") int age, @RequestParam("name") String name) {
        DemoBean bean = new DemoBean();
        bean.setUserName(name);
        bean.setAge(age);
        return bean;
    }//:
```


## RequestBody
```jsx
      // @RequestBody 테스트
      let btnReqBody = document.querySelector("#btnReqBody");
      btnReqBody.addEventListener('click', () => {
        let options = {
          method: 'POST', 
          url:'/demo/res-body/request-body',
          body: {
            userId: "a123",
            userName: "홍길동", 
            age: 30
          }
        }
        ajax(options)
          .then((response) => {
            console.log(response);
            log(JSON.stringify(response));
          })
          .catch(error => {
          })
      }) //:

```

```java
    /** applicatin/json 반환 */
    @PostMapping("/request-body")
    public @ResponseBody DemoBean requestBody(HttpServletRequest request, HttpServletResponse response
            , @RequestBody DemoBean bean) {
        System.out.println("test");
        return bean;
    }//:
```

## ModelAttribute 
```jsx
      // @ModelAttribute 테스트
      let btnModelAttr = document.querySelector("#btnModelAttr");
      btnModelAttr.addEventListener('click', () => {
        let options = {
          method: 'POST', 
          url:'/demo/res-body/model-attribute?userId=a123&userName=홍길동&age=30'
        }
        ajax(options)
          .then((response) => {
            console.log(response);
            log(JSON.stringify(response));
          })
          .catch(error => {
          })
      }) //:      
```

```java

    /** @ModelAttribue 사용. JSON 반환 */
    @PostMapping("/model-attribute")
    public @ResponseBody DemoBean modelAttribute(HttpServletRequest request, HttpServletResponse response
            , @ModelAttribute DemoBean bean) {
        return bean;
    }//:

```


## Image 다운로드 
```jsx

      // 이미지 다운로드  테스트
      let btnImageDown = document.querySelector("#btnImageDown");
      btnImageDown.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-body/get-image',
          headers: {
           'Accept': 'image/png', // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지 
           'Content-Type': 'application/octet-stream',  // 서버에 어떤 형식의 데이터를 보내는지 알려줌
          }
        }
        ajax(options)
          .then((response) => {
            let img = document.createElement('img')   // image element 생성
            document.body.append(img)  // 생성한 요소를 body에 추가
            img.src = URL.createObjectURL(response) // 이미지 소스 
          })
          .catch(error => {
          })
      }) //:
```

```java
    /** 이미지 다운로드 */
    @GetMapping(value = "/get-image", produces = MediaType.IMAGE_PNG_VALUE)
    public @ResponseBody byte[] getImage(HttpServletRequest request, HttpServletResponse response) throws IOException {
        InputStream in = getClass().getResourceAsStream("/public/demo/images/demo/gominsi.png");
        return IOUtils.toByteArray(in);
    }// :
```

## Octet Stream을 이용한 이미지 다운로드
```jsx

      // octet-stream 사용 이미지 다운로드  테스트
      let btnImageDownOctet = document.querySelector("#btnImageDownOctet");
      btnImageDownOctet.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-body/get-image-octet',
        }
        ajax(options)
          .then((response) => {
            let img = document.createElement('img')   // image element 생성
            document.body.append(img)  // 생성한 요소를 body에 추가
            img.src = URL.createObjectURL(response) // 이미지 소스 
          })
          .catch(error => {
          })
      }) //:
```
```java
    /** octet-stream으로 이미지 다운로드 */
    @GetMapping(value = "/get-image-octet", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
    public @ResponseBody byte[] getImageOctet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        InputStream in = getClass().getResourceAsStream("/public/demo/images/demo/gominsi.png");
        response.setHeader("Content-Disposition", "attachment;filename=gominsi.png");
        response.setHeader("Content-Transfer-Encoding", "binary");
        //response.setContentLength((int) file.length());
        return IOUtils.toByteArray(in);
    }// :
```      

## Octet Stream을 사용한 파일 다운로드
```jsx
     // octet-stream 사용 파일 다운로드 테스트
      let btnFileDown = document.querySelector("#btnFileDown");
      btnFileDown.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-body/get-image-octet',
          headers: {
           'Accept': 'application/octet-stream', // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지 
           //'Accept': 'image/png', // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지 
           'Content-Type': 'application/json',  // 서버에 어떤 형식의 데이터를 보내는지 알려줌
          }
        }
        ajax(options)
          .then((response) => {
            debugger
            // Blob 객체를 url로 만들때는 URL.createObjectURL() (opens new window)이 메소드를 사용.
            //const url  = window.URL.createObjectURL(new Blob(respone));
            const url  = window.URL.createObjectURL(response);
            const link = document.createElement('a');
            link.href = url 
            // download를 추가하면, 웹브라우저의 설정에 상관없이 링크 대상을 다운로드합니다. 
            // <a href="images/image.jpg" download>Go</a>는 
            // 는 image.jpg 파일을 다운로드합니다.
            link.setAttribute("download", "image.png");
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
          })
          .catch(error => {
          })
      }) //:
```
```java
    /** octet-stream으로 이미지 다운로드 */
    @GetMapping(value = "/get-image-octet", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
    public @ResponseBody byte[] getImageOctet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        InputStream in = getClass().getResourceAsStream("/public/demo/images/demo/gominsi.png");
        response.setHeader("Content-Disposition", "attachment;filename=gominsi.png");
        response.setHeader("Content-Transfer-Encoding", "binary");
        //response.setContentLength((int) file.length());
        return IOUtils.toByteArray(in);
    }// :

```