**Java加密解密**

[TOC]

# MD5

## 实现

```Java
  /**
   * MD5加密
   * @param data
   * @return
   * @throws Exception
   */
  public static byte[] encryptMD5(byte[] data) throws Exception {
    MessageDigest md5 = MessageDigest.getInstance("MD5");
    md5.update(data);
    return md5.digest();
  }
```

# SHA

## 实现

```Java
/**
*
* @param data to be encrypted
* @param shaN encrypt method,SHA-1,SHA-224,SHA-256,SHA-384,SHA-512
* @return 已加密的数据
* @throws Exception
*/
public static byte[] encryptSHA(byte[] data, String shaN) throws Exception {
   MessageDigest sha = MessageDigest.getInstance(shaN);
   sha.update(data);
   return sha.digest();

}
```

# Base64

## 实现

```Java
public class Base64 {
    /**
     * BASE64解密
     *
     * @param key the String to be decrypted
     * @return byte[] the data which is decrypted
     * @throws Exception
     */
    public static byte[] decryptBASE64(String key) throws Exception {
        return Base64.decode(key,Base64.DEFAULT);
    }
    /**
     * BASE64加密
     *
     * @param key the String to be encrypted
     * @return String the data which is encrypted
     * @throws Exception
     */
    public static String encryptBASE64(byte[] key) throws Exception {
        return Base64.encodeToString(key, Base64.DEFAULT);
    }
}
```

# AES


## 实现

```Java
public class AESUtils {

  /**
   * AES加解密
   *
   * @author liuguoquan
   */

  //AES加密类型
  public static final String ALGORITHM = "AES/CBC/NoPadding";
  //加密向量
  public static String ENCRYPT_VECTOR = "0102030405123456";
  //加密密钥
  public static String PRIVATE_KEY = "5b26b58980d35723";

  /**
   * AES加密
   *
   * @param data 要加密的数据
   * @throws Exception
   */
  public static byte[] encrypt(byte[] data) throws Exception {

    byte[] encrypted;

    Cipher cipher = Cipher.getInstance("AES/CBC/NoPadding");
    int blockSize = cipher.getBlockSize();

    // 填充数据
    int plaintextLength = data.length;
    if (plaintextLength % blockSize != 0) {
      plaintextLength = plaintextLength + (blockSize - (plaintextLength % blockSize));
    }
    byte[] plaintext = new byte[plaintextLength];
    System.arraycopy(data, 0, plaintext, 0, data.length);
    SecretKeySpec keyspec = new SecretKeySpec(PRIVATE_KEY.getBytes(), "AES");
    IvParameterSpec ivspec = new IvParameterSpec(ENCRYPT_VECTOR.getBytes());
    cipher.init(Cipher.ENCRYPT_MODE, keyspec, ivspec);
    encrypted = cipher.doFinal(plaintext);
    return encrypted;
  }

  /**
   * @param data 要解密的数据
   * @throws Exception
   */
  public static byte[] decrypt(byte[] data) throws Exception {
    byte[] originalData = null;
    Cipher cipher = Cipher.getInstance("AES/CBC/NoPadding");
    SecretKeySpec keyspec = new SecretKeySpec(PRIVATE_KEY.getBytes(), "AES");
    IvParameterSpec ivspec = new IvParameterSpec(ENCRYPT_VECTOR.getBytes());
    cipher.init(Cipher.DECRYPT_MODE, keyspec, ivspec);
    originalData = cipher.doFinal(data);
    return originalData;
  }
}

/**
* 初始化密钥
* @param seed
* @return
* @throws Exception
*/
public static String initKey(String seed) throws Exception {
   SecureRandom secureRandom = null;

   if (seed != null) {
       secureRandom = new SecureRandom(decryptBASE64(seed));
   } else {
       secureRandom = new SecureRandom();
   }

   KeyGenerator kg = KeyGenerator.getInstance(ALGORITHM);
   kg.init(secureRandom);

   SecretKey secretKey = kg.generateKey();

   return encryptBASE64(secretKey.getEncoded());
}
```

# DES

## 实现

```Java
/**
* DES加密
* @param data
* @param key
* @return
* @throws Exception
*/
public static byte[] encrypt(byte[] data, byte[] key) throws Exception {
   DESKeySpec dks = new DESKeySpec(key);
   SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
   SecretKey secretKey = keyFactory.generateSecret(dks);
   Cipher cipher = Cipher.getInstance("DES");
   cipher.init(Cipher.ENCRYPT_MODE, secretKey);
   return cipher.doFinal(data);
}

/**
* DES解密
* @param data
* @param key
* @return
* @throws Exception
*/
public static byte[] decrypt(byte[] data, byte[] key) throws Exception {
   DESKeySpec dks = new DESKeySpec(key);
   SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
   SecretKey secretKey = keyFactory.generateSecret(dks);
   Cipher cipher = Cipher.getInstance("DES");
   cipher.init(Cipher.DECRYPT_MODE, secretKey);
   return cipher.doFinal(data);
}

/**
* 初始化密钥
*/
public static String initKey(String seed) throws Exception {
   SecureRandom secureRandom = null;
   if (seed != null) {
       secureRandom = new SecureRandom(decryptBASE64(seed));
   } else {
       secureRandom = new SecureRandom();
   }
   KeyGenerator kg = KeyGenerator.getInstance("DES");
   kg.init(secureRandom);
   SecretKey secretKey = kg.generateKey();
   return encryptBASE64(secretKey.getEncoded());
}
```

# RSA

## 实现

### 初始化密钥

```Java
  /**
   * 初始化密钥
   *
   * @throws Exception
   */
  public static Map<String, Object> initKey() throws Exception {
    KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
    keyPairGen.initialize(1024);

    KeyPair keyPair = keyPairGen.generateKeyPair();

    // 公钥
    RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();

    // 私钥
    RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();

    Map<String, Object> keyMap = new HashMap<String, Object>(2);

    keyMap.put(PUBLIC_KEY, publicKey);
    keyMap.put(PRIVATE_KEY, privateKey);
    return keyMap;
  }
```

### 加密解密

```Java
  /**
   * 公钥加密
   * Encrypt the data with the public key.
   *
   * @param data data to be encrypted.
   * @param publicKeyBytes public key in binary format.
   * @return encrypted data.
   */
  public static byte[] encryptWithPublicKey(byte[] data, byte[] publicKeyBytes) throws Exception {

    byte[] encryptedData = null;

    //Create a new X509EncodedKeySpec with the given encoded key.
    X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(publicKeyBytes);

    //RSA key factory.
    KeyFactory keyFactory = KeyFactory.getInstance("RSA");

    //Get the public key from the provided key specification.
    PublicKey publicKey = keyFactory.generatePublic(x509EncodedKeySpec);

    //Init the ciper.
    //Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
    Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());

    //Encrypt mode!
    cipher.init(Cipher.ENCRYPT_MODE, publicKey);

    //Do the encryption now !!!!
    encryptedData = cipher.doFinal(data);

    return encryptedData;
  }

  /**
   * 私钥解密
   * Decrypt the data with private key bytes.
   *
   * @param data data to be decrypted.
   * @param privateKeyBytes private key in binary format.
   * @return decrypted data.
   */
  public static byte[] decryptWithPrivateKey(byte[] data, byte[] privateKeyBytes) throws Exception {

    byte[] encryptedData;
    // Create a new PKCS8EncodedKeySpec with the given encoded key.
    PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(privateKeyBytes);
    //RSA key factory.
    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    //Get the private key from the provided key specification.
    PrivateKey privateKey = keyFactory.generatePrivate(pkcs8EncodedKeySpec);
    //Init the ciper.
    //Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
    Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
    //Decrypt mode!
    cipher.init(Cipher.DECRYPT_MODE, privateKey);

    //Do the decryption now !!!!
    encryptedData = cipher.doFinal(data);
    return encryptedData;
  }
```

### 数字签名

```Java
  /**
   * 用私钥对信息生成数字签名
   *
   * @param data 加密数据
   * @param privateKey 私钥
   * @throws Exception
   */
  public static String sign(byte[] data, String privateKey) throws Exception {
    // 解密由base64编码的私钥
    byte[] keyBytes = decryptBASE64(privateKey);

    // 构造PKCS8EncodedKeySpec对象
    PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(keyBytes);

    // KEY_ALGORITHM 指定的加密算法
    KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);

    // 取私钥匙对象
    PrivateKey priKey = keyFactory.generatePrivate(pkcs8KeySpec);

    // 用私钥对信息生成数字签名
    Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);
    signature.initSign(priKey);
    signature.update(data);

    return encryptBASE64(signature.sign());
  }
  
    /**
   * 公钥校验数字签名
   * @param data 加密数据
   * @param publicKey 公钥
   * @param sign 数字签名
   * @return 校验成功返回true 失败返回false
   * @throws Exception
   */
  public static boolean verify(byte[] data, String publicKey, String sign) throws Exception {

    // 解密由base64编码的公钥
    byte[] keyBytes = decryptBASE64(publicKey);

    // 构造X509EncodedKeySpec对象
    X509EncodedKeySpec keySpec = new X509EncodedKeySpec(keyBytes);

    // KEY_ALGORITHM 指定的加密算法
    KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);

    // 取公钥匙对象
    PublicKey pubKey = keyFactory.generatePublic(keySpec);

    Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);
    signature.initVerify(pubKey);
    signature.update(data);

    // 验证签名是否正常
    return signature.verify(decryptBASE64(sign));
  }
```

# CRC16

## 实现

```Java
public class CRC16CheckUtil {
    
    private static final int NUMBER7 = 7;
    private static final int NUMBER8 = 8;
    private static final int NUMBER15 = 15;
    
    /**
     * 生成CRC16校验码
     * @param bytes
     * @return
     */
    public static int crc16ToUnsignShort(byte[] bytes) {
        short crc = (short) 0;
        int i = 0;

        while (i < bytes.length) {
            for (int j = 0; j < NUMBER8; j++) {
                boolean c15 = (crc >> NUMBER15 & 1) == 1;
                boolean bit = (bytes[i] >> (NUMBER7 - j) & 1) == 1;

                crc <<= 1;

                if (c15 ^ bit) {
                    crc ^= 0x1021;
                }
            }
            i++;
        }
        return crc & 0xFFFF;
    }

}
```


