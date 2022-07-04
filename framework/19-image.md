# Image 처리 



## AWT 
AWT는 간단한 오퍼레이션을 수행하는 built-in Java Library이다. 


### 이미지 로드
```java
String imagePath = "path/to/your/image.jpg";
BufferedImage myPicture = ImageIO.read(new File(imagePath));
```




## References
[Working with Images in Java](https://www.baeldung.com/java-images)