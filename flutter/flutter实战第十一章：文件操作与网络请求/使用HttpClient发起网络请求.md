## 使用HttpClient发起网络请求

dart的io库中有一些Http请求的类，可以用`HttpClient`类发起请求

### 1. 创建HttpClient

```dart
HttpClient httpClient = HttpClient();
```

### 2. 打开Http连接，设置请求头

先构建uri，可以添加Query参数：

```dart
Uri uri = Uri(scheme: "https", host: "flutterchina.club", queryParameters: {
    "xx":"xx",
    "yy":"dd"
  });
```

然后通过上面创建的uri创建request：

```dart
HttpClientRequest request = await httpClient.getUrl(uri);
```

设置header：

```dart
request.headers.add("user-agent", "test");
```

如果是post或put等可以携带请求体方法，可以通过HttpClientRequest对象发送请求体，如：

```dart
String payload="...";
request.add(utf8.encode(payload)); 
//request.addStream(_inputStream); //可以直接添加输入流
```

### 3. 发起请求并等待响应

```dart
HttpClientResponse response = await request.close();
```

### 4. 读取响应内容

`HttpClientResponse`对象，它包含响应头（header）和响应流(响应体的Stream)，接下来就可以通过读取响应流来获取响应内容。在读取时我们可以设置编码格式，这里是utf8。

```dart
String responseBody = await response.transform(utf8.decoder).join();
```

### 5. 请求结束，关闭`HttpClient`

```dart
httpClient.close();
```

关闭client后，通过该client发起的所有请求都会终止。



## HttpClient常用配置

| 属性                  | 含义                                                         |
| --------------------- | ------------------------------------------------------------ |
| idleTimeout           | 对应请求头中的keep-alive字段值，为了避免频繁建立连接，httpClient在请求结束后会保持连接一段时间，超过这个阈值后才会关闭连接。 |
| connectionTimeout     | 和服务器建立连接的超时，如果超过这个值则会抛出SocketException异常。 |
| maxConnectionsPerHost | 同一个host，同时允许建立连接的最大数量。                     |
| autoUncompress        | 对应请求头中的Content-Encoding，如果设置为true，则请求头中Content-Encoding的值为当前HttpClient支持的压缩算法列表，目前只有"gzip" |
| userAgent             | 对应请求头中的User-Agent字段。                               |

这些配置可以通过`HttpClient`设置，也可以通过`HttpClientRequest`设置，区别就是全局还是只针对当前请求起效。



## HTTP请求认证

Http有一个认证机制，就是保护部分资源，请求时需要帐户密码。Http认证包括：Digest 认证、Client 认证、Form Based 认证等。感觉使用很少，暂时不管。



## 代理

也跳过。



## 证书校验

默认https自己的tsl/ssl的部分不用我们处理，但是自签名的证书或者非CA认证的证书则需要我们手动校验。`HttpClient`的校验逻辑如下：

1. 如果请求的Https证书是可信CA颁发的，并且访问host包含在证书的domain列表中(或者符合通配规则)并且证书未过期，则验证通过。
2. 如果第一步验证失败，但在创建HttpClient时，已经通过 SecurityContext 将证书添加到证书信任链中，那么当服务器返回的证书在信任链中的话，则验证通过。
3. 如果1、2验证都失败了，如果用户提供了`badCertificateCallback`回调，则会调用它，如果回调返回`true`，则允许继续链接，如果返回`false`，则终止链接。

### 示例：

```dart
String PEM="XXXXX";//可以从文件读取
...
httpClient.badCertificateCallback=(X509Certificate cert, String host, int port){
  if(cert.pem==PEM){
    return true; //证书一致，则允许发送数据
  }
  return false;
};
```

这是存了证书的内容，还可以添加证书到本地证书信任链中

```dart
SecurityContext sc = SecurityContext();
//file为证书路径
sc.setTrustedCertificates(file);
//创建一个HttpClient
HttpClient httpClient = HttpClient(context: sc);
```
