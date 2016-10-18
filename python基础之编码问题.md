# python基础之编码问题
## 本节内容
1. 字符串编码问题由来
2. 字符串编码解决方案

## 1.字符串编码问题由来

由于字符串编码是从ascii--->unicode--->utf-8(utf-16和utf-32等)演变过来的，再加上类似于中国的gbk编码等，这些编码互相之间并不兼容，所以编写的软件实现跨语言平台运行就会出现字符乱码问题。。。

须知内容如下：


1. 在python2默认编码是ASCII, python3里默认是utf-8(文件编码默认是utf-8，字符串编码默认是unicode)
2. unicode 分为 utf-32(占4个字节),utf-16(占两个字节)，utf-8(占1-4个字节)， so utf-8就是unicode
3. 在py3中encode,在转码的同时还会把string 变成bytes类型，decode在解码的同时还会把bytes变回string

## 2.字符串编码解决方案

首先，需要明白一点，unicode编码兼容所有编码格式，unicode编码在各种不同编码转换之间充当一个中间桥梁的角色，假如ascii编码要想转换成gbk编码，那就必须先解码，转换成unicode编码，然后再重新编码成gbk编码才算完成了整个过程。从其他编码转换成unicode编码的过程叫做解码(decode)，从unicode编码转换成其他编码的过程叫做编码(encode)。PS:utf-8编码默认不兼容gbk编码，需要转换成unicode编码才能兼容gbk编码。

涉及到编码解码方式可以参照如下图所示：
![](http://7xsn7l.com1.z0.glb.clouddn.com/编码解码.png)

编码问题涉及到如下几个方面：

1. 文件的编码格式
2. 字符串的编码格式
3. 输出字符串的终端编码格式

文件的编码格式和字符串的编码格式以及终端的编码格式一致才能正常的输出想要的字符串。

在python中进行转码的有两个函数，是encode()编码函数，以及decode()解码函数。其中encode函数中需要填上该字符串的源编码格式，decode函数中需要填上该字符串需要编码的字符串格式。测试代码如下，原编码格式是utf-8格式字符串：

	s="特斯拉"
	s_to_unicode=s.decode("utf-8")#解码成unicode编码格式
	print(s)
	print(s_to_unicode)
	unicode_to_gbk=s_to_unicode.encode("gbk")#编码成gbk编码格式
	print(unicode_to_gbk)
	gbk_to_unicode=unicode_to_gbk.decode("gbk")#解码成unicode编码格式
	print(gbk_to_unicode)
	unicode_to_utf8=gbk_to_unicode.encode("utf-8")#编码成utf-8编码格式
	print(unicode_to_utf8)