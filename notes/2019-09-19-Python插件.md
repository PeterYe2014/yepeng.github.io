# Python一些重要的插件和特殊的功能
### __future__模块

大家都晓得python2.x与python3.x 版本有较大的语法差距，如何将最新版本的语法向下兼容呢，这里就要使用 __future__ 模块了。

通过 unicode_literals 让python2.x支持python 3.0中的字符串表示方式（python3.x 中把字节串和字符串分开了，分别是不同的类型：bytes, str ）

```python

# python 2.x 
'abc' => str
u'abc' => unicode

# python 3.x
'abc'=>unicode
b'abc'=>str

from __future import unicode_literals
# 后面都采用python3.x 的规则

```
 division 模块可以支持python3.x的除法语法
 print_function 修改print函数的表现
absolute_import 多行import的语法、绝对import和相对import

 ```python

# python 2.x
 
 10 / 3 # 地板除法，向下取整
 
>>> print ("Hello", "world")
('Hello', 'world')
 
# python 3.x

10 / 3 # 精确除法
10 // 3 # 地板除法 

>>> print ("Hello", "world")
Hello world
 ```

### hashLib——常用的摘要算法
常用的摘要算法有：
 sha1(), sha224(), sha256(), sha384(), sha512(), blake2b(), and blake2s(). md5()
 比较新的有：
sha3_224(), sha3_256(), sha3_384(), sha3_512(), shake_128(), shake_256().

```python
import hashlib
m = hashlib.sha256()
m.update(b"Nobody inspects") // 注意接受编码后的字符串，str 要用 encode方法
m.digest();
m.hexdigest();
```

或者另一种调用方法:

```python
h = hashlib.new('ripemd160') 
h.update(b"Nobody inspects the spammish repetition")
h.hexdigest()
```
多次update和一次update计算的摘要值是不一样的