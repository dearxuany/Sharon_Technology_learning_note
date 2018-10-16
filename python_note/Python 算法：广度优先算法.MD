# Python 算法
## 广度优先算法 Breadth-First Search
### 广度优先算法作用
广度优先算法Breadth-First Search，简称BFS，是一种图形搜索演算法。</br>
广度优先算法指出了路径图中是否有从A到B的路径，如果有则可搜索出其最短路径（所经节点最少），即搜到到达目的节点的路径后即停止后续搜索。</br>

### 广度优先算法的运行时间
添加每个节点的运行时间 O(1) ，V个节点则是O(V)</br>
遍历每条路径的运行时间 O(1)，E条路径则是O(E)</br>
所以广度优先算法的运行时间为 O(V+E)</br>

### 广度优先算法实现过程
一个字典表示图的所有节点及各个节点间的关系；</br>
从数组中获取根节点，新建一个deque队列用于存放要和根节点相关的第一层关系对象；</br>
通过循环弹出各个节点，判断第一层节点中是否有目标对象；</br>
如果没有在第一层节点中找到目标对象，则在deque队列中添加与第一层节点相关的下一层节点，同时用空列表标记已检查过的节点以免造成无限循环；</br>
进入第二层节点的循环，先判断当前节点是否已检查过然后再判断节点是否为目标节点；</br>
节点层层叠加，直到找到目标对象为止，如果检查完所有节点后，没有发现目标对象，则判断为没有该路径。</br>

[关于python的双端队列](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E9%9B%86%E5%90%88%E3%80%81%E5%A0%86%E3%80%81%E5%8F%8C%E7%AB%AF%E9%98%9F%E5%88%97%E3%80%81%E9%98%9F%E5%88%97.MD#%E5%8F%8C%E7%AB%AF%E9%98%9F%E5%88%97%E5%8F%8A%E5%85%B6%E4%BB%96%E9%9B%86%E5%90%88)

### Python 广度优先算法实现
* 实例</br>
有个人员关系图，需要从Sharon找到不同的人并输出Sharon和目标人物的最亲近关系层数</br>

* 路径图的表示</br>
使用字典表示，字典的item无序，所以添加顺序不影响结果。</br>
```
>>> graph={}
>>> graph['Sharon']=['Tom','Jack']
>>> graph['Lily']=['Lisa','Sharon','Jack']
>>> graph['Tom']=['Jack','Sharon']
>>> graph['Jack']=['Lily','Sharon','Tom','Lala']
>>> graph['Lisa']=['Lily']
>>> graph['Lala']=['Jack']
>>> graph
{'Sharon': ['Tom', 'Jack'], 'Lily': ['Lisa', 'Sharon', 'Jack'], 'Tom': ['Jack', 'Sharon'], 'Jack': ['Lily', 'Sharon', 'Tom', 'Lala'], 'Lisa': ['Lily'], 'Lala': ['Jack']}
```
* 广度优先算法实现</br>
由于人物关系复杂，关系中可能会有环的出现，所以后续搜索时要注意搜索过的节点的去重，以免造成死循环。</br>
```
#! /usr/bin/python3

from collections import deque

def goalinput():
    global goalnode
    goalnode=input('Place input the goal node you want to find: ')

def breadthFirstSearch(firstNode):
    goalinput()
    waitToSearch=deque()  # 空队列用于存放节点
    waitToSearch += graph[firstNode] # 添加根节点，graph应为无序字典
    searched=[]  # 空列表用于存放已查找过的节点
    i=0   # 初始层数
    while waitToSearch:
        nodeWaitingSearch=waitToSearch.popleft()  # 弹出一个要搜索的元素
        if nodeWaitingSearch not in searched:
            if nodeWaitingSearch == goalnode:
                print('Found the goal-node!! i=',i)
                return True
            else:
                # 将与当前搜索节点相联系的节点添加到队列中
                waitToSearch += graph[nodeWaitingSearch]
                i=i+1
                # 将该已经搜索过的节点添加到列表中
                searched.append(nodeWaitingSearch)
    return False

breadthFirstSearch('Sharon')
```
```
Place input the goal node you want to find: Lala
Found the goal-node!! i= 4

Place input the goal node you want to find: Lily
Found the goal-node!! i= 3
```
