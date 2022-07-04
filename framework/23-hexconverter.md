# 16진수 변환

byte 배열을 16진수로 변환하고 16진수를 byte 배열로 변환하는 메서드를 제공한다.

## HexConverter

먼저 getHexConverter() 메서드를 사용하여 HexConverter를 가져온다.

### 16진수 문자열로 변환하기

strToHex() 메서드를 사용하여 문자열을 16진수 문자열로 변환한다.

```java
String str = "변환할 문자열"; 
String hexStr = DataConverter.getHexConverter().strToHex(str);
System.out.println(hexStr);
```

변환된 문자열은 다음과 같다.

```shell
ebb380ed9998ed95a020ebacb8ec9e90ec97b4
```

### 16진수 문자열을 문자열로 변환하기

hexToString() 메서드를 사용하여 16진수 문자열을 Java 문자열로 변환한다.

```java
String str = "변환할 문자열";
String hexStr = DataConverter.getHexConverter().strToHex(str);
System.out.println(hexStr);
String convertedStr = DataConverter.getHexConverter().hexToString(hexStr);
System.out.println(convertedStr);
```

변환된 문자열은 다음과 같다.

```shell
변환할 문자열
```
