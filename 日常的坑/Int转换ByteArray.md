Int转换ByteArray

今天遇到这么一个问题，从UI层接受一个String的输入，但是实际输入的是一个数字，然后需要把这个数字转换成字节数组byte array，最开始尝试直接使用String.toByteArray去做，这个Kotlin方法其实就等于Java中通过String.getBytes来做。但是他的原理是把String分割成一个Char的数组，再把每个Char parse成Byte一次，比如输入“1000”，他最终就是把1，0，0，0分别转换成byte，也就是49,48,48,48。这显然不是想要的。

于是找String to ByteArray的方法找了半天，一直找不到合适的，后来发现其实这是一个误区，只要把这个String先转换成一个Int，然后想办法把Int再转成Byte Array就好了。那么Int有四个byte那么大，怎么转换成Byte数组呢，其实就很自然地把Int分为四个byte，也就是两个16进制位，或者8个二进制位一个Byte就可以了。然后依次把他们转成Byte，甚至于其实就不需要转，如果都换成二进制或者十六进制的表达方式，只需要按照位分割就好了。