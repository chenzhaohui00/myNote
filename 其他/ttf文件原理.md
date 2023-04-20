#### 1.ttf文件的本质是什么？

和 Java 的 class文件 类似，就是一个存了一堆表的文件，约定好了 ttf文件 的规则，文件头有文件整体描述信息的表，以及后面所有的表的描述信息（表名，offset开始的位置，length描述信息等）。每个表中存储着自己的数据。

#### 2.字体文字是怎么以数字的形式存储在ttf文件中的？

字体文字其实本质上也是图形了，每个表中的数据存储着文字图形的数据，这些数据就是通过贝塞尔方程式描述曲线的数据，这样字体文字这种图形数据就变成了数字数据。

#### 3.如何查找一个具体的字形

我们知道，使用ttf的font时，只需要指定一个字符比如`\ue680`，就可以找到一个字形了。具体这个字符是如何在ttf文件中对应一个字形的呢，这个问题就要了解一下ttf文件中部分表的作用了。

一个简单的ttf文件的 所有表的位置描述 类似这种：

```yaml
{
	GSUB: {offset: 312, length: 66}
	OS/2: {offset: 380, length: 86}
	cmap: {offset: 476, length: 368}
	glyf: {offset: 852, length: 68}
	head: {offset: 224, length: 54}
	hhea: {offset: 188, length: 36}
	hmtx: {offset: 468, length: 8}
	loca: {offset: 844, length: 6}
	maxp: {offset: 280, length: 32}
	name: {offset: 920, length: 621}
	post: {offset: 1544, length: 47}
}
```

当然这不是ttf中真正存储的机构和样式，这里只是解析以后把位置信息统一列出来而已，方便看。

具体这些表中有一些涉及查找字体关键步骤的表有这个几个：

- glyf表：存储字形数据的表
- maxp表： 存储了glyf表中一共有多少个字形
- loca表： 存储了每个字形在 glyf表 中的位置（offset）
- cmap表：存储了每个字形的id和其在 glyf 中的 index。

所以查找一个文件就可以通过一下几个步骤：

1. 解析字符对应的id，比如 `\ue680`, 对应数值是 59008。
2. 从 cmap表 中找到此字形在 glyf表 中的 index
3. 去 loca表 中找到此字形在 glyf表 中的 offset
4. 去 glyf表 中找到这个字形的数据

#### 内容来源文章

[TTF文件探秘 - 掘金 (juejin.cn)](https://juejin.cn/post/6844904062928846862#heading-10)