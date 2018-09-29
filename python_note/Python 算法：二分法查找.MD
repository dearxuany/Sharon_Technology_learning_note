# 二分法查找 Binary search
时间复杂度：O(logN)</br>
缺点：数据必须是有序的</br>
</br>
* 二分法查找的 python3 实现</br>
给出列表和要找的数值，输出该数值在列表中的index。</br>
如果该元素在列表中存在，则输出index，跳出循环条件为目标数值和mid所指向的元素值相等；</br>
如果该元素在列表中不存在，则返回None，跳出循环条件为low大于high。</br>
```
#! /usr/bin/python3

def binarySearch(goalRange,goalNum):
    global mid
    rangeLength=len(goalRange)
    high=len(goalRange)-1
    low=0
    while low <= high:
        mid=int((high+low)/2)
        if goalNum > goalRange[mid]:
            low=mid+1
        if goalNum < goalRange[mid]:
            high=mid-1
        if goalNum == goalRange[mid]:
            return goalRange[mid]
    return None
```
将以上代码放入python3的自定义模块所在的目录中
```
>>> import python3_algorithms_BinarySearch
>>> list=[2,5,6,8,12,44,67,89,99,102,445,667,888]
>>> python3_algorithms_BinarySearch.binarySearch(list,99)
8
>>> python3_algorithms_BinarySearch.binarySearch(list,666)
>>>
```

* 基于二分法查找的猜数字小游戏</br>
玩家输入猜数字的范围，然后心想一个数字让计算机猜。计算机会问一些问题，玩家必须诚实回答。计算机根据玩家的问题回答状况通过二分查找法的原理找出玩家心想的那个数字。</br>
注意：和二分查找法不同的是，判断过程是基于玩家的诚实回答，而不是基于计算机的判断。</br>
```
#! /usr/bin/python3

import time

def rangeTest():
    global start,end
    try:
        start=int(input('Please input the start number of the guessing range: '))
        end=int(input('Please input the end number of the guessing range: '))
        if start > end:
            raise Exception
    except:
        print('Please input two numbers and the start num should be smaller than the end num.')
        rangeTest()
    return start,end

def gameStartInfor(start,end):
    print('The guessing range is {} to {}.'.format(start,end))
    print('Please meditate a number in the above range in mind. ')
    print('The game will start 3 second later.')
    time.sleep(3)
    print('Game start: Please answer the following questions honestly!')
return

def createGuessRange(star,end):
    global guessRange
    guessRange=list(range(start,end+1))
    return guessRange

def binarySearch(guessRange):
    global mid
    rangeLength=len(guessRange)
    high=len(guessRange)-1
    low=0
      while True:
        mid=int((high+low)/2)
        ans=input('Is the number equals or smaller/bigger than {}?(e/s/b)'.format(guessRange[mid]))
        if ans == 'b':
            low=mid+1
        if ans == 's':
            high=mid-1
        if ans == 'e':
            break
    return guessRange[mid]

if __name__ == '__main__':
    rangeTest()
    gameStartInfor(start,end)
    createGuessRange(start,end)
    binarySearch(guessRange)
    print(guessRange[mid])


Please input the start number of the guessing range: 1
Please input the end number of the guessing range: 35469
The guessing range is 1 to 35469.
Please meditate a number in the above range in mind. 
The game will start 3 second later.
# 心想3249
Game start: Please answer the following questions honestly!
Is the number equals or smaller/bigger than 17735?(e/s/b)s
Is the number equals or smaller/bigger than 8867?(e/s/b)s
Is the number equals or smaller/bigger than 4433?(e/s/b)s
Is the number equals or smaller/bigger than 2216?(e/s/b)b
Is the number equals or smaller/bigger than 3324?(e/s/b)s
Is the number equals or smaller/bigger than 2770?(e/s/b)b
Is the number equals or smaller/bigger than 3047?(e/s/b)b
Is the number equals or smaller/bigger than 3185?(e/s/b)b
Is the number equals or smaller/bigger than 3254?(e/s/b)s
Is the number equals or smaller/bigger than 3219?(e/s/b)b
Is the number equals or smaller/bigger than 3236?(e/s/b)b
Is the number equals or smaller/bigger than 3245?(e/s/b)b
Is the number equals or smaller/bigger than 3249?(e/s/b)e
3249
```
