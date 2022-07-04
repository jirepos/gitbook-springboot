# CryptUtils.java

CryptUtils 클래스의 내부 클래스를 사용하면 MD2, MD4, MD5, SHA-1, SHA-256, SHA-512등 다양한 암호화 알고 리즘을 사용하여 문자열을 암호화 할 수 있다.

## SHA-256 암호화

CryptUtils의 내장 클래스인 Digest를 사용하여 문자열을 SHA256으로 암호화 하는 코드이다. HexConverter를 사용하여 암호화된 byte 배열을 16진수 문자열로 변환한다.

```java
String strToEncrypt = "hello";
// digest()로 암호화된 배열을 구한다.
// AlgorithmEnum.SHA256을 사용하여 암호화할 알고리즘을 선택한다.
byte[] cryptedBytes = CryptUtils.getDigest().digest(strToEncrypt, AlgorithmEnum.SHA256);
// byte를 출력하기 위해서 HexConverter를 사용한다.
String hexString = DataConverter.getHexConverter().bytesToHex(cryptedBytes);
System.out.println(hexString);
```

다른 암호화 알고리즘을 선택하고 싶으면 AlgorithmEnum 클래스를 사용한다. 출력결과는 다음과 같다.

```shell
2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

HexConverter를 사용하지 않고 암호화된 문자열을 바로 16진수 문자열로 변환하려면 digestToHex() 메서드를 사용한다.

```java
// SHA256으로 암호화하고 16진수 문자열로 변환한다.
String hexStr = CryptUtils.getDigest().digestToHex(strToEncrypt,AlgorithmEnum.SHA256);
System.out.println(hexStr);
```

## AES 암호화

문자열을 암호화하고 복호화 하기 위해서 AES 알고리즘을 사용할 수 있다. CryptUtils 클래스는 AES 암호화를 지원하기 위해 Aes 내장 객체를 가지고 있다. 암호화하기 위해서는 Key가 필요한데, 128 bit 또는 256 bit를 선택할 수 있다.

암호화된 문자열을 Base64로 인코딩되어 반환된다. 암호화는 encrypt() 메서드를 사용한다.

```java
String strToEncrypt = "암호화복호화 테스트.";
// 키의 길이는 128 또는 256으로 선택할 수 있다. 
String key = "1234567890123456"; //  1286bit , 16byte 문자열
//String key = "12345678901234567890123456789012"; //  256bit , 32byte 문자열
Aes aes = CryptUtils.getAES();
String encryptedStr = aes.encrypt(key, strToEncrypt);
System.out.println("암호화된 문자열:");
```

결과는 다음과 같다.

```shell
rO5b+MFuQEg4MzD7iusanxD7C0whwkucUnSb7XWHrME=
```

암호화된 문자열을 복호화하기 위해서는 암호화활 때 사용한 동일한 Key를 사용해야 한다. decrypt() 메서드를 사용하여 복호화한다.

```java
String strToEncrypt = "암호화복호화 테스트.";
// 키의 길이는 128 또는 256으로 선택할 수 있다. 
String key = "1234567890123456"; //  1286bit , 16byte 문자열
Aes aes = CryptUtils.getAES();
String encryptedStr = aes.encrypt(key, strToEncrypt);
System.out.println("암호화된 문자열:");
System.out.println(encryptedStr);
String decryptedStr = aes.decrypt(key, encryptedStr);
System.out.println(decryptedStr);
```

복호화된 문자열은 다음과 같다.

```shell
암호화복호화 테스트.
```
