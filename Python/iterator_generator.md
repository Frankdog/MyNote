#iterator 
iterator 协议
class 定义
```python
__iter__()
next()
```


#generator
尾递归 yield

#decorator
**eg**
```python
def decorator(func):
    def wrapper(*arg,**kw):
        print "before decorator"
        return func(*arg,**kw)
    return wrapper

@decorator
def test():
    print "test"
```


#with
class 定义上下文管理器
```python
__enter__()
__exit_(self,exc_type,exc_value,traceback)
```
**eg**
```python
class prerequest(object):
    def __init__(self):
        pass
    def __enter__(self):
        print "__enter__"
    def __exit__(self,exc_type,exc_value,traceback):
        print "__exit__"

with prerequest():
    print "do something"

```



