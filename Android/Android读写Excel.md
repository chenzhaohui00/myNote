## Android操作Excel现状

Android读写Excel的方式主要还是要通过两套java库，一个是jxl，全程是Java Excel Api，另一个是Apache poi。两者对比，前者作为一套纯java api，跨平台支持的很好，而后者功能更加强大，能更好地支持图形、图标、公式、宏等各种格式，而且后者效率更高。

但是在android的使用上，这两者都有几个问题

1. 内存优化比较差，读取大量数据容易出现oom
2. 作为一套基础的api，使用起来没有那么无脑和简单，会有一些学习成本
3. 都是java库，缺少高质量的基于他们做二次开发的android library。

尝试在github上一搜，针对android的excel操作的库基本都是基于这两者的，而且专门做android的这种库的star都很少。可能是因为毕竟android上这方面的需求比较有限。



## 使用EsayExcel在Android端的尝试

目前找到的做excel操作的最好的一个java的库就是[EasyExcel](https://github.com/alibaba/easyexcel)，他是基于poi做的二次开发，而且做了内存优化，使用起来也简单，也就是说解决了我们上面提到的前两个问题，而且有28k的star数，唯一的问题就是，作为一个基于apache poi做的二次开发的库，他本身没有对android做很好的的适配。

从网上的资料来看，历史版本中有一些在android上成功使用起来的先例，但是我尝试了一下，有几个问题不好解决，最后还是放弃了。这里说一下遇到的几个问题。

### 1. xml解析的报错

具体来说就是，一调用写excel的方法，就会报错：`java.lang.NoClassDefFoundError: Failed resolution of: Ljavax/xml/stream/XMLStreamReader`或`NoClassDefFoundError: Failed resolution of :Ljavax/XML/stream/XMLEventFactory`。

这个原因就是apache poi使用到了同是apache的一个xml的库，这个库在java环境下是有的，而android环境并没有，stackoverflow上有网友给出的解决方案是手动导入他使用到的包：`implementation 'javax.xml.stream:stax-api:1.0'`。

另外，针对xml还有一些其他的报错，看issue中说的是easyExcel使用的一个xmlbeans包的一些版本会在android平台上报错，给的解决方式是手动导入一个不会出错的xmlbean的包，并且在导入easyExcel的时候做exclude，像这样：

```
implementation ("com.alibaba:easyexcel:3.3.2") {
    exclude(group: "org.apache.xmlbeans", module: "xmlbeans")
}
implementation group: 'org.apache.xmlbeans', name: 'xmlbeans', version: '3.1.0'
```

### 2. 缺少awt包

解决上面的问题以后，就开始报一个这种的报错：`java.lang.ClassNotFoundException: Didn't find class "java.awt.font.FontRenderContext`

又缺少了awt包，那么awt是什么呢？查了一下，发现是java做跨平台的UI的库，全称叫"Abstract Window Toolkit"。这个库就不少解决了，搜了一顿，也没发现什么比较好的解决方案。在github中发现了一个尝试解决这个问题的library，就是这个 [poi-on-anrdoid](https://github.com/centic9/poi-on-android)，虽然他跟esayExcel没有什么直接关系，但是它是专门做的android上poi的适配和使用的一个sample project。

这个library里面有一个module叫poishadow的，号称可以帮助生成apache POI的jar包，其中包括所有必要的依赖项，并修复了一些通常阻碍您在Android上部署Apache POI的事情。我尝试了一下在用esayExcel的测试项目中copy了这个module，发现问题没有解决。具体他这个module里面build.gradle的一些细节由于我菜鸡的gradle和groovy水平，并没有看出来。



## 调研结果

时间关系，这个调研就到此结束了。期间我一度生出了自己去学一下jxl或poi的api，然后自己写的想法。但是理智告诉我，花费再多的精力有点不值得了，对于这个简单的导出需求，自己用jxl和poi写如果再出现问题时间上可能变得更加不可控，之前领导有说过可以用csv的格式导出，我简单看了下csv格式，十分简单，在没有一些特殊字符的情况下甚至不需要做任何特殊处理，只需要用逗号和\r\n做基本的分隔即可。最终决定这次需求用csv。

虽然这是一次失败的调研和尝试，但是至少也了解了android上目前对excel的导入导出的一些现状。如果以后依然有这方面的需求，可以快速尝试一下star没有那么多的android上读写excel的库，有可能也有简单好用且稳定的库，在不然就得自己写了，估计jxl的api会更简单一些，而且基于基本的java api，跑在android上问题也不大，对于内存问题，就自己控制些读写的量应该就可以了，别一次都读出来放内存里。





