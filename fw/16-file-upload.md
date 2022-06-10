# File Upload 
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="/js/http/http.js"></script>
  <title>File Upload Demo</title>
</head>
<body>
  <form>
    <input id="fileInput" type="file"  name="pics"  />
  </form>
  <script>
    let fileInput = document.querySelector("#fileInput");
    fileInput.addEventListener('change', (e) => {
      let files = e.target.files; 
      let file = files[0];
      console.log(file.size);
      console.log(file.type);
      console.log(file.name);
      let reader = new FileReader(); 
      let formData = new FormData();
      formData.append("uploadFile", file);
      formData.append("realName", file.name);
        // ajax test
        let options = {
          url: "/demo/file/upload/", 
          method: "POST",
          isFileUpload: true,
         // 폼데이터로 업로드할 때에는 headers가 비어야 한다.
          headers: {
          },
          body: formData 
        };        

        ajax(options)
         .then((response) => {
           console.log(response); 
         })
         .catch((error) => {
            console.log(error);
         });
    })

  </script>
  
</body>
</html>
```
```java
@Controller
@Slf4j
@RequestMapping("/demo/file")
public class FileUploadDemo {

    @PostMapping(value = "/upload", produces = ResMediaType.JSON)
    public @ResponseBody
    FileInfo uploadFile(HttpServletRequest request, HttpServletResponse response
            ,@RequestParam("realName") String realName, @RequestParam("uploadFile") MultipartFile file) {

        String uploadDir = "f:/upload/";
        String guid = IdGenerator.generatorId();
        String ext = StringUtil.getFilenameExtension(file.getOriginalFilename());
        String imagePath = uploadDir + guid + "." + ext;
        String fileName = guid + "." + ext;
        try {
            FileUploader.upload(uploadDir, fileName, file);
        }catch(Exception e) {
            throw new CustomException(ErrorCode.OTHER_ERROR);
        }
        FileInfo imgPath = new FileInfo();
        imgPath.setAbsolutePath(imagePath);
        imgPath.setName(fileName);
        return imgPath;
    }//
}///~
```