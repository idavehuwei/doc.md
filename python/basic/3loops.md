#loops循环语句

####一 if语句,if语句配合else使用,可以没有else.
1. 单分支if语句

```
age = input('Age:')
password = '67'
if age == password:
    print('==')
```

2. 多分支语句

```
age = int(input('Age:'))
password = 67
if age == password:
    print('==')
if age > password:
    print('>')
if age < password:
    print('<')
else:
    print('age error')
#这个例子并不符合逻辑,只是为了举例
```

需要注意的是else语句是否执行,是判断同级上一个if语句条件不满足就执行else,在上一个if前面的if并不影响else的执行.所以就应该使用elif

```
age = int(input('Age:'))
password = 67
if age == password:
    print('==')
elif age > password:
    print('>')
elif age < password:
    print('<')
else:
    print('age error')


```



####二 while语句,无限循环使用break语句跳出(可以没有跳出语句,但是就要注意设置while的跳出条件,不然会死循环)

1. 满足自定条件跳出

```
count = 0
while True:
    count = count + 1
    print(count)
    if count == 20:
        break
```
2. 定义退出的条件

```
count = 0
while count < 10:
    count = count + 1
    print(count)

```
3. while也可以配合else执行一些特殊的代码


```
password = 67
count = 0
while count < 5:
    count = count + 1
    age = input('Age:')
    if age.isdigit():
        age = int(age)
        if age == password:
            print('Very Good!')
            break
        elif age > password:
            print('>')
        else:
            print('<')
    else:
        print('The input is not the digit, the program exits')
        break
else:
    print('Try it too many times')
print('Always executed')
```

####三 for语句类似while语句,for是遍历一个条件,不会形成死循环(相对)所以可以不用设置退出条件.
1. 简单的for循环,

```
for i in range(10):
    i = i + 1
    print(i)
    
for i in [1, 6, 22, 48, 10, 8, 33 ]:
    print(i)
    if i == 8:
        break
```

2. 合理使用while和for能优化代码

```
passwd = 67
for i in range(10):
    age = input('Age: ')
    if age.isdigit():
        age = int(age)
        if age >passwd:
            print('>')
        elif age < passwd:
            print('<')
        else:
            print('Very Good!')
            break
else:
    print('The input is not the digit, the program exits')
print('Always executed')
```



####补充: 
1. continue和break的区别,continue是跳过本次循环进行下一次循环,break是跳出整个循环语句


```
count = 100
for i in range(10):
    print('In this loop count is :',count)
    a = input('Whatever:')
    if a == 'c':
        continue
    if a == 'b':
        break
    if a == 'q':
        exit()
    count = count + 1
print('Other things!')
```

2. return跳出多重循环(其实ruturn不是这个作用,但是现在暂时只能想到这个办法比较简单)

```
count = 0
for i in range(10):
    print('-loop- i: ',i)
    for j in range(10):
        print('--loop-- j: ', j)
        for k in range(10):
            print('---loop--- k:', k)
            count = count + 1
print(count)
```
这个循环会让count累加到1000,假如我们给定到达888的时候跳出该怎么做呢?
```
flag = False
count = 0
while not flag:
    for i in range(10):
        print('-loop- i: ',i)
        while not flag:
            for j in range(10):
                print('--loop-- j: ', j)
                while not flag:
                    for k in range(10):
                        print('---loop--- k:', k)
                        count = count + 1
                        if count == 888:
                            print(count)
                            flag = True
```

上面这种做法,我们可以看到i和j还是循环多了两次,因此在每层循环必须都判断一下count的数值.
正确的做法是封装一个函数
```
def mul_loops():
    flag = 0
    for i in range(10):
        print('-loop- i: ', i)
        for j in range(10):
            print('--loop-- j: ', j)
            for k in range(10):
                print('---loop--- k:', k)
                if flag == 888:
                    return flag
                flag = flag + 1
a = mul_loops()
print(a)
```