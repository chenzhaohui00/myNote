# AOSP下载编译踩坑

## 硬件要求

内存：make的时候要求最低16G内存

硬盘：起码500G吧，我给了600G，在折腾的过程中依然出现过硬盘不足的情况，因为光初始源码包就要80G，解压缩完要200G，如果期间失败了，删除到回收站里，再重新解压什么的，搞着搞着可能就硬盘空间不足了。。。

## Ubuntu系统安装

整体也是装个系统盘，然后安装，整个是按照这个视频来的：

https://www.bilibili.com/video/BV1Cc41127B9/

## 踩坑

### 1. Repo版本未更新

`repo sync` 的过程中报错如下：

```
Fetching: 100% (1352/1352), done in 9m42.033s
info: A new version of repo is available
warning: repo is not tracking a remote branch, so it will not receive updates
=========================================================================================================
Repo command failed: repounhandledexceptionerror gitcommanderror: 'reset --keep v2.44^0' on repo failed stderr: error: entry 'project.py' not uptodate. cannot merge. 
fatal: 不能重置索引文件至版本 'v2.44^0'。
```

看起来似乎是说repo版本更新了，但是查了一圈又没有发现怎么更新repo版本，而且都说repo可以自己更新。而我们前面下载repo的时候，下的也是一个单独的文件，里面也只是定义了很多repo的方法，不是一个完整的软件。

后面仔细了解了一下，repo是安装在aosp源码中的，要更新repo版本，需要cd到 aosp 中的 repo 子项目的目录下，去 git pull 一下子就好了。也就是：`cd .repo/repo`， 然后 `git pull`即可。



### 2. repo init的时候报错 has been initialized，让删除.repo然后try again？

`repo sync` 以后需要 repo init 到某一个版本的分支上，init 以后就会弹出提示:

```
repo has been initialized in /home/cyq/AOSP/aosp
If this is not the directory in which you want to initialize repo, please run:
	rm -r /home/cyq/AOSP/aosp/.repo
and try again.
```

这个地方看起来是说之前已经初始化过了，然后让删掉这个 .repo 再重试，但其实没有报错，这就是初始化成功了，他就是告诉我们初始化到了这个位置，如果初始化的位置不错，让我们把他删掉然后重试。



### 3. 不同版本选择导致的make时的报错

我刚开始想看android13的源码，于是选了一个android13.0_r83的分支，是android13的最后的一个release分支，但是代码同步下来make的时候各种报错。尝试了一些办法解决，很费时间而且并没有成功解决问题。

后来我重新做一遍全部流程的时候，选择了一个android12的最后的release分支，好像是android12.1_r29，整个流程走下来make的时候没有遇到什么问题。

所以，如果编译某一个版本遇到很多问题，怎么都编译不过的话，就换一个版本吧，有可能问题就迎刃而解了。



### 4. ASFP

Google给看aosp源码专门做了个AS的版本，叫Android Studio For Platform，简称为ASFP。它的好处是：

1. 直接识别并导入aosp项目
2. 集成了AOSP的编译工具链和build系统，比如有Soong和Makefile
3. 可以调试AOSP代码，包括内核服务、HAL层以及系统组件。还可以设断点，观察变量等。
4. 更好的源码索引和导航能力，帮助我们更好地理解系统服务之间的依赖关系



### 5. ASFP警告 External file changes maght be slow 或者 inotify(7) setting too low

inotify是linux系统的一个内核子系统，用于监视文件的增删改。Linux系统默认的`fs.inotify.max_user_watches`是65536，但AOSP项目文件有几十万甚至更多，就会报这两个警告.

这时候可以通过这种方式设置：`sudo sysctl fs.inotify.max_user_watches=524288`. 这个方式设置的这个watch值在重启以后就失效了，这时候就可以在/etc/sysctl.conf文件中添加如下配置：

```
fs.inotify.max_user_watches=524288
sudo sysctl --system
```

这样重启以后就依然会设置这个值了。





