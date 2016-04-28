---
layout: postlayout
title: 构建安全的http数据传输
description: 数据传输中保证数据安全，使用加密算法对数据进行加密，尤其是互联网环境下对外暴露接口，比如restfull接口，数据的安全性更是重中之重
thumbimg: 1346208288725.jpg
categories: [encrpted]
tags: [encrpted,restfull]
---

## 实现方式

- 传统的办法：使用自签名证书，借用jdk和web容器的ssl层实现，这种方法比较常用，也比较省事。
- 手动编程的方法：类似于自己写了一层ssl的实现。原理也很简单，对方把数据加密后传给服务端，这边解密后该怎么处理就怎么处理，完事以后把响应的数据加密传给客户端，客户端解密之后该怎么处理就怎么处理。
　　现只介绍第二种，因为加密方案是可以随意更改的（比如把其中的某个算法用别的算法替换）。还有一点原因是，自己写的东西更加容易掌控，如果加密层出现问题，最后一点原因是，基于算法而不是基于Java类库，更容易制作各种语言的客户端。<br>

## 算法设计

以上基本上已经完成了整个加密解密功能的设计，接下来的工作就是将工作落实到实处，到底加密算法如何选择？
经过一番百度和google，发现算法主要分为以下三种：

>1、不可逆加密算法，比如md5就是这样一种，这种算法一般用于校验，比如校验用户的密码对不对。<br>
>2、对称加密算法，这种算法是可逆的，两边拥有同一个密钥，使用这个密钥可以对数据加密和解密，一般用于数据>>加密传输。特点是速度快，但安全性相对于非对称加密较低。<br>
>3、非对称加密算法，这种算法依然是可逆的，两边拥有不同的密钥，一个叫公钥，一个叫私钥，两边也都可以对数据加密和解密，一般用于数字签名。特点是速度较慢，但安全性相对于对称加密更高。

之前听说过ssl的实现是几种算法混合使用的，这给了LZ很大的启示。既然每种算法都有它的优势，我们为何不混合使用呢。
于是，想来想去，决定使用md5（不可逆加密）+des（对称加密）＋rsa（非对称加密）的加密方式，编码格式可以使用Base64。来看看需求，主要有两点。

- 1.客户端需要授权，也就是说发布的服务不是谁想调就能调的。<br>
- 2.数据在传输过程中是加密的，并且安全性要等同于非对称加密算法的安全性，但性能要等同于对称加密的速度。<br>
我们来看看以上的算法实现能否满足需要，过程是这样的：
  
>1、假设给客户端一个授权码，比如123456。再假设客户端现在需要传的数据是{"name":"xiaolongzuo"}。（请求数据和响应数据都是json格式）<br>
>2、客户端需要先对123456进行md5加密，然后放入到传输数据中。也就是传输的数据会变成{"name":"xiaolongzuo","verifyCode":"md5(123456)"}<br>
>3、客户端生成des的随机密钥（注意，对称密钥每次都是随机生成的），假设是abcdef，客户端使用该密钥对传输数据进行des加密，并且对随机密钥进行rsa加密，最终拼成一个json。也就是最终传输的数据会变成{"privateKey":"rsa(abcdef)","data":"des({"name":"xiaolongzuo","verifyCode":"md5(123456)"})"}<br>
>4、服务端使用相反的过程对数据进行解密即可，并验证解密后的授权码md5(123456)是否存在，如果不存在，则认为该客户端未被授权。当服务端返回数据时，依旧使用abcdef对数据进行des加密即可。
　　
	安全性分析：假设以上的数据被黑客拦截，那么黑客最主要做的就是破解rsa算法的私钥（私钥只有LZ有，客户端组件中会附带公钥），这个问题听说是比较难的，具体为什么，这就不是LZ需要考虑的了，LZ还没这个能力。基于这个前提，LZ可以认为传输的数据还是比较安全的。
	性能分析：由于我们的rsa只对长度比较短的des私钥进行加密，因此非对称加密速度慢的特点并不会影响我们太多。几乎上所有的传输数据，我们都是使用的des进行加密，因此在速度上，几乎等同于对称加密的速度。

## 算法示例

>此实例在于使用约定秘钥使用对称加密，对加密后的数据再使用MD5加密。

将数据使用秘钥对称加密：

```java?linenums
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SecurityUtil {
	public static byte[] doEncrypt(String original, String key)	throws Exception {
		try {
			byte[] byteKey = AES.asBin(key);
			if (original != null && original.trim().length() > 0) {
				byte[] byteEncrypted = AES.doEncrypt(original.getBytes("UTF-8"), byteKey);
				return byteEncrypted;
			}
		} catch (Exception e) {
			log.error(e.getMessage(), e);
			throw new Exception("加密异常");
		}
		return null;
	}
}
```
二进制转为十六进制：

```java?linenums
public static String asHex(byte buf[]) {
	if (buf == null) {
		return "";
	}
	// buf.length * 2 字节数组转化为16进制需乘以2
	StringBuffer strbuf = new StringBuffer(buf.length * 2);
	int i;
	for (i = 0; i < buf.length; i++) {
		if (((int) buf[i] & 0xff) < 0x10)
			strbuf.append("0");// 若buf[i] & 0xff在10以内加0
		strbuf.append(Long.toString((int) buf[i] & 0xff, 16));
	}
	return strbuf.toString();
}
```
对加密数据转化为十六进制：

```java?linenums
byte[] aesJson = SecurityUtil.doEncrypt(outJson, secretKey);
String mdJson = asHex(aesJson);
```