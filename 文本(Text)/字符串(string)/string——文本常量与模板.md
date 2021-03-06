目的：包含用于处理文本的常量和类
string模块是从最早期版本的Python开始存在的。很多早期在string模块中实现的功能现在已经移植到了str对象方法中。目前string模块保留了几个在处理str对象时很有用的常量和类。下面我们将集中讨论这几个常量和类。
##函数
capwords()函数能将一个字符串中每一个单词的首字母大写。
```python
# string_capwords.py

import string

s = 'The quick brown fox jumped over the lazy dog.'

print(s)
print(string.capwords(s))
```
输出的结果就像是先调用split()获得分词结果，然后再将返回列表中的每一个单词大写，最后再使用join()方法将结果组合起来。
```bash
$ python3 string_capwords.py

The quick brown fox jumped over the lazy dog.
The Quick Brown Fox Jumped Over The Lazy Dog.
```
##模板
字符串模板是PEP 292规范的一部分，并替代原来的内置插值语法。使用string.Template插值，前缀添加$字符(如$var)将被识别为一个变量。另外，如果需要将变量与其他文本区分开来，可以使用花括号(如${var})。
下面这个例子对字符串进行了比较，一个是使用了%操作符的相似字符串插值的字符串模板，另一个是使用str.format()的新的格式化字符串语法。
```python
# string_template.py
import string

values = {'var': 'foo'}

t = string.Template("""
Variable        : $var
Escape          : $$
Variable in text: ${var}iable
""")

print('TEMPLATE:', t.substitute(values))

s = """
Variable        : %(var)s
Escape          : %%
Variable in text: %(var)siable
"""

print('INTERPOLATION:', s % values)

s = """
Variable        : {var}
Escape          : {{}}
Variable in text: {var}iable
"""

print('FORMAT:', s.format(**values))
```

在上面的示例程序中，前两种情况的触发字符($或%)都需要连续输入两次进行转义。对于format的语法，左花括号{和右花括号}都需要重复两次去转义。
```bash
$ python3 string_template.py

TEMPLATE:
Variable        : foo
Escape          : $
Variable in text: fooiable

INTERPOLATION:
Variable        : foo
Escape          : %
Variable in text: fooiable

FORMAT:
Variable        : foo
Escape          : {}
Variable in text: fooiable
```
模板和字符串插值或format语法一个关键的不同点在于无需考虑参数的类型。输入的值会转换成字符串值，然后将转换后的字符串插入输出结果中。没有格式化选项可用。例如，没有办法控制用于表示浮点数值的位数。
但是，有一个好处是，使用safe_substitute()方法可以避免，如果不是模板所需的全部值作为参数提供的异常。
```python
# string_template_missing.py
import string

values = {'var': 'foo'}

t = string.Template("$var is here but $missing is not provided")

try:
    print('substitute()     :', t.substitute(values))
except KeyError as err:
    print('ERROR:', str(err))

print('safe_substitute():', t.safe_substitute(values))
```
因为在传入字典中没有missing这个键对应的值，所以substitute()抛出了KeyError。使用safe_substitute()的话，它会捕捉到这个错误，并让变量依然保留在文本中，而不是抛出错误。
```bash
$ python3 string_template_missing.py

ERROR: 'missing'
safe_substitute(): foo is here but $missing is not provided
```
##高级模板
string.Template的默认语法是可以修改的，可以调整正则表达式在模板正文中用于寻找变量名的匹配模式。
```python
# string_template_advanced.py
import string


class MyTemplate(string.Template):
    delimiter = '%'
    idpattern = '[a-z]+_[a-z]+'


template_text = '''
  Delimiter : %%
  Replaced  : %with_underscore
  Ignored   : %notunderscored
'''

d = {
    'with_underscore': 'replaced',
    'notunderscored': 'not replaced',
}

t = MyTemplate(template_text)
print('Modified ID pattern:')
print(t.safe_substitute(d))
```
在上面的示例中，替换规则已经发生了变化，所以分隔符变成了%而不是$，变量名中间的某个地方必须包含有下划线。因为%notunderscored没有下划线字符，所以它不会被替换掉。
```bash
$ python3 string_template_advanced.py

Modified ID pattern:

  Delimiter : %
  Replaced  : replaced
  Ignored   : %notunderscored
```
对于更复杂的更改，可以重写模式参数(pattern attribute)和定义一个全新的正则表达式。新的模式必须包含四个命名组，用于捕获转义的分隔符，命名变量，支撑版本的变量名，以及无效的分隔符模式。
```python
# string_template_defaultpattern.py
import string

t = string.Template('$var')
print(t.pattern.pattern)
```
t.pattern的值是一个已编译的正则表达式，原始字符串对于它的模式属性是有效的。
```bash
\$(?:
  (?P<escaped>\$) |                # two delimiters
  (?P<named>[_a-z][_a-z0-9]*)    | # identifier
  {(?P<braced>[_a-z][_a-z0-9]*)} | # braced identifier
  (?P<invalid>)                    # ill-formed delimiter exprs
)
```
下面的例子定义了一种可以创造新模板类型的新模式，使用`{{var}}`作为可变语法。
```python
# string_template_newsyntax.py
import re
import string


class MyTemplate(string.Template):
    delimiter = '{{'
    pattern = r'''
    \{\{(?:
    (?P<escaped>\{\{)|
    (?P<named>[_a-z][_a-z0-9]*)\}\}|
    (?P<braced>[_a-z][_a-z0-9]*)\}\}|
    (?P<invalid>)
    )
    '''


t = MyTemplate('''
{{{{
{{var}}
''')

print('MATCHES:', t.pattern.findall(t.template))
print('SUBSTITUTED:', t.safe_substitute(var='replacement'))
```
命名和支撑的模式必须提前分别给出，即使它们是相同的值。运行上面程序会得到以下输出：
```bash
$ python3 string_template_newsyntax.py

MATCHES: [('{{', '', '', ''), ('', 'var', '', '')]
SUBSTITUTED:
{{
replacement
```
##格式化
Formatter类实现了和str中的format()方法相同的布局规范语言。它的功能包括类型强制转换，对齐，属性和字段引用，模板参数支持命名参数和位置参数，以及类型特定的格式化选项。在大多数时候，format()方法能更方便地使用这些功能，Formatter类常用于在需要变体的情况下构建子类。
##常量
string模块包含许多与ASCII和数字字符集相关的常量。
```python
# string_constants.py
import inspect
import string


def is_str(value):
    return isinstance(value, str)


for name, value in inspect.getmembers(string, is_str):
    if name.startswith('_'):
        continue
    print('%s=%r\n' % (name, value))
```
这些常量在处理ASCII数据的时候十分有用，但是由于现在在Unicode格式文本中遇到非ASCII数据的情况越来越多，所以它的应用的有限制的。
```bash
$ python3 string_constants.py

ascii_letters='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVW
XYZ'

ascii_lowercase='abcdefghijklmnopqrstuvwxyz'

ascii_uppercase='ABCDEFGHIJKLMNOPQRSTUVWXYZ'

digits='0123456789'

hexdigits='0123456789abcdefABCDEF'

octdigits='01234567'

printable='0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQ
RSTUVWXYZ!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~ \t\n\r\x0b\x0c'

punctuation='!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'

whitespace=' \t\n\r\x0b\x0c'
```



























































































































