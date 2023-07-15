## resource android:attr/dialogCornerRadius not found

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



## You need to use a Theme.AppCompat theme (or descendant) with this activity

这是因为：

1. Activity继承的是AppCompatActivity，但是theme并没有用Theme.AppCompat的theme或子theme。
2. 在Dialog中错误地使用了getApplicationContext()，导致他认为我们的activity用的theme不对。

