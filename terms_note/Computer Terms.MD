# IT计算机领域常见术语
开这一篇笔记的原因：</br>
某些术语的相关知识比较零散，暂且不知应放置在何处；</br>
现阶段只需要了解部分相关知识，而暂且不需要深究；</br>
暂且没有空深究，先记个名词。</br>

## 递归
递归：函数自己调用自己</br>
对比循环：在某些情况下，循环性能更好；递归则在编码上更为清晰，易于理解。</br>
```
def rangeTest():
    global start,end
try: 
    #　基线条件：导致函数不再调用自己，防止无限循环
        start=int(input('Please input the start number of the guessing range: '))
        end=int(input('Please input the end number of the guessing range: '))
　　　　#　递归条件：导致函数自己调用自己
        if start > end:
            raise Exception
    except:  
        print('Please input two numbers and the start num should be smaller than the end num.')
        rangeTest()
return start,end
```
## 调用栈
在内存中存储多个函数的变量的栈，所有函数调用时它们的变量都会进入调用栈，先调用的函数的变量先压入，后调用的后压入，函数return后被弹出释放内存。</br>
调用栈可能很长，可能占用大量内存。比如上面输入检验的这种情况，如果我一直输入不符合要求的值，那么这个函数的调用栈就可能被堆得很高很高而导致栈溢出，此时应该考虑改写代码，可以考虑使用“尾递归”。</br>
