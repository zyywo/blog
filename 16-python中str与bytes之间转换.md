<!--markdown-->python中字符串(str)与字节(bytes)之间的转换方法

{%highlight python linenos%}

b = b'example'
s =  'example'

# str to bytes
bytes(s, encoding='utf8')
str.encode(s)

# bytes to str
str(b, encoding='utf8')
bytes.decode(b)

{%endhighlight%}	