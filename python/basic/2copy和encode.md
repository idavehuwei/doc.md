#编码,深浅copy
##encode,decode在python2中使用的一些迷糊,python3中更容易理解
要理解encode和decode,首先我们要明白编码,字符和二进制的相关概念,简单来说,计算机的产生和设计只能进行二进制的运算,聪明的先辈们发明了ASCii编码用不同的二进制来表示不同英文字符实现.后来由于各国计算机的普及和推广,由于语言的原因产生了:Unicode,GB2312,UTF-8等不同的字符编码集.

在python3中对字符数据是使用的str类型,二进制数据使用的是bytes.而我们要让计算机能够处理字符串就必须能将字符转换为二进制数据(我们在idle中编写的代码这个步骤由解释器处理了).有时候为了我们在编写代码时能灵活的处理数据,就需要字符和二进制数据的转换.就产生了encode和decode方法,两者的关系如下:

```
#str->bytes:encode编码
#bytes->str:decode解码
str_a = str('我是二进制')

#编码,
bytes_uft8 = str_a.encode(encoding='utf-8')       #字符串转换为utf-8的二进制
print(bytes_uft8,type(bytes_uft8))

bytes_gbk = str_a.encode(encoding='gbk')          #字符串转换为gbk的二进制
print(bytes_gbk,type(bytes_gbk))

#解码
str_uft8 = bytes_uft8.decode(encoding='utf-8')   #二进制转换为utf的字符串
print(str_uft8,type(str_uft8))

str_gbk = bytes_gbk.decode(encoding='gbk')       #二进制转换为gbk的字符串
print(str_gbk,type(str_gbk))


#保存到文件中,utf-8.txt将编码换为GBK会乱码,是因为GBK不兼容utf8.
#但是utf-8兼容gbk,所以gbk.txt用utf-8不会乱码.
with open('gbk.txt','wb') as f,open('utf-8.txt','wb') as f1:
    f.write(bytes_gbk)
    f1.write(bytes_uft8)
```
编码是将字符串转换成二进制编码.   
解码是将二进制编码转换为字符串.   
python2中默认使用ASCII编码,python3使用Unicode   
获取系统默认编码:
```
import sys
print(sys.getdefaultencoding())
```

简单来说就是Unicode是二进制字符编码,兼容GBK,UTF-8,所有字符编码要转换都需要通过Unicode来作为桥梁来相互转换.在python解释器使用的字符编码是Unicode.中国使用的操作系统默认编码是GBK.而我们常用程序又有可能使用的其它国家的编码.所以这之间就存在一个编码和解码的过程.

所以我们记住utf8的python字符可以转化为gbk,big5等编码转换,但是要吧gbk,big5等编码转换为utf8必须要知道字符原来使用的那种编码集.不知道原来的字符集就没有办法转化为utf8.
```
#打开test.txt切换编码为utf8,gbk,big5可以看到不同效果
a = '中國'
b_utf8 =a.encode(encoding='utf8')
b_big5 =a.encode(encoding='big5')
b_gbk = a.encode(encoding='gbk')

with open('test.txt','wb') as f:
    f.write(b_utf8)
    f.write(b_big5)
    f.write(b_gbk)
```



最后:   
1, python3是支持直接使用UTF-8(因为UTF-8是Unicode的一个扩展集,能够自动转换),但是在不同语言系的操作系统中可能会乱码.所以知道为什么有时候编写程序的时候全是乱码b'\xxx\xxx'的数据了吧,这是字符文本在Unicode中的编码  
2, 关于为什么输出的是\xxx的数据.这个又涉及到二进制和16进制的转换了,我猜想应该是Unicode是使用16进制来记录二进制.

###深浅copy
浅copy的方式,浅copy也是一种面向对象的实现,初学者可以这样去理解浅copy只是复制了对象的内存地址,对于一些复杂的数据类型对象使用的内存地址进行.
当列表中包含一个子列表时,复制的也是这个子列表的内存地址.
```
#申明一个列表
human = ['name',['age',18]]
#使用几种浅copy方式复制数据
p1 = human[:] #切片的方式
p2 = list(human) #使用内置函数

import copy
p3 = copy.copy(human) #使用copy模块浅copy方法

p4 = human.copy() #使用列表的copy方法,其实也是引用的copy模块浅copy的方法 


```
这时候我们假如human是同一个生日的同学类型.他们的name属性不一致,但是随着时间的变化,年龄都会增长,所以他们的age的变化都会一致

```
p1[0] = 'dongxie'
p2[0] = 'xidu'
p3[0] = 'nandi'
p4[0] = 'beigai'
print(p1,p2,p3,p4)
#这时候不管修改谁的年龄,大家年龄都会变
p1[1][1] = 23
print(p1,p2,p3,p4)

```