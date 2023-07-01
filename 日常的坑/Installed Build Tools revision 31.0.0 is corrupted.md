升级`buildToolsVersion`到 `31.0.0` 以上，报错“Installed Build Tools revision xx.x.x(eg. 31.0.0) is corrupted”

去对应目录，把所有d8改为dx，在win上：

1. 到 "C:\Users\user\AppData\Local\Android\Sdk\build-tools\31.0.0"下，把d8.bat改为dx.bat
2. 进入lib目录，把d8.jar改为dx.jar

在mac上:

1. 到 ~/Library/Android/sdk/build-tools/31.0.0 下
1. d8改为dx
1. 进入lib目录，把d8.jar改为dx.jar