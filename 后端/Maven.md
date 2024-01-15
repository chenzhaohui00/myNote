## Maven目录

- maven目录
  	- bin： 运行脚本，要配环境变量的目录
   - conf：配置文件目录
     - settings.xml : 主要要配置的文件
     - toolchains.xml 
   - lib：maven也是用java写的，这里是运行Maven所需的java类库
   - boot：类加载器，先不用管。

## Maven配置

就是修改上面提到的settings.xml，主要要修改三处：

1. 本地仓库位置： `<localRepository>`标签
2. maven下载镜像：`<mirror>`标签
3. mavent编译项目时的jdk版本：`<profile>标签`

### IDEA中的Maven配置

在设置中的`Maven`中配置Maven的`home path`和`settings.xml` 文件的位置，repository会自动督导`settings`中配置的位置



## GAVP属性

GAVP 分别指：`GroupId`，`ArtifactId`，`Version`，`Packaging`四个属性。前三个是必要，打包方式`Packaging`有默认值。

- GroupId：公司.业务线，eg: com.taobao.tddl
- ArtifactId：产品名-模块名，eg: tc-client
- Version:  主版本号.次版本号.修订号
- Packagin: jar(默认值)、war（java web工程）、pom（不打包，做继承的父工程）



查常用的一个maven库的网址：mvnrepository.com



## IDEA内建Maven的java项目和web项目

### java项目

new module 时 build system 选择 `maven`，然后进去以后就可以改pom.xml来修改配置，在`<dependencies>`中添加需要的lib

### web项目

1. 先建普通的java项目
2. 两种方式
   1. 用插件：JBLJavaToWeb
   2. 手动：
      1. 在`pom.xml`中把`packaging`改为`war`
      2. 配置号resource目录和web.xml的目录
      3. 创建Maven Archetype module，archetype选择webapp

