# Python 算法
## 二分法查找 Binary Search
时间复杂度：O(logN)</br>
缺点：数据必须是有序的</br>

### 二分法查找的 python3 实现</br>
给出列表和要找的数值，输出该数值在列表中的index。</br>
如果该元素在列表中存在，则输出index，跳出循环条件为目标数值和mid所指向的元素值相等；</br>
如果该元素在列表中不存在，则返回None，跳出循环条件为low大于high。</br>
```
#! /usr/bin/python3

def binarySearch(goalRange,goalNum):
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
            return mid
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

### 基于二分法查找的猜数字小游戏</br>
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

## 选择排序 Selection Sort
时间复杂度：O(n^2)</br>
缺点：慢</br>

### 选择排序的 python3 实现
利用选择排序来对列表中的元素进行以小到大的排序。</br>
先找出列表中的最小值，然后将其从原列表中弹出并插入到新列表，再从原列表中找出新的最小值，然后再次从原列表弹出这个新最小值插入到新列表末尾，循环以上操作直到所有元素迭代完成。</br>
注意：for n in range(0)时，即迭代次数为0，实际上没有进入到for循环当中。</br>
```
#! /usr/bin/python3

# 找出列表中的最小值
def findSmallest(lst):
    listLen=len(lst)
    smallest=lst[0]
    for n in range(len(lst)-1):
        if smallest > lst[n+1]:
            smallest=lst[n+1]
    return smallest

# 将最小值从原列表弹出并插入到新列表
def smallToBig(lst):
    lstNew=[]
    for i in range(len(lst)):
        smallest=findSmallest(lst)
        lstNew.append(lst.pop(lst.index(smallest)))
    return lstNew
```
将以上代码放入到自定义模块所在文件夹中
```
>>> import python3_algorithms_SelectionSort
>>> lst=[0,56,22,69,57,62,15,4878,366,36,3]
>>> python3_algorithms_SelectionSort.smallToBig(lst)
[0, 3, 15, 22, 36, 56, 57, 62, 69, 366, 4878]
```

## 快速排序 Quick sort
快速排序（Quicksort）是对冒泡排序的一种改进。</br>
基本思想：</br>
通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小；</br>
然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。</br>

### D&C 工作原理
1.最简单的基线条件；</br>
2.可以缩小问题规模使其符合基线条件的递归条件。</br>
对于数组排序来说，不用排序的有空数组和只有一个元素的数组，所以这个是排序的基线条件。

### 快速排序原理
1.基准值 pivot </br>
从数组中选一个元素值作为基准值，基准值的选择影响排序速度；</br>
2.分区 partitioning </br>
根据基准值可得三个无序分区，子数组内所有元素小于基准值、基准值、子数组内所有元素大于基准值；</br>
3.再次循环以上步骤，将子数组划分为更小的数组，直到所有元素被划分排序完毕</br>
4.合并多个子数组</br>
```
#! usr/bin/python3

def quicksort(array):
    # 基准条件
    if len(array) < 2 :
        return array
    else:
       # 基准值
       pivot = array[0]
       smaller=[x for x in array[1:] if x <= pivot]
       bigger=[x for x in array[1:] if x > pivot]
       # 递归
       return quicksort(smaller)+[pivot]+quicksort(bigger)
```
```
>>> from python3_algorithms_Quicksort import quicksort
>>> lst=[49,38,65,97,76,13,27,49]
>>> quicksort(lst)
[13, 27, 38, 49, 49, 65, 76, 97]
```
### 快速排序的运行时间
快排是最快的排序算法之一，基准值的选择影响排序速度：</br>
在最糟情况下，快速排序运行时间为O(n^2)，通常是序列有序而且把元素中的最大值/最小值作为pivot的时候发生；</br>
平均状况下，快速排序的运行时间为O(n log n)，每次选的pivot都刚好是序列元素的中值；</br>
由于大O表示法的常量问题，快排的常量比合并排序小，所以尽管运行时间是一样的，但平均状况下快排会比合并排序快。</br>
