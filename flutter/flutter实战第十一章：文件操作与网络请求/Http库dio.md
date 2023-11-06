## 引入dio

引入

```yaml
dependencies:
  dio: ^x.x.x #请使用pub上的最新版本
```

导包并创建实例

```dart
import 'package:dio/dio.dart';
Dio dio =  Dio();
```

一个dio实例可以发起多个http请求，一般来说，APP只有一个http数据源时，dio应该使用单例模式。

## 发起请求

### Get

```dart
Response response;
response=await dio.get("/test?id=12&name=wendu")
print(response.data.toString());
```

把query参数通过对象传入：

```dart
response=await dio.get("/test",queryParameters:{"id":12,"name":"wendu"})
print(response);
```



### Post

```dart
response=await dio.post("/test",data:{"id":12,"name":"wendu"})
```



### 发送多个并发请求

```dart
response= await Future.wait([dio.post("/info"),dio.get("/token")]);
```



### 下载文件

```dart
response=await dio.download("https://www.google.com/",_savePath);
```



### 发送 FormData

```dart
FormData formData = FormData.from({
   "name": "wendux",
   "age": 25,
});
response = await dio.post("/info", data: formData)
```



### 通过FormData上传多个文件

