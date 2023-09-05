## 文件名显示为`\345\256\236`这种数字编码

查了一下，是 git 对于`0x80` 以上的字符进行`quote`，这样可以使用如下指令如下即可关闭此功能：

`git config --global core.quotepath false`