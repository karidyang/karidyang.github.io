---
layout: post
title: "develop weixin web server"
date: 2014-06-03 09:40:49 +0800
comments: true
categories: java, weixin

---

最近做了一个微信的商户平台，有些心得，和大家分享一下。

### 一、申请帐号

首先肯定需要注册一个公众平台帐号了，现在注册就能免费成为订阅号，能使用一些基本的接口功能，如：接收消息，发送消息等。而高级的功能，必须是认证的商户，并且缴纳300元/年的认证费（有点坑啊，一年交一次）。具体的接口文档，可以去查看<a href="https://open.weixin.qq.com/cgi-bin/frame?t=resource/res_main_tmpl&lang=zh_CN">微信公众平台开发者文档</a>。

### 二、接入

###### 1、申请消息接口

在公众平台网站的高级功能 – 开发模式页，点击“成为开发者”按钮，填写URL和Token，其中URL是开发者用来接收微信服务器数据的接口URL。Token可由开发者任意填写，用作生成签名（该Token会和接口URL中包含的Token进行比对，从而验证安全性）。 

###### 2、验证URL有效性

微信服务器会对你刚才填的URL做GET请求，并带上下面4个参数
<table>
<thead>
<tr>
<td>参数</td>
<td>描述</td>
</tr>
</thead>
<tbody>
<tr>
<td>signature</td>
<td>微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。</td>
</tr>
<tr>
<td>timestamp</td>
<td>时间戳</td>
</tr>
<tr>
<td>nonce</td>
<td>随机数</td>
</tr>
<tr>
<td>echostr</td>
<td>随机字符串</td>
</tr>
</tbody>
</table>

开发者通过检验signature对请求进行校验（下面有校验方式）。若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，成为开发者成功，否则接入失败。

```
加密/校验流程如下：
1. 将token、timestamp、nonce三个参数进行字典序排序
2. 将三个参数字符串拼接成一个字符串进行sha1加密
3. 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信
```

检验signature的Java示例代码:

```

/**
 * 签名校验工具类
 * Created by karidyang on 14-2-23.
 */
public class SignUtils {
    public static final String TOKEN = "";

    /**
     * 验证签名
     * @param signature 签名
     * @param timestamp 时间戳
     * @param nonce 随机码
     * @return 校验是否成功
     */
    public static boolean checkSign(String signature, String timestamp, String nonce) {
        if (signature == null) {
            return false;
        }
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("SHA-1");

            String[] arr = new String[] { TOKEN, timestamp, nonce };
            Arrays.sort(arr);
            String str = StringUtils.join(arr,"");
            byte[] digest = messageDigest.digest(str.getBytes());
            String tmpStr = byteToStr(digest);
            return tmpStr != null && tmpStr.equals(signature.toUpperCase());

        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }

        return false;
    }

    /**
     * 将字节数组转换为十六进制字符串
     *
     * @param byteArray
     * @return
     */
    private static String byteToStr(byte[] byteArray) {
        String strDigest = "";
        for (int i = 0; i < byteArray.length; i++) {
            strDigest += byteToHexStr(byteArray[i]);
        }
        return strDigest;
    }

    /**
     * 将字节转换为十六进制字符串
     *
     * @param mByte
     * @return
     */
    private static String byteToHexStr(byte mByte) {
        char[] Digit = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F' };
        char[] tempArr = new char[2];
        tempArr[0] = Digit[(mByte >>> 4) & 0X0F];
        tempArr[1] = Digit[mByte & 0X0F];

        String s = new String(tempArr);
        return s;
    }
}
```

###### 3、成为开发者

验证URL有效性成功后即接入生效，成为开发者。如果公众号类型为服务号（订阅号只能使用普通消息接口），可以在公众平台网站中申请认证，认证成功的服务号将获得众多接口权限，以满足开发者需求。

此后用户每次向公众号发送消息、或者产生自定义菜单点击事件时，响应URL将得到推送。

公众号调用各接口时，一般会获得正确的结果，具体结果可见对应接口的说明。返回错误时，可根据返回码来查询错误原因。全局返回码说明

用户向公众号发送消息时，公众号方收到的消息发送者是一个OpenID，是使用用户微信号加密后的结果，每个用户对每个公众号有一个唯一的OpenID。

此外请注意，微信公众号接口只支持80接口。



### 三、设置菜单

未完待续。