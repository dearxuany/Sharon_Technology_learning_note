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

## 狄克斯特拉算法 Dijkstra's Algorithm
狄克斯特拉算法是加权图中找出一个节点到其余各节点的最短路径（花销最少）算法，解决的是单向图中最短路径问题，主要特点是以起始点为中心向外层层扩展，直到扩展到终点为止。</br>
对比广度优先算法，广度优先算法找最短路径的评判标准是节点到节点间跳数最少，但是没有计算其每一跳间的花销，而狄克斯特拉算法会计算花销最少的路径，花销最少的路径不一定是跳数最少的那一条。广度优先算法找到目的节点就不会再往下搜索，而狄克斯特拉算法会搜索所有节点。广度优先算法用于非加权图的最短路径计算，狄克斯特拉算法用于无负权边的加权图的最少花销路径计算。</br>
注意：Dijkstra's Algorithm 不能用于包含负权边的图，如果要计算有负权边的图应该使用 Bellman-Ford算法。</br>

### 加权图的表示
```
# 使用嵌套的字典表示整个加权图
>>> graph={}
>>> graph
{}
# 起点到a或b
>>> graph['start']={}
>>> graph['start']['a']=6
>>> graph['start']['b']=2
>>> graph
{'start': {'a': 6, 'b': 2}}
# a到终点
>>> graph['a']={}
>>> graph['a']['end']=1
# b到a或终点
>>> graph['b']={}
>>> graph['b']['a']=3
>>> graph['b']['end']=5
>>> graph
{'start': {'a': 6, 'b': 2}, 'a': {'end': 1}, 'b': {'a': 3, 'end': 5}}
# 终点也是需要的，只是没有后续的点了
>>> graph['end']={}
>>> graph
{'start': {'a': 6, 'b': 2}, 'a': {'end': 1}, 'b': {'a': 3, 'end': 5}, 'end': {}}
```
### 总花销表示
用字典表示从起点到各节点的总花销，开头的时候只有和起点直连的点是能确定总花销的，其余不直连的点用infinity表示。
```
>>> costs={}
# a和b与起点直连，所以有初始值
>>> costs['a']=6
>>> costs['b']=2
# 终点和起点不直连，先用inf表示，以便以后更新
>>> costs['end']=float('inf')
>>> costs
{'a': 6, 'b': 2, 'end': inf}
```

### 父节点表示
用字典表示当前节点的上一节点，一开始也是只有和起点直连的节点可以确定父节点是起点，其余节点都是暂且无法确定的，所以暂且用None表示待以后更新。如果一个节点和起点直连，同时可以由其他节点到达，此时也只写父节点为起点，因为起点是必经之地，之后再根据花销更新此节点的父节点。
```
>>> parentNode['a']='start'
>>> parentNode['b']='start'
>>> parentNode['end']=None
>>> parentNode
{'a': 'start', 'b': 'start', 'end': None}
```
### 算法逻辑
根据花销找与当前节点最近的直连节点；</br>
将从当前节点到最近直连节点的花销添加到总花销 costs 中；</br>
将最近直连节点作为父节点更新到parentNode中；</br>
标记处理过的节点；</br>
根据上述过程遍历所有花销最小的节点，直到到达终点，得出从起点到终点的花销最少的路径。</br>
```
#! /usr/bin/python3

def find_the_smallest_cost_neighbour_node(costs):
    # 初始化最小花费及花费最小的节点
    smallestCost=float('inf')
    smallestCostNode=None
    # 找出从起点到当前节点的最少花费
    for node in costs:
        cost = costs[node]
        # 之前标记过的最近邻近节点不会再被考虑
        if cost < smallestCost and node not in searched:
            smallestCost=cost
            smallestCostNode=node
    return smallestCostNode

def dijkstra_Algorithm(graph,parentNode,costs):
    global searched
    searched=[] # 存放已处理的节点
    # 找与起点直连的最近邻居
    costSmallestNeighbour=find_the_smallest_cost_neighbour_node(costs)
    while costSmallestNeighbour is not None:
        cost=costs[costSmallestNeighbour]  # 由起点到最近邻近节点的花费
        neighbourCost=graph[costSmallestNeighbour]  #从起点的邻近节点到其相邻节点的距离
        # 更新由起点经最近邻居去各节点的总花费
        for n in neighbourCost.keys():
            newCost = cost + neighbourCost[n]
            if costs[n]> newCost:
                costs[n]=newCost
                parentNode[n]=costSmallestNeighbour
        # 将最小邻近节点加入到以搜索节点中
        searched.append(costSmallestNeighbour)
        # 重新找从起点经之前找到的邻近节点到各点的最短路径，直到找到所有花费最小的点
        costSmallestNeighbour=find_the_smallest_cost_neighbour_node(costs)
     return costs,parentNode

if __name__ == '__main__':
    graph={'start': {'a': 6, 'b': 2}, 'a': {'end': 1}, 'b': {'a': 3, 'end': 5}, 'end': {}}
    costs={'a': 6, 'b': 2, 'end': float('inf')}
    parentNode={'a': 'start', 'b': 'start', 'end': None}
print(dijkstra_Algorithm(graph,parentNode,costs))
```
```
# 直接作为主程序运行输出结果
({'a': 5, 'b': 2, 'end': 6}, {'a': 'b', 'b': 'start', 'end': 'a'})
# 作为模块导入输出
>>> graph={'start': {'a': 6, 'b': 2}, 'a': {'end': 1}, 'b': {'a': 3, 'end': 5}, 'end': {}}
>>> costs={'a': 6, 'b': 2, 'end': float('inf')}
>>> parentNode={'a': 'start', 'b': 'start', 'end': None}
>>> dijkstra_Algorithm(graph,parentNode,costs)
({'a': 5, 'b': 2, 'end': 6}, {'a': 'b', 'b': 'start', 'end': 'a'})
# 作为模块导入计算另外一个加权图
>>> graph
{'start': {'a': 5, 'b': 2}, 'a': {'c': 2, 'd': 4}, 'b': {'a': 8, 'c': 7}, 'c': {'end': 1}, 'd': {'end': 3, 'c': 6}, 'end': {}}
>>> costs
{'a': 5, 'b': 2, 'c': inf, 'd': inf, 'end': inf}
>>> parentNode
{'a': 'start', 'b': 'start', 'c': None, 'd': None, 'end': None}
>>> dijkstra_Algorithm(graph,parentNode,costs)
({'a': 5, 'b': 2, 'c': 7, 'd': 9, 'end': 8}, {'a': 'start', 'b': 'start', 'c': 'a', 'd': 'a', 'end': 'c'})
# 可见花销最小的路径是 start-a-c-end 花销为8
```

