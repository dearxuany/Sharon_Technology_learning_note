# Python 随机与统计
## random
模块random：包含生成伪随机数的函数。</br>
</br>
为何是伪随机？</br>
就像一个自认为很客观的人始终会有自己的偏好一样，计算机的随机其实不是真正的随机，它背后的逻辑是可预测的。</br>
用于加密和安全时，应选用os模块中的urandom函数。random模块中SystemRandom也有类似的功能，可提供接近于真正随机的数据。</br>
```
>>> import random

# 函数random输出0-1（含0和1） 伪随机数（浮点型），其他位数按需求做乘法
>>> random.random()
0.01531871642958682
>>> random.random()*10
2.613878017481688
>>> random.random()*100
44.130898788882874
>>> random.random()*100000
18424.90026089124

# 随机返回指定范围n到m（含n和m）的实数（此处用均匀分布，统计相关还有其他分布形式可选）
>>> random.uniform(1,233)
77.61569907520544
>>> random.uniform(165455,3633333)
535833.2506287073

# 生成随机整数，支持设置范围和步长，参数：start,stop,step，注意范围含start不含stop
>>> random.randrange(1,10)
4
>>> random.randrange(1,10,2)
5
>>> random.randrange(1,10,2)
1
>>> random.randrange(1,10,2)
9

# 在序列中抽取随机数
>>> x=['a','f','h','j','kj',1,2,3]
>>> random.choice(x)
'j'

# 打乱序列中元素顺序并确保每种可能输出概率相同，函数直接修改原序列
>>> x
[1, 'h', 'kj', 3, 'a', 2, 'j', 'f']
>>> random.shuffle(x)
>>> x
[3, 'kj', 'j', 2, 1, 'a', 'f', 'h']

# 均匀在指定范围内随机抽取n个样本
>>> x
[3, 'kj', 'j', 2, 1, 'a', 'f', 'h']
>>> random.sample(x,5)
[2, 'h', 'a', 3, 'f']
>>> random.sample(range(0,10),5)
[9, 3, 4, 0, 1]
```
实例：</br>
掷骰子游戏</br>
```
#! /usr/bin/python3

import random

def diceGame():
    diceNum=int(input('How many dices do you want?'))
    diceFace=int(input('How many fices of each dice do you want?'))

    resultList=[]
    for n in range(1,diceNum+1):
        result=random.randrange(1,diceFace+1)
        resultList.append(result)

    print(resultList)

def test():
    try:
        diceGame()
    except:
        print('The input should be positive integer!')
        test()

test()
```
```
$ python3 DiceGame.py
How many dices do you want?3
How many fices of each dice do you want?6
[2, 3, 4]

$ python3 DiceGame.py
How many dices do you want?s
The input should be positive integer!
How many dices do you want?4
How many fices of each dice do you want?g
The input should be positive integer!
How many dices do you want?4
How many fices of each dice do you want?6
[2, 4, 5, 2]
```
实例：</br>
从英文标准字典文件中随机抽取一个词
```
#! /usr/bin/python3

import random
import fileinput

# 从文件获取字典信息
englishDict=list(fileinput.input())
# 随机输出一个词
print(random.choice(englishDict))
~                                      
```
```
# 字典信息由linux标准字典文件/usr/share/dict/words提供
$ python3 EnglishWordsRandom.py /usr/share/dict/words
butane
```

## 拓展
### linux和macOS 标准字典文件
linux和macOS 标准字典文件：/usr/share/dict/words </br>
https://en.wikipedia.org/wiki/Words_(Unix)
