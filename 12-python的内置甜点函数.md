<!--markdown-->#### enumerate
{% highlight python linenos %}
>>> l = ['a', 'b', 12, 'd']
>>> for i, d in enumerate(l):
...     print(i, d)
...
0 a
1 b
2 12
3 d
{% endhighlight %}

#### zip
{%highlight python linenos %}
>>> l = ['a', 'b', 'c', 'd']
>>> n = ['1', '2', '3', '4']
>>> for i, m in zip(l, n):
...     print(i, m)
...
a 1
b 2
c 3
d 4
{%endhighlight%}

#### splitlines, strip, rstrip
`str.splitlines(keepends)`：按照行（\\r,\\n,\\r\\n）分隔字符串,返回一个列表。如果参数keepends为False，不保留换行符，如果为True，则保留换行符。

`str.strip(chars)`：移除字符串**首尾**指定的字符，默认为空格。

`str.rstip(chars)`：移除字符串**末尾**指定的字符，默认为空格。

#### str.format
'<': Forces the field to be left-aligned within the available space (this is the default for most objects).

'>':  \	Forces the field to be right-aligned within the available space (this is the default for numbers).

#### f-string
python3.6开始支持此特性
```python
name = 'root'
age = 12
f'im {name}' # im root
F'im {name}, im {age}' # im root, im 12
```
#### 性能测试
使用**timeit**模块

### 类型标注

```
def greeting(name: str) -> str:
    return "Hello" + name
```
这个函数的输入被标注为是字符串(str)，返回值也被标注为字符串。

[官方doc](https://docs.python.org/3.1/library/string.html)	