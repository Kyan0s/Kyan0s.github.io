### 获取绿宝书音频

#### 0. 废话之缘起

雅思单词书绿宝书提供了 mp3 音频可供学习，但听完一个后想听上一个还得扫码。愤而下载了他们家 app 后，发现依旧得每听每扫，且没有已听音频的历史记录，真的让人好暴躁 ⋋_⋌   

于是就研究了下怎么下载书里的全部音频（在此之前还查了下某宝，可惜是无，只好自己动手了），也就有了这篇记录～ 

<br>

#### 1. 无脑尝试

抓包发现下载链接类似于这种：

```shell
https://pc-media.xdfsjj.com/2b0b5c7efef6e121c9e90e7a4683a9ba/62135128/tm7345927_aa95619311af1ab3f8d98cca670abefe52bb2f7a.mp3
```
随手重放发现能够下载，于是好整以暇地找了全部 48 个下载链接下过来了。当然直接浏览器打开会报 403 forbidden，需要手动设置 header （未 check 服务器能接受的最少 header 改动数量，抓包看见啥就全堆一起了）

```python
headers = [
    "Host: pc-media.xdfsjj.com",
    "Cookie: _yttoken_=yvwo3voz75knhygy; _ytuserid_=112909351",
    'Sec-Ch-Ua: "(Not(A:Brand";v="8", "Chromium";v="98"',
    "Sec-Ch-Ua-Mobile: ?0",
    'Sec-Ch-Ua-Platform: "Windows"',
    "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.82 Safari/537.36",
    "Sec-Fetch-Site: same-site",
    "Sec-Fetch-Mode: no-cors",
    "Sec-Fetch-Dest: audio",
    "Referer: https://dogwood.xdfsjj.com/",
    "Range: bytes=0-"
]
```
链接似乎存活了几个小时。在我想要跟小伙伴炫耀的时候，链接宝宝们华丽丽地过期了。好吧开始围观代码 (ಠ⌣ಠ)

<br>

#### 2. 深入（真的吗？）研究

##### 2.1  确认爬 bug 需要的参数

下载音频之前浏览器通过 getLinkUrl.do 获取 url 信息：

```shell
POST /resourceService/getLinkUrl.do HTTP/2
Host: dogwood.xdfsjj.com
Cookie: _yttoken_=手动打码.jpg; _ytuserid_=手动打码.jpg JSESSIONID=手动打码.jpg
Content-Length: 146
Sec-Ch-Ua: "(Not(A:Brand";v="8", "Chromium";v="98"
Accept: application/json, text/plain, */*
Content-Type: application/x-www-form-urlencoded
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.82 Safari/537.36
Sec-Ch-Ua-Platform: "Windows"
Origin: https://dogwood.xdfsjj.com
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://dogwood.xdfsjj.com/pc/audioDetail.html?id=45208&pcrId=9428294&resId=11190133&resSign=f451f1&type=14
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9

id=45208&type=14&resId=11190133&resSign=f451f1&v=2&_timestamp=1645445534123&_nonce=3edbeb9e-a962-4186-b15e-ac96e69f5c8e&_sign=F2D17C110819760C15F0
```

服务器的返回也很简单粗暴：

```shell
HTTP/2 200 OK
Date: Mon, 21 Feb 2022 12:12:38 GMT
Content-Type: application/json;charset=UTF-8
Content-Length: 392
Access-Control-Allow-Methods: GET,POST,OPTIONS
Access-Control-Allow-Headers: X-Requested-With,Content-Type,Accept,Origin,If-Modified-Since
Access-Control-Max-Age: 3600
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: https://dogwood.xdfsjj.com

{"encrypted":true,"encryptedData":"ubY4o0kjceB/yI1q1EO41+lsrherfc+7ZYtHOnPFdwHdefaai23GLHoMFY8aBNf7JE8JjtMIQrT+\r\noKB85UGwOfpnqrCVkk8nnkpLjQ0j8iwkBidMBIkeE/bzJDDam8f46a5nuhKcGx20zn93nzqh9q6g\r\nBnjyFKxLbG1NmaP5xlmzExj5vaLqlUUwH2r2eqY4c5IPmqyVgewijXZ6UeOcr4AE+/EX6enJBYuY\r\nicl8EvN+5P5Z75XQWdaGylm7EYg1gfCVUdsi7xTHFK4wvU2sWj8u/Nx7akWxpKlFEKlRm3t5zmn7\r\nSoSZU7LM04jQ8Ghp\r\n","success":true}
```

查看代码发现 `encryptedData` 为 `AES` 加密，且 `key` 华丽丽地写死在了代码中:

```javascript
if (n.success && n.encrypted && n.encryptedData) {
    var i = lt.AES.decrypt(n.encryptedData.replace(/\r\n/g, ""), lt.enc.Utf8.parse("Suj4XDDt3jPsH9Jj"), {
        mode: lt.mode.ECB
    })
      , o = JSON.parse(i.toString(lt.enc.Utf8));
    r.data = {
        success: n.success,
        msg: n.msg,
        errorCode: n.errorCode,
        data: o
    }
}
```

搜了一下这个 `key`，找到了一个现成的[解密代码](https://www.52pojie.cn/thread-1424921-1-1.html)，直接照搬得到了上一节中我们提到的 `url`:

```shell
{
    'host': '183.220.22.215', 
    'httpDns': False, 
    'mediaType': 3, 
    'needSignPro': False, 
    'resId': 11190133, 
    'url': 'https://pc-media.xdfsjj.com/0f201351c3631b74a485ac9ef2e145bf/621381C0/tm7345927_848f37af57d21f3e16a194d18463b2907b491860.mp3'
}
```

于是开始寻找上文 `request` 中：

+ 请求头中出现的 `_yttoken_, _ytuserid_, JSESSIONID, id, pcrId, resId, resSign, type`
+ 请求体中出现的 `id, type, resId, resSign, v, _timestamp, _nonce, _sign`

<br>

##### 2.2 来自 server 的参数

因为是无聊的 `copy & search` 就不写详细过程了。用户相关信息由对 `loginService/userInfo.do` 等的响应中指定，音频资源文件相关如的 `resId, resSign, type` 等由对 `bookService/detail.do` 的响应中指定。上述解密后的音频文件相关信息如下：

```json
{
    "bookId":45208,
    "canDown":true,
    "canShare":true,
    "fkId":1239660,
    "id":11190133,
    "idSign":"f451f1",
    "length":11101257,
    "mediaType":3,
    "pcrId":9428294,
    "pcrName":"Word List 48",
    "times":693,
    "title":"Word List 48",
    "type":2
}
```
<br>

##### 2.3 客户端生成的参数

`_timestamp, _nonce, _sign` 三个参数用以防止重放攻击。在代码中找了一下后两个参数的生成逻辑：

```javascript
function Er(e, t) {
    if (e.data || (e.data = {}),
    e.withCredentials = !0,
    "string" == typeof e.data)
        return e;
    if (e.data.v = 2,
    e.data._timestamp = +new Date + "",
    e.data._nonce = "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, (function(e) {
        var t = 16 * Math.random() | 0;
        return ("x" == e ? t : 3 & t | 8).toString(16)
    }
    )),
    !t) {
        var n = function(e) {
            if (!e)
                return null;
            var t = new RegExp("(^| )" + e + "=([^;]*)(;|$)")
               , n = document.cookie.match(t);
            return n ? decodeURIComponent(n[2]) : null
        }("refUserId");
        n && (e.data.refUserId = n);
        var r = window.YTLogger;
        if (r && r.deviceId && r.traceId) {
            var i = r.deviceId()
               , o = r.traceId();
            i && (e.data._deviceid = i),
            o && (e.data._traceId = o)
        }
    }
    return e.data._sign = function(e) {
        var t = e.data
           , n = Object.keys(t).sort().reduce((function(e, n) {
            return e + (null === t[n] || void 0 === t[n] ? "" : t[n]) + n
        }
        ), "");
        return lt.MD5(n).toString().toUpperCase().substring(0, 20)
    }(e),
    e
}
```

<br>


#### 03. 杂碎

代码逻辑存在于两个 `js` 文件中。其中对 `server` 返回信息进行处理的逻辑位于 `index.1647aa023bcaf3996f5a.js` 中，对客户端生成参数的处理逻辑位于 `vendor.b3cabdc0a79b34b906c0.js` 中。两个代码文件均经过混淆（应该不是程序员手动写成这样的吧……）；比起上文生成 `nonce` 等的代码，`index.1647aa023bcaf3996f5a.js` 中的这部分代码看上去混淆得更为明显：

![](../images/20220221-obfuscate.png)

然而，为什么要把密钥明晃晃地放在代码里呢……

以及新东方和书链合作整了个这套系统，书链的东西大概也能用这一套流程。~~自动化代码嘛咕咕咕~~

<br>
