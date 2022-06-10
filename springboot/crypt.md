# 암복호화

암복호화에 대한 방법을 알아본다. 복호화가 가능한 암호화는 AES를 사용한다. 복호화가 불가능한 알고리즘은 MD5, SHA-256, SHA3-256을 사용한다. 이러한 알고리즘을 사용한 암호화를 위해서 MessageDigest를 사용한다. 로그인과 같이 민감한 정보를 다룰 때에는 RSA 암호화를 사용해야 한다.

프로젝트에서 일관된 암호화 로직을 작성할 수 있도록 암호화를 지원하는 클래스를 제공하는 것이 좋다.

## AES 암복호화

고급 암호화 표준(Advanced Encryption Standard)이라고 불리는 AES 암호 알고리즘은 DES를 대체한 암호 알고리즘이며 암호화와 복호화 과정에서 동일한 키를 사용하는 대칭 키 알고리즘이다. DES에 비해서 키 사이즈가 자유롭다. 즉, 가변 길이의 블록과 가변 길이의 키 사용이 가능하다.(128bit, 192bit, 256bit) 더 자세한 사항은 알려고 하지 말자. 머리 아프다.

AES 암호화를 위해서 CryptUtils 클래스를 사용한다. 먼저 암호화를 위한 키를 정의한다. 키의 길이는 128 bit의 암호화를 위해서는 16 Byes 문자열이 필요하고, 256 Bit 암호화를 위해서는 32 Byte의 문자열이 필요한다.

```java
// 키의 길이는 128 또는 256으로 선택할 수 있다. 
String key = "1234567890123456"; //  1286bit , 16byte 문자열
//String key = "12345678901234567890123456789012"; //  256bit , 32byte 문자열
```

암호화할 문자열을 정의한다.

```java
String strToEncrypt = "암호화복호화 테스트.";
```

AES 인스턴스를 구한다.

```java
Aes aes = CryptUtils.getAES();
```

encrypt() 메소드를 이용하여 암호화를 한다.

```java
String encryptedStr = aes.encrypt(key, strToEncrypt);
System.out.println("암호화된 문자열:");
System.out.println(encryptedStr);
```

다음과 같이 결과가 나올 것이다.

```
암호화된 문자열:
rO5b+MFuQEg4MzD7iusanxD7C0whwkucUnSb7XWHrME=
```

암호화할 때 사용했던 키를 사용하여 복호화 한다.

```java
String decryptedStr = aes.decrypt(key, encryptedStr);
System.out.println(decryptedStr);
```

다음과 같이 원래 문자열이 출력된다.

```
암호화복호화 테스트.
```

## 단방향 암호화

MD5, SHA-256 등의 암호화 알고리즘은 단방향 암호화 이다. 복호활를 할 수 없다. 이러한 암호화는 MessageDigest 클래스를 사용하는데 프로젝트에서 공통된 방식으로 코드를 작성하기 위해서 CryptUtils 클래스에 래핑한 매소들을 만들어 놓았다.

암호화를 하기 위해서 Digest 인스턴스를 구하고 인스턴스의 digest() 메소드를 사용하여 암호ghk한다. 사용할 수 있는 알고리즘은 AlgorithmEnum 클래스에 정리해 놓았다.

```java
public enum AlgorithmEnum {

	MD5("MD5"),
	SHA3256("SHA3-256"),
	SHA256("SHA-256");
	
	/** 알고리즘 이름 */
	private String algorithm;
	/** 알고리즘 이름을 반환한다. */
	public String getAlgorithm() {
		return this.algorithm;
	}
	AlgorithmEnum(String algorithm) {
		this.algorithm = algorithm;
	}
}///~
```

아래코드는 SHA256으로 암호화를 한다. 암호화된 바이트 배열을 HexConverter의 bytesToHex()를 사용하여 16진수 문자열로 변환할 수 있다.

```java
String strToEncrypt = "hello";
// digest()로 암호화된 배열을 구한다.
// AlgorithmEnum.SHA256을 사용하여 암호화할 알고리즘을 선택한다.
byte[] cryptedBytes = CryptUtils.getDigest().digest(strToEncrypt, gorithmEnum.SHA256);
// byte를 출력하기 위해서 HexConverter를 사용한다.
String hexString = HexConverter.bytesToHex(cryptedBytes);
System.out.println(hexString);
```

출력결과는 다음과 같다.

```
2CF24DBA5FB0A30E26E83B2AC5B9E29E1B161E5C1FA7425E73043362938B9824
```

암호화된 결과는 byte 배열로 반환하는데 digestToHex() 메소드를 사용하여 16진수 문자열로 변화할 수 있다.

```java
// SHA256으로 암호화하고 16진수 문자열로 변환한다.
String hexStr = CryptUtils.getDigest().digestToHex(strToEncrypt,AlgorithmEnum.A256);
System.out.println(hexStr);
```

### CryptUtils 소스

아래는 CryptUtils 전체 코드이다.

```java
package com.sogood.core.crypt;

import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.util.Base64;

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;

import com.sogood.core.codec.HexConverter;

public class CryptUtils {
  
	/** Aes 인스턴스 */
	public static Aes aes = new Aes();
	/** Digest 인스턴스 */
	public static Digest digest = new Digest();
	/** Aes 인스턴스를 반환한다. */
	public static Aes getAES() {
		return aes;
	}//:
	/** Digest 인스턴스를 반환한다.*/
	public static Digest getDigest() {
		return digest;
	}
	
	/**
	 * MessageDigest를 이용한 HASH 값을 생성한다.
	 * @author behap
	 *
	 */
	public static class Digest {
		
		/**
		 * 문자열을 HASH 값으로 변환한다. 
		 * @param strToDigest  변환할 문자열
		 * @param algorithm  알고리즘. 
		 * @return
		 * 		변환된 바이트 배열 
		 * @throws Exception
		 */
		public byte[] digest(String strToDigest, AlgorithmEnum algorithm)   {
			try {
				//final MessageDigest digest = MessageDigest.getInstance("SHA3-256");
				final MessageDigest digest = MessageDigest.getInstance(algorithm.getAlgorithm());
				final byte[] hashbytes = digest.digest(strToDigest.getBytes(StandardCharsets.UTF_8));
				return hashbytes;
			}catch(Exception e) {
				throw new RuntimeException(e);
			}
		}
		/**
		 * 문자열을 해쉬값으로 변환하고 Hex 문자열로 반환한다. 
		 * @param strToDigest 변환할 문자열
		 * @param algorithm 알고리즘
		 * @return
		 * 		변환된 문자열 
		 * @throws Exception
		 */
		public String digestToHex(String strToDigest, AlgorithmEnum algorithm)   {
			try {
				byte[] bytesDigested = digest(strToDigest, algorithm);
				return HexConverter.bytesToHex(bytesDigested);
			}catch(Exception e) {
				throw new RuntimeException(e);
			}
		}//:
		
	}//:
	
	/**
	 * AES 암호화를 지원한다.  암호화를 위한 키는 16바이트여야 한다. 
	 */
	public static class Aes { 
		/**
		 * 문자열을 암호화 한다. 
		 * 
		 * @param key  암호화할 키 
		 * @param strToEncrypt  암호화할 문자열 
		 * @return
		 * 		암호화된 문자열을 바이트 배열로 반환한다.
		 * @throws Exception
		 */

		public byte[] encryptToBytes(String key, String strToEncrypt) throws Exception  {
			SecretKeySpec secretKey = new SecretKeySpec(key.getBytes(), "AES");
			Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
			cipher.init(Cipher.ENCRYPT_MODE, secretKey);
			return cipher.doFinal(strToEncrypt.getBytes("UTF-8"));
			
		}//:
		
		/**
		 * 문자열을 암호화 한다. 
		 * 
		 * @param key  암호화할 키 
		 * @param strToEncrypt  암호화할 문자열 
		 * @return
		 * 		암호화된 문자열을 base64로 encode해서  반환한다
		 * @throws Exception
		 */
		public String encrypt(String key, String strToEncrypt) throws Exception  {
			String encryptedStr = Base64.getEncoder().encodeToString(encryptToBytes(key, strToEncrypt));
			return encryptedStr;
		}//:
		
		/**
		 * 암호화된 바이트 배열을 복호화 한다. 
		 * 
		 * @param key 암호화 키 
		 * @param bytesToDecrypt  복호화할 바이트 배열
		 * @return
		 * 		복호화된 바이트 배열 
		 * @throws Exception
		 */
		public byte[] decryptToBytes(String key, byte[] bytesToDecrypt) throws Exception {
			SecretKeySpec secretKey = new SecretKeySpec(key.getBytes(), "AES");
			Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5PADDING");
			cipher.init(Cipher.DECRYPT_MODE, secretKey);
			return  cipher.doFinal(bytesToDecrypt);
		}//:
		
		
		/**
		 * Base 64로 감싸진 문자열을 복호화 한다. 
		 * 
		 * @param key 암호화 키 
		 * @param strToDecrypt  복호화할 문자열 
		 * @return
		 * 		복호화된 문자열 
		 * @throws Exception
		 */
		public String decrypt(String key, String strToDecrypt) throws Exception {
			byte[] bytesToDecrypt = Base64.getDecoder().decode(strToDecrypt);
			String decreptedStr = new String(decryptToBytes(key, bytesToDecrypt));
			return decreptedStr;
		}//:
	}///~ AES
}
```

## RSA 암호화

프로젝트에서는 로그인이나 기타 중요한 내용을 서버로 전송할 때 RSA 방식의 암호화를 사용할 것이다. RSA에 대한 간단한 개요는 아래를 참고한다.

> RSA 암호는 공개키 암호시스템의 하나로, 암호화뿐만 아니라 전자서명이 가능한 최초의 알고리즘으로 알려져 있다. RSA가 갖는 전자서명 기능은 인증을 요구하는 전자 상거래 등에 RSA의 광범위한 활용을 가능하게 하였다.

> RSA는 두 개의 키를 사용한다. 여기서 키란 메시지를 열고 잠그는 상수(constant)를 의미한다. 일반적으로 많은 공개키 알고리즘의 공개키(public key)는 모두에게 알려져 있으며 메시지를 암호화(encrypt)하는데 쓰이며, 암호화된 메시지는 개인키(private key)를 가진 자만이 복호화(decrypt)하여 열어볼 수 있다.

간단히 설명하자면 공개키를 클라이언트에게 전송하고 클라이언트가 공개키로 암호화를 해서 전송하면 서버는 개인키로 전송된 데이터를 복호화 한다.

### 키생성

#### KeyPair 생성

암호화를 하려면 공개키/개인키 쌍(KeyPair)을 생성해야 한다. KenPairGenerator.initialize() 메서드에 키 크기를 지정해야 한다.

```java
// RSAUtils.java
public static KeyPair createKeyPair(int keySize) throws NoSuchAlgorithmException {
	KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
	//keyPairGen.initialize(2048);
	//keyPairGen.initialize(1024);
	keyPairGen.initialize(keySize);
	return keyPairGen.genKeyPair();
}// :
```

생성된 키에서 개인키와 공개키를 구할 수 있다.

```java
KeyPair keyPair = RSAUtils.createKeyPair(1024);  // 키 쌍 생성
PrivateKey privateKey = keyPair.getPrivate(); // 개인키 
PublicKey publicKey = keyPair.getPublic(); // 공개키
```

#### 생성된 키의 바이트 배열 구하기

생성된 키의 byte\[] 배열을 구하려면 Key.getEncoded() 메서드를 사용한다.

```java
byte[] privBytes = keyPair.getPrivate().getEncoded(); // 개인키 바이트로 변환 
byte[] pubBytes  = keyPair.getPublic().getEncoded(); // 공개키 바이트로 변환 
```

#### 키를 BASE64로 변환

키를 파일로 저장하거나 메모리에서 다룰려면 Base64로 인코딩하는 것이 편리하다. Base64는 java.util.Base64를 사용한다.

```java
String privEncodedStr = new String(java.util.Base64.getEncoder().encode(privBytes)); // Base64로 변환 
String publicEncodedStr = new String(java.util.Base64.getEncoder().encode(pubBytes)); // Base64로 변환 
```

RSAUtils 클래스의 privateKeyToBase64() 메서드를 이용하면 편리하다.

```java
privEncodedStr = RSAUtils.privateKeyToBase64(keyPair); // 개인키 바이트로 변환
System.out.println(privEncodedStr);
publicEncodedStr = RSAUtils.publicKeyToBase64(keyPair); // 공개키 바이트로 변환
System.out.println(publicEncodedStr);
```

### 암호화/복호화

앞에서 키를 생성하는 방법을 알았으니 이제 암호화 복호화를 하는 방법을 알아보자.

#### 공개키로 암호화

암호화를 할 때는 공개키로 암호화하고 복호화를 할 때는 개인키로 복호화한다. cipher.doFinal()을 이용하여 암호화를 진행한다.

```java
public static byte[] encrypt(byte[] bytesToEncrypt, PublicKey publicKey) throws InvalidKeyException,
		IllegalBlockSizeException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
	Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
	cipher.init(Cipher.ENCRYPT_MODE, publicKey);
	// Perform the actual encryption on those bytes
	byte[] cipherText = cipher.doFinal(bytesToEncrypt);
	return cipherText;
}// :
```

RSAUtils 클래스에 정의된 encrypt() 메서드는 암호화할 문자열과 공개키를 파라미터로 넘겨야 한다.

```java
String strToEncrypt = "암호화할 문자열입니다.";
KeyPair keyPair = RSAUtils.createKeyPair(1024);  // 키 쌍 생성
byte[] encryptedByte = RSAUtils.encrypt(strToEncrypt.getBytes(), keyPair.getPublic()); // 공개키로 암호화 
```

암호화한 문자열은 byte\[] 배열로 반환된다. Base64로 인코딩하여 사용한다.

```java
String encodedStr = new String(Base64.getEncoder().encode(encryptedByte)); // base64변환 
```

#### 개인키로 복호화

암호화된 문자열은 개인키로 복호화할 수 있다. 암호화된 문자열은 Base64로 인코딩하여 사용하기 때문에 다시 디코딩해서 원래의 byte\[]로 변환해야 한다. 복호화된 문자열은 byte\[]로 반환되기 때문에 문자열로 변환해야 한다.

```java
// 복호화
byte[] decodedByte = Base64.getDecoder().decode(encodedStr);  //Base64 decoding
String decryptedStr = new String(RSAUtils.decrypt(decodedByte, Pair.getPrivate())); // 복호화 
System.out.println("복호화된 문자열:" + decryptedStr);
```

### RSAUtils를 사용한 암호화/복호화

앞에서 RSA 암호화/복호화를 하는 방법을 설명했지만 생각보다 코딩이 길어 질 수 있다. 코드를 간단히 작성하기 위해서 RSAUtils.java에는 여러가지 매서드들을 미리 정의해 놓았다. 이 메서드들을 사용하는 방법을 설명한다.

#### Quick Start

암호화를 위해 키를 생성한다.

```java
KeyPair keyPair = RSAUtils.createKeyPair(1024);  // 키 쌍 생성
```

공개키와 비밀키를 Base64로 인코딩하여 보관한다.

```java
// RSAUtils 사용 
String base64PrivateKey = RSAUtils.privateKeyToBase64(keyPair); //개인키를 base64로 
String base64PublicKey  = RSAUtils.publicKeyToBase64(keyPair);  //공개키를 base64로 
```

Base64로 인코딩된 개인키와 공개키는 다음과 같이 개인키와 공개키로 변환할 수 있다.

```java
PrivateKey privKey = RSAUtils.base64ToPrivateKey(base64PrivateKey); // base64로 인코딩된 바이트 배열에서 개인키 구함
PublicKey pubKey = RSAUtils.base64ToPublicKey(base64PublicKey);     // base64로 인코딩된 바이트 배열에서 공개키 구함
```

공개키를 이용하여 암호화한다.

```java
encryptedByte = RSAUtils.encrypt(strToEncrypt.getBytes(), pubKey);  // 암호화
encodedStr = new String(Base64.getEncoder().encode(encryptedByte)); // base64 변환 
System.out.println("암호화된 문자열:" + encodedStr);
```

암호화된 문자열을 개인키를 이용하여 복호화 한다.

```java
decodedByte = Base64.getDecoder().decode(encodedStr);  // Base64 decoding
byte[] decryptedByte = RSAUtils.decrypt(decodedByte, vKey);
System.out.println("복호화된 문자열:" + new String(decryptedByte));
```

암복화를 RSAUtils를 사용하면 간단히 할 수 있다.

```java
// 간단하게 사용하는 방법 
String encryptedStr = RSAUtils.encrypt(strToEncrypt, base64PublicKey);  // 암호화한 문자열을 base64로 
System.out.println("암호화된 문자열:" + encryptedStr);
decryptedStr = RSAUtils.decrypt(encryptedStr, base64PrivateKey);  // base64로 암호화된 문자열을 복호화 
System.out.println("복호화된 문자열:" + decryptedStr);
```

아래는 RSAUtils.java의 전체 코드이다.

```java
package com.sogood.core.crypt;

import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStreamWriter;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;

import org.bouncycastle.util.io.pem.PemObject;
import org.bouncycastle.util.io.pem.PemWriter;

public class RSAUtils {

 
	// Key Section
	/**
	 * 공개키/개인키 KeyPair을 생성합니다.
	 * 
	 * @return
	 * @throws NoSuchAlgorithmException
	 * @throws Exception
	 */
	public static KeyPair createKeyPair(int keySize) throws NoSuchAlgorithmException {
		KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
		//keyPairGen.initialize(2048);
		keyPairGen.initialize(keySize);
		return keyPairGen.genKeyPair();
	}// :

	/**
	 * 개인키의 byte[]로 개인키를 생성합니다.
	 * 
	 * @param keyBytes 개인키의 바이트배열
	 * @return 개인키
	 * @throws InvalidKeySpecException
	 * @throws NoSuchAlgorithmException
	 * @throws Exception                the Exception
	 */
	public static PrivateKey getPrivateKey(byte[] keyBytes) throws InvalidKeySpecException, NoSuchAlgorithmException {
		PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(keyBytes);
		KeyFactory keyFactory = KeyFactory.getInstance("RSA");
		PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
		return privateKey;
	}// :

	/**
	 * 공개키의 바이트 배열로부터 공개키를 생성한다.
	 * 
	 * @param keyBytes 공개키의 바이트 배열
	 * @return
	 * @throws NoSuchAlgorithmException
	 * @throws InvalidKeySpecException
	 * @throws Exception
	 */
	public static PublicKey getPublicKey(byte[] keyBytes) throws NoSuchAlgorithmException, InvalidKeySpecException {
		X509EncodedKeySpec keySpec = new X509EncodedKeySpec(keyBytes);
		KeyFactory keyFactory = KeyFactory.getInstance("RSA");
		PublicKey publicKey = keyFactory.generatePublic(keySpec);
		return publicKey;
	}//

	/**
	 * 개인키를 Base64로 encoding하여 반환한다.
	 * 
	 * @param keyPair KeyPair 인스턴스
	 * @return Base64로 인코딩된 문자열
	 */
	public static String privateKeyToBase64(KeyPair keyPair) {
		byte[] privBytes = keyPair.getPrivate().getEncoded();
		return new String(java.util.Base64.getEncoder().encode(privBytes));
	}// :

	/**
	 * 공개키를 Base64로 encoding하여 반환한다.
	 * 
	 * @param keyPair KeyPair 인스턴스
	 * @return Base64로 인코딩된 문자열
	 */
	public static String publicKeyToBase64(KeyPair keyPair) {
		byte[] privBytes = keyPair.getPublic().getEncoded();
		return new String(java.util.Base64.getEncoder().encode(privBytes));
	}// :

	/**
	 * Base64로 인코딩된 문자열을 PrivateKey로 변환한다.
	 * 
	 * @param encodedPrivateKey
	 * @return PrivateKey 인스턴스
	 * @throws InvalidKeySpecException
	 * @throws NoSuchAlgorithmException
	 */
	public static PrivateKey base64ToPrivateKey(String encodedPrivateKey)
			throws InvalidKeySpecException, NoSuchAlgorithmException {
		byte[] decodedByte = Base64.getDecoder().decode(encodedPrivateKey.getBytes());
		return getPrivateKey(decodedByte);
	}

	/**
	 * Base64로 인코딩된 문자열을 PublicKey로 변환한다.
	 * 
	 * @param encodedPublicKey 인코딩된 문자열
	 * @return PublicKey 인스턴스
	 * @throws NoSuchAlgorithmException
	 * @throws InvalidKeySpecException
	 */
	public static PublicKey base64ToPublicKey(String encodedPublicKey)
			throws NoSuchAlgorithmException, InvalidKeySpecException {
		byte[] decodedByte = Base64.getDecoder().decode(encodedPublicKey.getBytes());
		return getPublicKey(decodedByte);
	}

	// encrypt/decryt section

	/**
	 * 공개키로 바이트를 암호화 한다.
	 * 
	 * @param bytesToEncrypt 암호화할 바이트 배열
	 * @param publicKey      공개키
	 * @return 암호화된 바이트 배열
	 * @throws InvalidKeyException
	 * @throws BadPaddingException
	 * @throws IllegalBlockSizeException
	 * @throws NoSuchPaddingException
	 * @throws NoSuchAlgorithmException
	 * @throws Exception
	 */
	public static byte[] encrypt(byte[] bytesToEncrypt, PublicKey publicKey) throws InvalidKeyException,
			IllegalBlockSizeException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
		Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
		cipher.init(Cipher.ENCRYPT_MODE, publicKey);
		// Perform the actual encryption on those bytes
		byte[] cipherText = cipher.doFinal(bytesToEncrypt);
		return cipherText;
	}// :

	/**
	 * 개인키로 암호화된 바이트 배열을 복호화 한다.
	 * 
	 * @param bytesToDecrypt 복호화 할 바이트 배열
	 * @param privateKey     개인키
	 * @return 복호화된 바이트 배열
	 * @throws NoSuchPaddingException
	 * @throws NoSuchAlgorithmException
	 * @throws InvalidKeyException
	 * @throws BadPaddingException
	 * @throws IllegalBlockSizeException
	 * @throws Exception
	 */
	public static byte[] decrypt(byte[] bytesToDecrypt, PrivateKey privateKey) throws NoSuchAlgorithmException,
			NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException {
		Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
		cipher.init(Cipher.DECRYPT_MODE, privateKey);
		return cipher.doFinal(bytesToDecrypt);
	}// :

	/**
	 * 문자열을 암호화한 후 Base64로 encodging 하여 반환한다.
	 * 
	 * @param strToEncrypt     암호화할 문자열
	 * @param base64PublicKey Base64로 인코딩된 공개키 문자열
	 * @return Base64로 변환된 암호화된 문자열
	 * @throws NoSuchAlgorithmException
	 * @throws InvalidKeySpecException
	 * @throws InvalidKeyException
	 * @throws IllegalBlockSizeException
	 * @throws BadPaddingException
	 * @throws NoSuchPaddingException
	 */
	public static String encrypt(String strToEncrypt, String base64PublicKey)
			throws NoSuchAlgorithmException, InvalidKeySpecException, InvalidKeyException, IllegalBlockSizeException,
			BadPaddingException, NoSuchPaddingException {
		PublicKey pubKey = base64ToPublicKey(base64PublicKey); // Base64로 인코딩된 문자열을 공개키로 변환
		byte[] encryptedByte = encrypt(strToEncrypt.getBytes(), pubKey); // 암호화
		return new String(Base64.getEncoder().encode(encryptedByte)); // Base64 변환
	}// :

	/**
	 * Base64로 인코딩된 암호화된 문자열을 문자열로 변환한다. 
	 * @param strToDecrypt Base64로 인코딩된 암호화된 문자열
	 * @param base64PrivKey Base64로 인코딩된 암호화된 개인키 
	 * @return
	 * 		복호화된 문자열
	 * @throws InvalidKeySpecException
	 * @throws NoSuchAlgorithmException
	 * @throws InvalidKeyException
	 * @throws NoSuchPaddingException
	 * @throws IllegalBlockSizeException
	 * @throws BadPaddingException
	 */
	public static String decrypt(String strToDecrypt, String base64PrivKey)
			throws InvalidKeySpecException, NoSuchAlgorithmException, InvalidKeyException, NoSuchPaddingException,
			IllegalBlockSizeException, BadPaddingException {
		PrivateKey privKey = base64ToPrivateKey(base64PrivKey); // Base64로 인코딩된 문자열을 개인키로 변환
		byte[] encryptedByte = Base64.getDecoder().decode(strToDecrypt); // Base64 decoding
		byte[] decryptedByte = decrypt(encryptedByte, privKey); // 복호화 
		return new String(decryptedByte);
	}// :

	// utility

	/**
	 * 공개키/개인키를 파일로 저장. 공개키를 디스크에 저장 String publicKeyFile = "/usr/home/key";
	 * RSAUtils.saveKeyAsFile(keyPair.getPublic().getEncoded(), publicKeyFile);
	 * 
	 * 개인키를 디스크에 저장 String privateKeyFile = "/usr/home/key";
	 * RSAUtils.saveKeyAsFile(keyPair.getPrivate().getEncoded(), privateKeyFile);
	 *
	 * @param keyBytes 키의 바이트배열
	 * @param filePath 저장할 파일명(풀패스)
	 * @throws IOException
	 * @throws Exception   the Exception
	 */
	public static void saveKeyAsFile(byte[] keyBytes, String filePath) throws IOException {
		// 파일 시스템에 암호화된 공개키를 쓴다.
		FileOutputStream fos = new FileOutputStream(filePath);
		fos.write(keyBytes);
		fos.close();
	}// :

	/**
	 * 공개키를 파일로 저장
	 * 
	 * @param keyPair
	 * @param filePath
	 * @throws IOException
	 */
	public static void savePublicKeyAsFile(KeyPair keyPair, String filePath) throws IOException {
		saveKeyAsFile(keyPair.getPublic().getEncoded(), filePath);
	}// :

	/**
	 * 개인키를 파일로 저장
	 * 
	 * @param keyPair
	 * @param filePath
	 * @throws IOException
	 */
	public static void savePrivateKeyAsFile(KeyPair keyPair, String filePath) throws IOException {
		saveKeyAsFile(keyPair.getPrivate().getEncoded(), filePath);
	}// :

	/**
	 * 공개키/개인키를 파일로 부터 읽어들인다.
	 * 
	 * @param filePath 저장된 파일명(풀패스)
	 * @return 키의 바이트배열
	 * @throws Exception the Exception
	 */
	public static byte[] getKeyFromFile(String filePath) throws Exception {

		FileInputStream fis = new FileInputStream(filePath);
		ByteArrayOutputStream baos = new ByteArrayOutputStream();
		int theByte = 0;
		while ((theByte = fis.read()) != -1) {
			baos.write(theByte);
		}
		fis.close();
		byte[] keyBytes = baos.toByteArray();
		baos.close();
		return keyBytes;
	}// :
 

	/**
	 * 공개키 또는 개인키를 PEM 파일형식으로 파일로 저장한다. 
	 * @param key    개인키 또는 공개키 
	 * @param description  키 구분자. "RSA PRIVATE KEY" 또는 "RSA PUBLIC KEY"
	 * @param filename 저장할 파일명 
	 * @throws Exception
	 */
  public static void writePemFile(Key key, String description, String filename)   {
    PemObject pemObject = new PemObject (description, key.getEncoded());
    PemWriter pemWriter = null;
    try {
      pemWriter = new PemWriter(new OutputStreamWriter(new FileOutputStream(filename)));  
      pemWriter.writeObject(pemObject);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }finally { 
			try {
				pemWriter.close();	
			} catch (Exception e) {	}
    }
  }//:

	/**
	 * 공개키 또는 개인키를 PEM 형식으로 변환하여 반환한다.  
	 * @param key 공개키 또는 개인키
	 * @param description 키 구분자. "RSA PRIVATE KEY" 또는 "RSA PUBLIC KEY"
	 * @return
	 * 	PEM 형식 문자열
	 */
  public static String writePemToString(Key key, String description)   {
    PemObject pemObject = new PemObject (description, key.getEncoded());
    PemWriter pemWriter = null;
    ByteArrayOutputStream bout = new ByteArrayOutputStream();
    try {
      pemWriter = new PemWriter(new OutputStreamWriter( bout ));  
      pemWriter.writeObject(pemObject);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }finally { 
			try {
				pemWriter.close();	
			} catch (Exception e) {	}
    }
    return bout.toString();
  }//:
}///~
```

## JavaScript와 Java간의 RSA 암복호화

RSA 암호화를 사용할 때 클라이언트인 JavaScript와 서버인 Javar간에 암호화 파일 형식을 맞추어야 한다. JSEncrypt 라이브러리를 사용할려고 하는데 JSEncrypt는 PEM 형식을 사용한다.

"PEM"(원래 "Privacy Enhanced Mail"의 약어)은 Apache 및 기타 웹 서버 플랫폼에서 사용되는 디지털 인증서 및 키에 대한 매우 일반적인 컨테이너 형식이다. X.509 인증서의 PEM 파일은 인증서 텍스트의 Base64 인코딩과 인증서의 시작과 끝을 표시하는 일반 텍스트 머리글 및 바닥 글을 포함하는 텍스트 파일이다.

아래와 같은 형식이다.

```
-----BEGIN CERTIFICATE-----
MIIGbzCCBFegAwIBAgIICZftEJ0fB/wwDQYJKoZIhvcNAQELBQAwfDELMAkGA1UE
BhMCVVMxDjAMBgNVBAgMBVRleGFzMRAwDgYDVQQHDAdIb3VzdG9uMRgwFgYDVQQK
DA9TU0wgQ29ycG9yYXRpb24xMTAvBgNVBAMMKFNTTC5jb20gUm9vdCBDZXJ0aWZp
...
Nztr2Isaaz4LpMEo4mGCiGxec5mKr1w8AE9n6D91CvxR5/zL1VU1JCVC7sAtkdki
vnN1/6jEKFJvlUr5/FX04JXeomIjXTI8ciruZ6HIkbtJup1n9Zxvmr9JQcFTsP2c
bRbjaT7JD6MBidAWRCJWClR/5etTZwWwWrRCrzvIHC7WO6rCzwu69a+l7ofCKlWs
y702dmPTKEdEfwhgLx0LxJr/Aw==
-----END CERTIFICATE-----
```

그런데 Java에서는 이러한 형식을 직접 읽거나 생성할 수 없다. 읽는 방법에 대해서는 [Baeldung](https://www.baeldung.com/java-read-pem-file-keys)에 잘 설명되어 있다.

다행이도 Java에서 생성한 암호화 키를 PEM 형식으로 변환하는 것은 어렵지 않다. 위에서 예시한 BEGIN과 END가 있는 라인만 제외하면 키는 동일하다.

### 의존성 추가

bouncycastle 라이브러리를 용하면 쉽게 PEM 파일을 생성할 수 있다. 먼저 의존성을 pom.xml 파일에 추가한다.

```xml
<dependency>
	<groupId>org.bouncycastle</groupId>
	<artifactId>bcprov-jdk15on</artifactId>
	<version>1.67</version>
</dependency>
```

### 키 쌍 생성

키 쌍의 생성은 RSAUtils를 사용한다. JSEncrypt 라이브러리는 1024 byte의 키를 사용한다. 키 길이를 맞추어야 한다.

```java
KeyPair keyPair = RSAUtils.createKeyPair(1024);
```

### 키 저장

RSAUtils 클래스에는 키를 저장할 수 있는 writePemFile() 메서드가 준비되어 있다. 이 메서드를 이용하여 공개키와 개인키를 저장한다.

```java
// RSAUtils.java
public static void writePemFile(Key key, String description, String filename)   {
  PemObject pemObject = new PemObject (description, key.getEncoded());
  PemWriter pemWriter = null;
  try {
    pemWriter = new PemWriter(new OutputStreamWriter(new FileOutputStream(filename)));  
    pemWriter.writeObject(pemObject);
  } catch (Exception e) {
    throw new RuntimeException(e);
  }finally { 
	try {
		pemWriter.close();	
	} catch (Exception e) {	}
  }
}//:
```

개인 키를 저장할 때에는 키설명을 위한 description을 아래 코드와 같이 작성한다. 사실 다른 문구를 써도 관계없다.

```java
RSAUtils.writePemFile(keyPair.getPrivate(), "RSA PRIVATE KEY", path + "id_rsa" );
```

공개키를 저장하는 코드는 다음과 같다.

```java
RSAUtils.writePemFile(keyPair.getPrivate(), "RSA PUBLIC KEY", path + "id_rsa.pub" );
```

### 클라이언트에서 RSA 암호화

브라우저에서 RSA 암호화를 사용하기 위해 JSEncrypt를 사용할 것이다. 라이브러리를 다운로드 받아 웹 폴더의 적당한위치에 저장한다. [https://github.com/travist/jsencrypt](https://github.com/travist/jsencrypt)로 로 가면 라이브러리의 사용방법을 볼 수 있고 다운로드 받을 수 있다.

### JSEncrypt 라이브러리 로드

HTML 페이지에서 자바르크립트를 로드한다.

```html
<script src="/js/crypt/jsencrypt.min.js"></script>
```

### 암호화

암호화를 하려면 서버에서 생성한 공개키가 있어야 한다. 다음과 같이 공개키를 저장한다.

```javascript
let pubKey = fetchPubKey(); // 서버에서 가져온 공개키 
```

공개키로 JSEncrypt 객체를 생성한다.

```javascript
var encrypt = new JSEncrypt();
encrypt.setPublicKey(pubKey);
```

enrypt() 함수를 사용하여 문자열을 암호화한다.

```javascript
var strToEncrypt = "암호화할 문자열";
var encrypted = encrypt.encrypt(strToEncrypt);
```

### 복호화

복호화를 하려면 개인키가 있어야 한다. 개인키를 네트워크 상에서 주고 받는 것은 매우 위험하므로 이렇게 개발하지 않는다. 이 글에서는 개인키로 복호화하는 방법을 설명하기 위해서 개인키를 가져오는 서버로부터 가져오는 것으로 가정하였다.

```java
var privKey = fetchPrivKey();
```

캐인키로 복호화 하려면 decrypt() 함수를 사용한다. 복호화 하기 전에 개인키를 저장해야 한다.

```javascript
var decrypt = new JSEncrypt();
decrypt.setPrivateKey(privKey);
var uncrypted = decrypt.decrypt(encrypted);
```
