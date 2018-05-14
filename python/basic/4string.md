#字符串处理

```
msg = 'my name is sylar'
```


#capitalize方法,将字符串的首字母大写
```
print 'capitalize方法:', msg.capitalize()
```
#swapcase方法用于对字符串的大小写字母进行转换。upper
```
print 'swapcase方法:', msg.swapcase()
print 'upper方法:',msg.upper()
msq_lower = msg.upper()
print 'lower方法:',msq_lower, msq_lower.lower()
```
#所有单词都是以大写开始，其余字母均为小写(见 istitle())
```
print 'title方法:',msg.title()
```


#count方法,查询子串出现的次数
```
print msg.count('name')
print 'count方法:', msg.count('a')
```
#center:用指定的字符,将字符串填充到指定的长度左
```
print 'center方法:', msg.center(40,'x')
```
#ljust将原字符串左对齐,并使用空格填充至指定长度的新字符串。如果指定的长度小于原字符串的长度则返回原字符.填充字符，默认为空格。
```
print 'ljust方法:', msg.ljust(20,'x')
```
#rjust将原字符串右对齐,并使用空格填充至长度 width 的新字符串。如果指定的长度小于字符串的长度则返回原字符串。
```
print 'rjust方法:', msg.rjust(20,'x')
```

###字符编码 #decode,encode
```
msg2 = '我是sylar'
msg3 = u'我是sylar'
print 'msg2:',msg2,type(msg2),'\tmsg3:', msg3, type(msg3)
msg2 = msg2.decode('utf-8')
msg3 = msg3.encode('utf8')
print 'msg2.decode:',msg2, type(msg2), '\tmsg3.decode:',msg3,type(msg3)
```

#endswith,startswith 查找字符串的开始和结束位置的字符串
```
print msg.endswith('sylar')
print msg.endswith('tom')
print msg.startswith('my')
print msg.startswith('MY')
```
#expandtabs 将制表符TAB替换为空格 windows这里不太明显.Linux下面是改变制表符的长度
```
msg_expandtabs = 'my name is\tsylar'
print 'expandtabs方法:',msg.expandtabs()
```
#fing,index方法 #index检查是否包含一个子串位置码和find方法一样,但是会报告一个错误返回!,find方法不会返回错误返回-1
```
print 'index方法:',msg.index('y')
```
#print 'index方法返回错误:',msg.index('sylra')
```
print 'find方法:',msg.find('y')
print 'find方法没找到:',msg.find('tom')
```
#rfind,rindex返回字符串最后一次出现的位置 find,index是第一次出现的位置
```
print 'rfind方法:',msg.rfind('y')
print 'rindex方法:',msg.rindex('y')
```
#format 格式化输出字符串
```
msg.format()
age = '88'
name = 'sylar'
print('{0} is {1} years old. '.format(name, age)) #输出参数
print('{0} is a girl. '.format(name))
```
#字符串类型判断
```
age_num = age.isalnum() #所有字符都是数字或者字母
age_dig = age.isdigit() #所有字符都是数字
age_alp = age.isalpha() #所有字符都是字母
age_low = age.islower() #所有字符都是小写
age_spc = age.isspace() #所有字符都是空白字符
age_tit = age.istitle() #所有单词都是首字母大写，像标题
age_upp = age.isupper() #所有字符都是大写
print '字符串类型判断:',age_num,age_dig,age_alp,age_low,age_spc,age_tit,age_upp
```

#lstrip方法用于 截掉字符串左边的空格或指定字符。rstrip 删除 string 字符串末尾的指定字符,
#strip方法用于移除字符串头尾指定的字符（默认为空格）
```
msg_strip = 'xxxkslfsjflsjxxx'
print 'lstrip方法:', msg_strip.lstrip('x')
print 'rstrip方法:',msg_strip.rstrip('x')
print 'strip方法:',msg_strip.strip('x')
```
#截取指定长度的字符串，原字符串右对齐，前面填充0
```
print 'zfill方法:',msg.zfill(10)
```


#replace方法 替换字符串中的字符串,如果指定第三个参数count，则替换不超过 count 次
```
print 'replace方法',msg.replace('is','\033[31;1mis\033[0m')
print 'replace方法',msg.replace('is','\033[31;1m%s\033[0m'%name)
```
#join将序列中的元素以指定的字符连接生成一个新的字符串
```
join_list = ['1','2','3','4']
msg_join =''.join(join_list)
print 'join方法:', msg_join ,type(msg_join)
```



#split,rsplit 将字符串格式化为列表,不指定切割字符默认为空格,还可以指定切割次数,split 从左往右,rsplit从右向左（默认为空格）
```
msg_spt = msg.split('y',1)
print msg_spt,type(msg_spt),msg_spt[0] #当列表内有中文是,直接打印列表返回Unicode编码,取值时则返回中文
msg_spt = msg.rsplit('y',1)
print msg_spt,type(msg_spt),msg_spt[0]
```
#partition 方法 将字符串划为元祖 根据指定的分隔符将字符串进行分割。
```
print 'partition方法:', msg.partition('y'), type(msg.partition('y'))
print 'rpartition方法:', msg.rpartition('y'),type(msg.rpartition('y'))
```
#splitlines按照行分隔，返回一个包含各行作为元素的列表，如果 num 指定则仅切片 num 个行
```
msg_splitlines='1\n2\n3\n4'
print 'splitlines',msg_splitlines.splitlines()
```




#translate()根据参数table给出的表(包含 256 个字符)转换字符串的字符, 要过滤掉的字符放到 del 参数中。