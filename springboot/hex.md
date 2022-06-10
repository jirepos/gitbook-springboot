# 16진수 변환

문자열을 HEX 문자열로 변환하고 HEX 문자열을 원래의 문자열을 변환해야할 때가 있다. 이런 경우 com.sogood.core.codec의 HexConverter 클래스를 사용한다.

```java
  @Test
  public void testHex() {
    String src = "문자열abcdef123456789"; 
    // 문자열을 HEX 문자열로 
    String hexStr = HexConverter.strToHex(src);
    System.out.println(HexConverter.strToHex(src));
    // 다시 문자열로 
    String convStr = HexConverter.hexToString(hexStr);
    System.out.println(convStr);
  }
```
