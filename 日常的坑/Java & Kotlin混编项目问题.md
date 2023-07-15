## Kapt和AnnotationProcessor

如果在kotlin代码中需要使用到kapt，添加了kapt相关代码后可能会发现其他AnnotationProcessor的也都不生效了，都要改成kapt

这次的情况是这样：

1. 我在手头的Java项目中新加的一个Kotlin的类，其中使用了dagger

2. 然后发现这个类相关的component始终无法生成

3. 花费好一阵子才发现因为使用kotlin写的类，需要用kotlin的注解生成器kapt

4. 然后我添加了如下代码:

   ```groovy
   apply plugin: 'kotlin-kapt'
   
   //depencies中添加
   kapt rootProject.ext.dependencies["dagger2-compiler"]
   
   //网上查的，在结尾添加
   kapt {
       generateStubs = true
   }
   ```

5. 然后发现原来项目中用的butterknife也开始报错找不到view

6. 最后发现butterknife的注解处理器也要改成kapt，就是这玩意：

   ```groovy
   //原来的annotationProcessor不管用了
   annotationProcessor(rootProject.ext.dependencies["butterknife-compiler"]) {
       exclude module: 'annotation'
   }
   //改成下面的
   kapt(rootProject.ext.dependencies["butterknife-compiler"]) {
       exclude module: 'annotation'
   }
   ```

   