# 问题: 大黄鸭是鸭子吗?

- “当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。”
- 我们并不关心对象是什么类型，到底是不是鸭子，只关心行为。

### 鸭子类型在动态语言中经常使用，非常灵活

> 从duck typing，我们可以联想到它的推导，并不在乎类型的真正实体，只要他的行为有duck的特性，那么我们就可以把它当做一只duck来看到。在动态语言设计中，可以解释为无论一个对象是什么类型的，只要它具有某类型的行为（方法），则它就是这一类型的实例，而不在于它是否显示的实现或者继承。

#### Python实现

```python
class Duck:
    def info(self):
        print ("duck")    
        
class Bird:
    def info(self):
        print ("bird")
        
class Doge:
    def info(self):
        print ("doge")

def in_the_forest(mallard):
    mallard.info()

duck = Duck()
bird = Bird()
doge = Doge()

for x in [duck, bird, doge]:
    in_the_forest(x)st(x)
```

###### 输出

```shell
duck
bird
doge
```
