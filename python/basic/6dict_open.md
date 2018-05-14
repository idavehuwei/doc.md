#dict字典和文件操作open
#字典dict
使用key来标注value的数据类型,key和value是一一对应的.在字典中key是唯一的,所以字典也是无序的.

```
#定义一个字典
dict = {
    'name' : 'sylar',
    'age' : 18,
    'post' : 'OPS',
    'salary' : 80000
}

###常用操作和方法

#访问字典的数据,使用定义的key来获取
print(dict_test['name'])

#修改字典
dict_test['name'] = 'Sylar'
print(dict_test['name'])

#copy复制一个字典
#clear删除所有元素
a = dict_test.copy()
print(a)
print(a.clear())

#fromkyes创建一个新字典，以序列中元素做字典的键，val 为字典所有键对应的初始值
a = {}
a = a.fromkeys([1,2,3,4,5],'defalt')
print(a)

#get获取元素值同dict_test['name']
print(dict_test.get('name'))

#item遍历所有元素,将元素以键值对元组的方式输出
print(dict_test.items())

#keys遍历所有元素的value
print(dict_test.keys())

#pop删除一个元素,并返回元素的value值
print(dict_test.pop('age'))
print(dict_test)

#popitem删除一个元素,并以元组的形式返回key,value值
print(dict_test.popitem())
print(dict_test)

#setdefault设置一个默认值的元素,如果key已经存在则返回对应的value
print(dict_test.setdefault('a',100))
print(dict_test)

#update用一个字典的数据去更新另外一个字典的数据
a = {'name':'tom','a':'b'}
dict_test.update(a)
print(dict_test)

#valus遍历所有元素的value
print(dict_test.values())

遍历字典的方法
for i in dict_test:
    print(i, dict_test[i])
for i in dict_test.items():
    print(i)
for k,v in dict_test.items():
    print(k, v)
```

#文件操作
对文件的打开读写是用open函数,打开文件的方式又分为:
r	以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。

rb	以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。

r+	打开一个文件用于读写。文件指针将会放在文件的开头。

rb+	以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。

w	打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。

wb	以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。

w+	打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。

wb+	以二进制格式打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。

a	打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。

ab	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。

a+	打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
ab+	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。

简单来说常用的:r+先读后写从头开始覆盖写.w+先写后读,默认先创建一个空文件默认也只能追加写在文件最后面(会清空同名文件谨慎使用).a+是追加读写模式,一般情况都用这种模式来操作文件.文件插入数据很麻烦,.所以要修改文件,建议读出一个文件写到一个新文件里,数据替换完成后替换原文件
```
#open函数使用方式,用一个文件句柄的方式去打开一个文件.操作完以后必须关闭文件.
f = open('a.txt','r+',encoding='utf-8')
a = f.read()
f.close()
print(a)

#常用方法

#read读取整个文件,由于是一次性读取整个文件只适合对小文件使用.可以指定参数读取多少个字节
a = f.read(1)
#close关闭文件
#readline每次读取一行
#readlines读取整个文件,可以指定参数读取多少行
#write写文件
#truncate截断一个文件,保留多少个字节
#flush手动将内存缓存的数据刷写到硬盘中,而不是等待程序的缓存控制策略去刷写数据
#tell返回文件指针的位置
#seek移动文件指针的位置
f = open('a.txt','r',encoding='utf-8')
print(f.readline())
print(f.readline())
print(f.readline())
print(  f.tell())
f.seek(10)
print(  f.tell())
print(f.readline())
f.close()


```

###with open as f 
为了更好保护文件,python中提供了with函数来保护对文件的操作.所以我们要进行文件操作一定要使用with函数.在python3中with支持同时操作多个文件

```
#因为读取文件是按行读取所以for line in f:,而不是使用for i in readline:.后一种方法是读取每一行的字符串的每一个字符
with open('a.txt','r',encoding='utf-8') as f,\
     open('b.txt','w+',encoding='utf-8') as f1:
    for line in f:
        text = line.strip()+'\n'
        f1.writelines(text)

```