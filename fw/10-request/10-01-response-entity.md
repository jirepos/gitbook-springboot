# ResponseEntity 사용 



## text/plain 요청
```jsx
      // HTML 요청 테스트 
      let btnText = document.querySelector("#btnText");
      btnText.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-entity/get-text' 
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
    @GetMapping("/get-text")
    public ResponseEntity<String> getText(HttpServletRequest request, HttpServletResponse response) {
        String greeting = "반가워요";
        HttpHeaders headers = new HttpHeaders();
        //headers.setContentType(MediaType.TEXT_PLAIN);  // MediaType을 설정해야 한다.
        headers.setContentType(new MediaType("text", "plan", StandardCharsets.UTF_8)); // UTF-8 설정을 해 주어야 한다.
        return new ResponseEntity<String>(greeting, headers,  HttpStatus.OK);
    }//:
```

## text/html 요청
```jsx
      let btnHtml = document.querySelector("#btnHtml");
      btnHtml.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-entity/get-html' 
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
    @GetMapping("/get-html")
    public ResponseEntity<String> getHtml(HttpServletRequest request, HttpServletResponse response) {
        String html = "<strong>나는 강하다</strong>";
        HttpHeaders headers = new HttpHeaders();
        //headers.setContentType(MediaType.TEXT_HTML);  // MediaType을 설정해야 한다.
        headers.setContentType(new MediaType("text", "html", StandardCharsets.UTF_8)); // UTF-8 설정을 해 주어야 한다.
        return new ResponseEntity<String>(html, headers,  HttpStatus.OK);
    }//:

```      


## JSON 요청 테스트
```jsx
      // JSON 요청 테스트 
      let btnJson = document.querySelector("#btnJson");
      btnJson.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-entity/get-json' 
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
    @GetMapping("/get-json")
    public ResponseEntity<DemoBean> getJson(HttpServletRequest request, HttpServletResponse response) {
        DemoBean bean = new DemoBean();
        bean.setUserName("홍길동");
        bean.setAge(10);
        return new ResponseEntity<DemoBean>(bean, HttpStatus.OK);
    }//:
```





## PathVariable 테스트
```jsx
      // @PathVariable 테스트
      let btnPath = document.querySelector("#btnPath");
      btnPath.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-entity/path-variable/30/홍길동' 
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
    public ResponseEntity<DemoBean> getJson(HttpServletRequest request, HttpServletResponse response
            , @PathVariable int age, @PathVariable String name) {
        DemoBean bean = new DemoBean();
        bean.setUserName(name);
        bean.setAge(age);
        return new ResponseEntity<DemoBean>(bean, HttpStatus.OK);
    }//:
```



## RequestParam 테스트
```jsx
      // @ReqeustParam 테스트
      let btnReqParam = document.querySelector("#btnReqParam");
      btnReqParam.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-entity/request-param?age=30&name=홍길동' 
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
    public ResponseEntity<DemoBean> requestParam(HttpServletRequest request, HttpServletResponse response
            , @RequestParam("age") int age, @RequestParam("name") String name) {
        DemoBean bean = new DemoBean();
        bean.setUserName(name);
        bean.setAge(age);
        return new ResponseEntity<DemoBean>(bean, HttpStatus.OK);
    }//:
```


## RequestBody 테스트
```jsx
      // @RequestBody 테스트
      let btnReqBody = document.querySelector("#btnReqBody");
      btnReqBody.addEventListener('click', () => {
        let options = {
          method: 'POST', 
          url:'/demo/res-entity/request-body',
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
    public ResponseEntity<DemoBean> requestBody(HttpServletRequest request, HttpServletResponse response
            , @RequestBody DemoBean bean) {
        System.out.println("test");
        return new ResponseEntity<DemoBean>(bean, HttpStatus.OK);
    }//:

```



## ModelAttribute 테스트
```jsx

      // @ModelAttribute 테스트
      let btnModelAttr = document.querySelector("#btnModelAttr");
      btnModelAttr.addEventListener('click', () => {
        let options = {
          method: 'POST', 
          url:'/demo/res-entity/model-attribute?userId=a123&userName=홍길동&age=30'
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
    public ResponseEntity<DemoBean> modelAttribute(HttpServletRequest request, HttpServletResponse response
            , @ModelAttribute DemoBean bean) {
        return new ResponseEntity<DemoBean>(bean, HttpStatus.OK);
    }//:

```



## 이미지 다운로드 
```jsx
      // 이미지 다운로드  테스트
      let btnImageDown = document.querySelector("#btnImageDown");
      btnImageDown.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-entity/get-image',
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
    @GetMapping(value = "/get-image")
    public ResponseEntity<byte[]> getImage(HttpServletRequest request, HttpServletResponse response) throws IOException {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.IMAGE_PNG); // Mime Type을 image/png로 설정
        headers.set("Content-Disposition", "attachment;filename=" + "gominsi.png");
       // response.setHeader("Content-Disposition", "attachment;filename=" + "gominsi.png");
        InputStream in = getClass().getResourceAsStream("/public/images/demo/gominsi.png");
        return new ResponseEntity<byte[]>(IOUtils.toByteArray(in), headers, HttpStatus.OK);
    }// :
```


## Octet Stream 사용 파일 다운로드
```jsx
      // octet-stream 사용 파일 다운로드 테스트
      let btnFileDown = document.querySelector("#btnFileDown");
      btnFileDown.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/res-entity/get-image-octet',
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
    @GetMapping(value = "/get-image-octet")
    public ResponseEntity<byte[]> getImageOctet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM); // Mime Type을 image/png로 설정
        headers.set("Content-Disposition", "attachment;filename=" + "gominsi.png");
        //response.setHeader("Content-Disposition", "attachment;filename=" + "gominsi.png");
        InputStream in = getClass().getResourceAsStream("/public/images/demo/gominsi.png");
        return new ResponseEntity<byte[]>(IOUtils.toByteArray(in), headers, HttpStatus.OK);
    }// :
```