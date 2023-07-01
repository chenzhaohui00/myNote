编译报错：resource android:attr/dialogCornerRadius not found

关于这个问题，基本解决方案就是两种：

1. 升级CompileSdkVersion到28或以上

2. 保证compileSdkVersion、targetSdkVersion、buildToolsVersion三者的版本一致，比如这种：

   ```groovy
   android = [
           compileSdkVersion: 33,
           buildToolsVersion: "33.0.2",
           targetSdkVersion : 33,
           ...
   ]
   ```

   

StackOverflow问题：https://stackoverflow.com/questions/49280632/error9-5-error-resource-androidattr-dialogcornerradius-not-found