#列表,元组,集合

#列表
在python中是由数个有序的元素组成的数据结构,每一个元素对应一个index索引来隐式标注元素在列表中的位置.是python中最常用的一种数据类型.需要注意的是列表中可以有重复相同的数据.

###列表的常用操作:包括索引,切片,增删改查等.
```
#首先申明3个列表
list_all = ['22','sylar','22','open','4','sklfj','open',16,'sjk113','skf_kjdk','open','skj*&%)','f_s2df']
list_num = [1,4,7,3,5,4]
list_alp = ['axod','sccde','lsoM','JLKSI','KDIiid']

#切片,是用index来获取列表的某个或几个元素,其中多个元素用[start:end:step]来获取,index是从0开始,-1,-2来表示列表的最右的后几位
print(list_all[0])
print(list_all[2:10:2])
print(list_all[7:-1])
print(list_all[7:])
print(list_all[:7])


#append 在列表末尾添加一个值
list_alp.append('append')
print(' append 方法--->:',list_alp)

#count 记录一个值在列表中出现的次数
print(' count 方法--->:',list_all.count('open'))

#copy复制一个列表
l_copy = list_all.copy()
print(l_copy)

#clear清空列表
print(l_copy.clear())

#extend将两个表合并
list_all.extend(list_num)
print(' extend 方法--->:',list_all)

#insert插入一个元素,可以指定index来确定元素的位置
list_alp.insert(2,'sylar')
print(' insert 方法--->:',list_alp)

#index ,打印列表中某个值的index
print(' index 方法--->:',list_all.index('a'))

#pop 删除列表中的元素,默认是最后一个
print(' pop 方法--->:',' pop--->:',list_all.pop(),' list_all--->:',list_all)
print(' pop 方法--->:',' pop--->:',list_all.pop(0),' list_all--->:',list_all)
#remove 删除列表中找到的值,如果有相同的只删除找到的第一个,如果值不存在会引发一个错误
print(' remove 方法--->:',list_all.remove('open'),' list_all--->:',list_all)

#reverse 将列表的元素进行反向排序
print(' reverse 方法--->:',list_all.reverse(),list_all)

#sort按照ASCII码给列表排序
print(' 方法--->:',list_all.sort(),list_all)


#补充:一个循环列表比较好用的函数enumerate,可以快速获取列表的元素和对应的index
for index,item in enumerate(list_all):
    print(index,item)
#或者:
for index,item in enumerate(list_all,1):
    print(index,item)
```

#元组
与列表类似的一个数据结构,不同在于元组的元素不能修改.需要注意的是元组的元素不能修改,但是元组的元素的元素是可以修改的.

###元组的常用操作切片和列表一样,只是没有了修改元素的方法
```
#申明元组
tup1 = (1,2,3,['a','a', 'b'])

#切片
print(tup1[0])
print(tup1[0:8:2])
print(tup1[3:-1])
print(tup1[3:])
print(tup1[:3])

#index获取元素的index值
print(tup1.index(2))

#count对相同元素进行统计
print(tup1.count(5))

#补充 元素的元素可以修改,深浅copy的经典案例
tup1[-1][1] = 'c'
print(tup1)
```

#集合
由多个无序且不重复的多个元素组成数据结构.所以集合也没有index.通常用来做去重和关系测试工作!集合也是无序的
```
#申明集合使用set函数.可以是列表元组字符串等数据类型,
set_test1 = set(list_num)
print(set_test1)
set_test2 = set([2,4,45,43,3,6,0,98,9,7,6])
print(set_test2)
set_test3 = set('sylar')
print(set_test3)
set_test4 = set([1,3,5])
print('set4',set_test4)

#集合常用操作,增删和关系测试

#add添加元素
set_test1.add(45)
print(set_test1)

#copy,复制
set_copy = set_test1.copy()
print(set_copy)

#clear清空
set_copy = set_copy.clear()
print(set_copy)

#difference差集,在set_test1中有在set_test2没有
a = set_test1.difference(set_test2)
print(a)

#intersection交集,在set_test1中有在set_test2也有的.
a = set_test1.intersection(set_test2)
print(set_test1 & set_test2)
print(a)

#isdisjoint,在set_test4和set_test5中都没有相同的元素.则返回True,反之
set_test5 = set([2,4,6])
a = set_test4.isdisjoint(set_test5)
print(a)

#issuperset父集issubset子集,判断一个集合是不是另外一个集合的父集或者子集.返回值True或者False
a = set_test1.issuperset(set_test4)
print(a)
a = set_test4.issubset(set_test1)
print(a)

#pop删除并返回任意一个元素
a = set_test1.pop()
print(a)

#remove删除指定元素
print(set_test1.remove(7))

#symmetric_difference对称差集,去除两个集合中都有的元素之后的结果
print(set_test1.symmetric_difference(set_test2))

#union并集两个集合组合到一起的结果
print(set_test1.union(set_test2))

#update只能把可迭代的对象加入原集合
set_test1.update(set_test5)
print(set_test1)

#数学运算符&交集,|并集,- 差集,^ 对称差集
print(set_test1 & set_test2)
print(set_test1 | set_test2)
print(set_test1 - set_test2)
print(set_test1 ^ set_test2)
```

###补充上面3种数据结构都可以使用的
```
#in判断元素是否在集合中
for i in set_test2:
    print(i)
if 0 not in list_num:print('yes')
```