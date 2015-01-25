title: 关于python字符编码问题
date: 2015-01-25 15:11:38
categories: python
tags: 编码
---

#### 1. 编码

理解python编码的一个关键是，编码是一个怎么样的行为:

> 编码其实把内存中比较呆板的字节(二进制)变成人类能阅读的文字的过程;

比较形象的表示如下(图来自于[廖岳峰的python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386819196283586a37629844456ca7e5a7faa9b94ee8000)):

![图1](http://frgeek-bed.qiniudn.com/4Untitled Diagram.png)

##### 1.1 python

python2.x 默认支持两种字符串，一种是unicode，一种是str(见 code1);在理解这两者的时候，可以这个认为，unicode是比较原始的数据，没有经过任何的压缩编码，而str其实是通过相应的unicode经过特定的编码才生成的，如何证明这点呢，看下面code2；从代码可以得到，这里的str是通过unicode经过utf-8编码得到的;

```
	
	code1
	------------------------------
	In [30]: type('a')
	Out[30]: str
	In [31]: type(u'a')
	Out[31]: unicode
	
	code2
	-----------------------------
	# 定义一个str
	a = '汗'
	# 得到a的长度
	In [39]: len(a)
	Out[39]: 3
	# a的二进制表示
	In [38]: a
	Out[38]: '\xe6\xb1\x97'
	
	# 一个汉字用3个字节表示，我们可以认为他可能是使用utf-8编码的
	In [34]: a.decode('utf-8')
	Out[34]: u'\u6c57'

```

##### 1.2 encode和decode

在python中字符编码的转化必须通过unicode才可以，不能直接从一种编码直接转化为另外一种编码方式，只能把一种编码转化为unicode，然后unicode再转化为其他的编码;这个过程如下:

```
	utf-8 -----> unicode -----> GBK
	
	1. 先把utf-8通过decode变成unicode
	2. 在把unicode通过encode变成GBk
```
	
为什么从unicode到其他的编码称为编码，而从其他的编码变成unicode叫做解码呢？这里其实可以类比于视频中的解码，由于真实视频数据占用的空间比较大，所以通过一定的压缩和编码的方式可以大大减少文件的体积，但是当这个文件需要被播放的时候，还是需要经过解码把原始数据恢复出来才能进行播放的；我们把原始数据到不同文件类型的过程叫做编码，而相反的就是解码；而unicode就类似于这边的原始数据.这样就可以很好记忆这两个过程;

```
	code3
	----------------------
	# 定义一个unicode字符串
	In [42]: c = u'汉子'
	
	# c真实在内存中的样子
	In [43]: c
	Out[43]: u'\u6c49\u5b50'
	
	# print输出是经过转化处理的,变成当前终端的编码
	In [44]: print c
	汉子
	
	# 获取默认的区域设置并返回元祖(语言, 编码)
	In [60]: print locale.getdefaultlocale()
	('zh_CN', 'UTF-8')

	# 把unicode编码为utf-8
	In [45]: c.encode('utf-8')
	Out[45]: '\xe6\xb1\x89\xe5\xad\x90'

	# a就变成了utf-8的编码形式
	In [46]: a = c.encode('utf-8')
	In [47]: a
	Out[47]: '\xe6\xb1\x89\xe5\xad\x90'

	In [48]: print a
	汉子
```

**注意：如果一个unicode调用decode，或者某一个编码的字符串调用encode函数的话，python是不会抛出异常的，他仅仅会返回一个内容一模一样的字符串(但是在内存中去是不一样的,id不一样)**

##### 1.3 文件存储

1. 默认的open、write、read的方法：

python中的read默认返回str，如果在写入的时候希望变成另外一种编码的话，就必须先转化为unicode，然后在encode变成其他的编码str，然后写入;
	
```
	# coding: utf-8
	f1 = open('1.txt','r')
	content = f1.read()
	unText = content.decode('utf-8')
	f1.close()

	f2 = open('2.txt','w')
	f2.write(unText.encode('GBK'))
	f2.close()

		1.txt:枫琳
		2.txt:·ãÁÕ
```

2. codec中的open、write、read

codec模块的open可以设置文件的编码方式，而且从这边读取的内容都是unicode，而且写入的时候，如果参数为unicode的话，会按照open指定的方式写入文件了；如果不是unicode的话就有问题，理论上我在看blog的时候说会自动的进行转化成unicode的，但是我尝试以后发现不行；
	
后来我发现上面的问题是依赖于自己的环境的，我用sys.getdefaultencoding()返回的是ascii的编码，如果我使用一下code4代码把我默认的环境字符编码为utf-8；基于这样的环境，如果我向write传入的utf-8编码的字符串的时候，就可以成功；但是如果我传入的不是utf-8的话，比如GBk编码的参数，就会有问题;
	
所以我可以得出以下结论:
	
1. 对于codec的file对象，write传入unicode参数是可以正确的执行;
2. 如果传入的参数不是unicode，是一些具有特定编码的字符串的话，那么codecs会调用系统默认的字符编码来进行解析成unicode，如果与系统的默认字符编码有关系，所以只能适应向对应的字符编码;
	
```	
	f = codecs.open('test.txt', encoding='UTF-8')
```

```
	code4:

	import sys
	reload(sys)
	sys.setdefaultencoding('utf-8')
```

##### 1.4 unicode注意点

关于`'\u4546\u4565'` 与 `u'\u4546\u4556'`的区别，前者是str，而后者是unicode，但是这个可以进行转化，这个转化的方式如下:


	a = '\u4546\u4565'
	b = a.decode('unicode_escape')

	这之后就变成了unicode了;

#### 2. 总结

由于对python其实还不是很熟悉，如果在解释过程有什么错误，请发邮件给我，[我的邮件](smy19890720@gmail.com),我绝对会查证修改...





	
	
