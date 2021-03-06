目标：将URL拆分为组件

urllib.parse模块提供了操作URL及其组成部分的功能，可以将它们分解或组装起来。

##解析
urlparse()函数返回一个ParseResult对象，它类似一个包含6个元素的元组。
```python
# urllib_parse_urlparse.py
from urllib.parse import urlparse

url = 'http://netloc/path;param?query=arg#frag'
parsed = urlparse(url)
print(parsed)
```
通过元组接口得到的URL部分分别是机制、网络位置、路径、路径段参数（有一个分号与路径分开）、查询和片段。
```bash
$ python3 urllib_parse_urlparse.py

ParseResult(scheme='http', netloc='netloc', path='/path',
params='param', query='query=arg', fragment='frag')
```
尽管返回值像是一个元组，实际上它基于命名元组，这是一种支持通过下标索引和名字属性去访问URL各个部分的元组派生类。属性API不仅对开发人员来说易于使用，还允许访问tuple API未提供的很多值。
```python
# urllib_parse_urlparseattrs.py
from urllib.parse import urlparse

url = 'http://user:pwd@NetLoc:80/path;param?query=arg#frag'
parsed = urlparse(url)
print('scheme  :', parsed.scheme)
print('netloc  :', parsed.netloc)
print('path    :', parsed.path)
print('params  :', parsed.params)
print('query   :', parsed.query)
print('fragment:', parsed.fragment)
print('username:', parsed.username)
print('password:', parsed.password)
print('hostname:', parsed.hostname)
print('port    :', parsed.port)
```
当URL中有用户名和密码时，则它们是有效的，不设置它们则获取到None。主机名(hostname)即是网络位置(netloc)，全为小写并带有一个端口号。端口号会自动转换为一个整数值，如果没有则为None。
```bash
$ python3 urllib_parse_urlparseattrs.py

scheme  : http
netloc  : user:pwd@NetLoc:80
path    : /path
params  : param
query   : query=arg
fragment: frag
username: user
password: pwd
hostname: netloc
port    : 80
```
urlsplit()函数是urlparse()的变体。它的行为与urlparse()稍微有些不同，因为它不会从URL分解参数。这对于遵循RFC 2396的URL很有用，它支持对应路径的每一段参数。
```python
# urllib_parse_urlsplit.py
from urllib.parse import urlsplit

url = 'http://user:pwd@NetLoc:80/p1;para/p2;para?query=arg#frag'
parsed = urlsplit(url)
print(parsed)
print('scheme  :', parsed.scheme)
print('netloc  :', parsed.netloc)
print('path    :', parsed.path)
print('query   :', parsed.query)
print('fragment:', parsed.fragment)
print('username:', parsed.username)
print('password:', parsed.password)
print('hostname:', parsed.hostname)
print('port    :', parsed.port)
```
由于没有分解参数，元组API会显示5个参数而不是6个，这里没有params属性。
```bash
$ python3 urllib_parse_urlsplit.py

SplitResult(scheme='http', netloc='user:pwd@NetLoc:80',
path='/p1;para/p2;para', query='query=arg', fragment='frag')
scheme  : http
netloc  : user:pwd@NetLoc:80
path    : /p1;para/p2;para
query   : query=arg
fragment: frag
username: user
password: pwd
hostname: netloc
port    : 80
```
要从一个URL中剥离出片段标识符，如从一个URL中查找基页面名，可以使用urldefrag()。
```python
# urllib_parse_urldefrag.py
from urllib.parse import urldefrag

original = 'http://netloc/path;param?query=arg#frag'
print('original:', original)
d = urldefrag(original)
print('url     :', d.url)
print('fragment:', d.fragment)
```
返回值是一个基于命名元组的DefragResult类对象，包括URL基址和片段标识符。
```bash
$ python3 urllib_parse_urldefrag.py

original: http://netloc/path;param?query=arg#frag
url     : http://netloc/path;param?query=arg
fragment: frag
```
##反解析
有几种方法可以将各个分离的URL部分重新汇总到一个字符串中。用于解析的URL对象有一个geturl()方法。
```python
# urllib_parse_geturl.py
from urllib.parse import urlparse

original = 'http://netloc/path;param?query=arg#frag'
print('ORIG  :', original)
parsed = urlparse(original)
print('PARSED:', parsed.geturl())
```
geturl()只能处理urlparse()和urlsplit()返回的对象。
```bash
$ python3 urllib_parse_geturl.py

ORIG  : http://netloc/path;param?query=arg#frag
PARSED: http://netloc/path;param?query=arg#frag
```
一个包含字符串的常规元组可以使用urlunparse()组合成一个URL。
```python
# urllib_parse_urlunparse.py
from urllib.parse import urlparse, urlunparse

original = 'http://netloc/path;param?query=arg#frag'
print('ORIG  :', original)
parsed = urlparse(original)
print('PARSED:', type(parsed), parsed)
t = parsed[:]
print('TUPLE :', type(t), t)
print('NEW   :', urlunparse(t))
```
因为urlparse()返回的ParseResult类型对象可以当成一个元组使用，在上例中，我们显式地创建了一个新的元组来说明urlunparse()也能够处理正常的元组。
```bash
$ python3 urllib_parse_urlunparse.py

ORIG  : http://netloc/path;param?query=arg#frag
PARSED: <class 'urllib.parse.ParseResult'>
ParseResult(scheme='http', netloc='netloc', path='/path',
params='param', query='query=arg', fragment='frag')
TUPLE : <class 'tuple'> ('http', 'netloc', '/path', 'param',
'query=arg', 'frag')
NEW   : http://netloc/path;param?query=arg#frag
```
如果输入的URL包含多余的部分，在重建URL时会将这些部分舍弃。
```python
# urllib_parse_urlunparseextra.py
from urllib.parse import urlparse, urlunparse

original = 'http://netloc/path;?#'
print('ORIG  :', original)
parsed = urlparse(original)
print('PARSED:', type(parsed), parsed)
t = parsed[:]
print('TUPLE :', type(t), t)
print('NEW   :', urlunparse(t))
```
在上例中，原始URL中没有参数，查询和片段。新的url和原始url看起来不一起，但是它符合标准。
```bash
$ python3 urllib_parse_urlunparseextra.py

ORIG  : http://netloc/path;?#
PARSED: <class 'urllib.parse.ParseResult'>
ParseResult(scheme='http', netloc='netloc', path='/path',
params='', query='', fragment='')
TUPLE : <class 'tuple'> ('http', 'netloc', '/path', '', '', '')
NEW   : http://netloc/path
```
##连接
除了解析URL外，urlparse还包括urljoin()方法，可以有相对片段构造绝对URL。
```python
# urllib_parse_urljoin.py
from urllib.parse import urljoin

print(urljoin('http://www.example.com/path/file.html',
              'anotherfile.html'))
print(urljoin('http://www.example.com/path/file.html',
              '../anotherfile.html'))
```
在上例中，当计算第二个URL时，路径中的相对部分(“../”)会被自动处理。
```bash
$ python3 urllib_parse_urljoin.py

http://www.example.com/path/anotherfile.html
http://www.example.com/anotherfile.html
```
非相对路径的处理与os.path.join()的处理方式相同。
```python
# urllib_parse_urljoin_with_path.py
from urllib.parse import urljoin

print(urljoin('http://www.example.com/path/',
              '/subpath/file.html'))
print(urljoin('http://www.example.com/path/',
              'subpath/file.html'))
```
如果要连接到URL的路径以斜杠(/)开头，则URL的路径会被重置到顶级路径。相反，如果没有以斜杠(/)开头，则路径会自动添加到URL路径的后面。
```bash
$ python3 urllib_parse_urljoin_with_path.py

http://www.example.com/subpath/file.html
http://www.example.com/path/subpath/file.html
```
##编码查询参数
参数在添加到URL之前，需要编码这些参数。
```python
# urllib_parse_urlencode.py
from urllib.parse import urlencode

query_args = {
    'q': 'query string',
    'foo': 'bar',
}
encoded_args = urlencode(query_args)
print('Encoded:', encoded_args)
```
编码会替换掉一些特殊字符（如，空格）以确保传给服务器的参数是符合标准的。
```bash
$ python3 urllib_parse_urlencode.py

Encoded: q=query+string&foo=bar
```
要在查询字符串中使用含有单独变量的序列值，应该在调用urlencode()时将doseq设置为True。
```python
# urllib_parse_urlencode_doseq.py
from urllib.parse import urlencode

query_args = {
    'foo': ['foo1', 'foo2'],
}
print('Single  :', urlencode(query_args))
print('Sequence:', urlencode(query_args, doseq=True))
```
结果是一个查询字符串，其中几个值与同一个名称相关联。
```bash
$ python3 urllib_parse_urlencode_doseq.py

Single  : foo=%5B%27foo1%27%2C+%27foo2%27%5D
Sequence: foo=foo1&foo=foo2
```
解码查询字符串，可以使用parse_qs()或者parse_qsl()。
```python
# urllib_parse_parse_qs.py
from urllib.parse import parse_qs, parse_qsl

encoded = 'foo=foo1&foo=foo2'

print('parse_qs :', parse_qs(encoded))
print('parse_qsl:', parse_qsl(encoded))
```
parse_qs()的返回值是一个名字与值相匹配的字典，parse_qsl()的返回值是一个包含元组的列表，每一个元组包括了名字与值。
```bash
$ python3 urllib_parse_parse_qs.py

parse_qs : {'foo': ['foo1', 'foo2']}
parse_qsl: [('foo', 'foo1'), ('foo', 'foo2')]
```
在传递参数给urlencode()时，URL中可能会引起服务端解析错误的查询参数中的特殊字符会被“引用”起来。在本地引用它们并制作安全版本的字符串，可以直接使用quote()或quote_plus()函数。
```python
# urllib_parse_quote.py
from urllib.parse import quote, quote_plus, urlencode

url = 'http://localhost:8080/~hellmann/'
print('urlencode() :', urlencode({'url': url}))
print('quote()     :', quote(url))
print('quote_plus():', quote_plus(url))
```
quote_plus()中引用的实现对它所替换的字符更具攻击性。
```bash
$ python3 urllib_parse_quote.py

urlencode() : url=http%3A%2F%2Flocalhost%3A8080%2F%7Ehellmann%2F
quote()     : http%3A//localhost%3A8080/%7Ehellmann/
quote_plus(): http%3A%2F%2Flocalhost%3A8080%2F%7Ehellmann%2F
```
对引用操作进行反操作，可以使用unquote()或unquote_plus()。
```python
# urllib_parse_unquote.py
from urllib.parse import unquote, unquote_plus

print(unquote('http%3A//localhost%3A8080/%7Ehellmann/'))
print(unquote_plus(
    'http%3A%2F%2Flocalhost%3A8080%2F%7Ehellmann%2F'
))
```
这样已编码的值就会被转换为正常的字符串URL。
```bash
$ python3 urllib_parse_unquote.py

http://localhost:8080/~hellmann/
http://localhost:8080/~hellmann/
```












