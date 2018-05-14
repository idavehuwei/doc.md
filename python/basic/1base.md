#python3基础(一)

#### 1. python文件主程序入口文件一般来要申明python路径,编码信息,作者说明等:

```
#!/usr/bin/env python
# _*_ coding: utf-8 _*_
# Author: sylar
# Date:xxxx
# Dec: xxx
```

#### 2. python中相同的内存数据只会保存一份,变量赋值是通过复制内存地址进行.A复制给B.B是将A的内存地址复制一份,A重新赋值内存地址将变化,所以B不会改变.(因此造成了深浅copy的问题.)

```
a = 1
b = a
print(id(a),id(b))
print(a,b)
a = a + 1
print(id(a),id(b))
print(a,b)
```

####3. python3和python2的区别(比较简单的,复杂的会在后面慢慢深入):   
(1)python2默认已unicode编码处理代码,python3默认以utf-8处理代码.  
(2)python3没有raw_input只input,python2中不建议使用input  
(3)python3不能使用<>,只能使用!=来表示不等于  

####4. python中单引号'和双引号"作用完全一样.
####5. 变量命名只能使用字母数字和下划线,多个单词组合的变量名建议用_分割单词.全大写变量名为常量(约定俗成,变量本身是可以修改的).
####6. 字符串格式化方法  
(1)+号拼接

```
name = input('name:')
age = input('age:')
info = '''
------------info of ''' + name + '''------------
Name: ''' + name + '''
Age: ''' + age

print(info)

```
(2)%s格式化输出(注意%d,%s,%f的数据类型不同.input会默认接受数据类型为字符串)

```
name = input('name:')
age = input('age:')
info = '''
------------info of  %s------------
Name: %s
Age: %s
'''%(name,name,age)

print(info)
```

(3)format方法

```
name = input('name:')
age = input('age:')
info = '''
------------info of  {who}------------
Name: {who}
Age: {how_old}
'''.format(who=name,how_old=age)

print(info)
```

```
#不建议使用
name = input('name:')
age = input('age:')
info = '''
------------info of  {0}------------
Name: {0}
Age: {1}
'''.format(name,age)
print(info)
```

####7. type查看数据类型,help查看帮助, dir查看内置方法,id查看对象在解释器中的内存地址


