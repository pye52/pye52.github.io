---
title: SSL证书版本问题
date: 2017-01-21 16:37:45
tags:
- Android
- SSL
---

当使用SSL和服务器建立远程连接时，若采用的证书版本不对，则可能会报错

报错内容如下

```java
java.io.IOException: Wrong version of key store.
com.android.org.bouncycastle.jce.provider.JDKKeyStore.engineLoad(JDKKeyStore.java:812)
...
```

<!-- more -->

解决办法就不啰嗦了，直接贴stackoverflow的回答

**No need to do every thing again !!!**

You need to change the type of the keystore, from BKS to BKS-v1 (BKS-v1 is an older version of BKS). Because the BKS version changed as said [here](http://android-developers.blogspot.fr/2013/02/using-cryptography-to-store-credentials.html)

There is another solution, that is much much easier:

1. **Using Portecle:**
   - Downloads Portecle [http://portecle.sourceforge.net/](http://portecle.sourceforge.net/)
   - Open your bks file with the password and portecle
   - Do Tools>>Change Keystore Type>>**BKS-v1**
   - Save the file
2. **You may use KeyStore Explorer**

The new file will be encoded with BKS-v1 and will not show anymore the error....

**Note:** Android works with differents BKS version: for instance, API 15 will require BKS-1 contrary to API 23 which require BKS, so you may need to put both files in your app.

**Edit:** Replaced Build.VERSION_CODES.JELLY_BEAN by Build.VERSION_CODES.JELLY_BEAN_MR1, thanks to L3K0V

**Note 2:** You can use this code:

```java
int bks_version;
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
    bks_version = R.raw.publickey; //The BKS file
} else {
    bks_version = R.raw.publickey_v1; //The BKS (v-1) file
}
KeyStore ks = KeyStore.getInstance("BKS");
InputStream in = getResources().openRawResource(bks_version);  
ks.load(in, "mypass".toCharArray());
```



注意上面代码，在SDK 17之前的Android都需要采用BKS-v1的证书，因此只需要下载网址里的工具进行转换则可。[工具下载地址](https://sourceforge.net/projects/portecle/)