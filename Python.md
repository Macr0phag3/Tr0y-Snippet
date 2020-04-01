# python-snippets

注：
1. snippets 本着越少代码越好的方式来写的，有些写法并不推荐
2. snippets 不保证性能，有需要可以自行测试
3. snippets 在 `3.7.3` 下完成，低于此版本有可能无法使用，请自行测试

## print 加颜色
简单，但是 Windows 不适用：

```
def put_color(string, color, bold=False):
    bold = [int(bold), ['2', '2;1'][bold]][color == 'gray']

    colors = {
        'red': '31',
        'green': '32',
        'yellow': '33',

        'blue': '34',
        'pink': '35',
        'cyan': '36',
        'gray': '37',
        'white': '37',
    }

    return '\033[40;{};{};40m{}\033[0m'.format(
        bold, colors.get(color, '37'),
        str(string)
    )

```

需要安装 `colorama`，但是 Windows 适用：
```
from colorama import Fore, Style

def put_color(string, color, bold=True):
    '''
    give me some color to see :P
    '''

    if color == 'gray':
        COLOR = Style.DIM + Fore.WHITE
    else:
        COLOR = getattr(Fore, color.upper(), Fore.WHITE)

    return f'{Style.BRIGHT if bold else ""}{COLOR}{str(string)}{Style.RESET_ALL}'

```

## 将 n 秒转为合适时间单位
```
from itertools import accumulate
from operator import truediv, mul


def timer_unit(s):
    d = [(i, u) for i, u in zip(accumulate([s, 60, 60, 24, 30, 12], truediv), 'smhDMY') if i >= 1]
    m = [(i, u) for i, u in zip(accumulate([s, *[1000]*2], mul), 's ms us'.split()) if i >= 1]

    return [*d[::-1], *m, (s, 'us')][0]
    # return num, unit

```

```
In [6]: timer_unit(0.0001)
Out[6]: (100.0, 'us')

In [7]: timer_unit(0.01)
Out[7]: (10.0, 'ms')

In [8]: timer_unit(1)
Out[8]: (1, 's')

In [9]: timer_unit(60)
Out[9]: (1.0, 'm')

In [10]: timer_unit(60*60)
Out[10]: (1.0, 'h')

In [11]: timer_unit(60*60*24)
Out[11]: (1.0, 'D')

In [12]: timer_unit(60*60*24*30)
Out[12]: (1.0, 'M')

In [13]: timer_unit(60*60*24*30*12)
Out[13]: (1.0, 'Y')
```

## 将 n bytes 转为合适的容量单位
```
from operator import truediv
from itertools import accumulate


def cap_unit(b):
    d = [(i, u) for i, u in zip(accumulate([b, *[1024]*4], truediv), 'bkmgt') if i >= 1]

    return [*d[::-1], (b, 'b')][0]
    # return num, unit

```

## 类 json 格式的字典计数
```
def jdict_counter(jdict, kwd):
    count = {}
    for i in jdict:
        count[i[kwd]] = count.setdefault(i[kwd], 0) + 1

    return count
```

```
In [1]: a = [{'name': '1', 'score': 60}, {'name': '1', 'score': 59}, {'name': '1', 'score': 100}, {'name': '2', 'score': 10}, {'name': 'b', 'score': 100}, {'name': 'c', 'score': 0}]
jdict_counter(a)


In [2]: def jdict_counter(jdict, kwd):
     ...:     count = {}
     ...:     for i in jdict:
     ...:         count[i[kwd]] = count.setdefault(i[kwd], 0) + 1
     ...:
     ...:     return count
     ...:

In [3]: jdict_counter(a, 'name')
Out[3]: {'1': 3, '2': 1, 'b': 1, 'c': 1}
```

## 美化 json/dict
```
import ast
import json

def json_beautify(data):
    if isinstance(data, str):
        data = ast.literal_eval(data)

    return json.dumps(data, indent=2)

```

```
In [18]: print(json_beautify("{'a': 1, 'b': 2}"))
{
  "a": 1,
  "b": 2
}

In [19]: print(json_beautify("{\"a\": 1, \"b\": 2}"))
{
  "a": 1,
  "b": 2
}

In [20]: print(json_beautify("{'a': 1, 'b': 2}"))
{
  "a": 1,
  "b": 2
}

In [21]: print(json_beautify("{'a': 1, \"b\": 2}"))
{
  "a": 1,
  "b": 2
}

In [22]: print(json_beautify({'a': 1, 'b': 2}))
{
  "a": 1,
  "b": 2
}

In [23]: print(json_beautify({'a': 1, "b": 2}))
{
  "a": 1,
  "b": 2
}

In [24]: print(json_beautify({'a': 1, 'b': 2, 'c': {'a': 1, 'b': 2, 'c': [1, 2, 3, 4, 5]}}))
{
  "a": 1,
  "b": 2,
  "c": {
    "a": 1,
    "b": 2,
    "c": [
      1,
      2,
      3,
      4,
      5
    ]
  }
}
```

## 运行时代码时进入交互式 debug 模式
`__import__('code').interact(local=dict(globals(), **locals()))`

插入代码中，然后 `python xxx.py` 的过程中会出现提示：
```
» python check_new.py
[+] Get data from es
  [-] count of rules: 32

[+] Get data from mongo
  [-] count of rules: 516

[*] New rules
  [1] hello, world!
Python 3.7.3 (default, Jun  8 2019, 00:01:18)
[Clang 10.0.1 (clang-1001.0.46.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> db
Database(MongoClient(host=['127.0.0.1:27017'], document_class=dict, tz_aware=False, connect=True), 'test')
>>>
```

## 字符串转 requests 代码
```

class string2requests:
    def __init__(self, string):
        self.requests = string.strip().split("\n")
        self._params = {}
        self._data = {}
        self._headers = {}
        self._url = ''

    def _get_headers(self):
        if self.requests[-2] == '':  # 有 post 数据
            headers = self.requests[1:-2]
            [
                self._data.update(dict([i.split('=')])) for i in self.requests[-1].split('&') if '=' in i and i[-1] != '='
            ]

        else:
            headers = self.requests[1:]
            if self._method == 'post':
                print("# [WARN] Method is POST but post data is empty!\n")

        if self._method == 'get':
            self._url = headers[0].split(': ')[-1]

        [
            self._headers.update(dict([i.split(': ')])) for i in '&'.join(headers).split('&') if ': ' in i and i[-1] != ': '
        ]
        return self._headers

    def _get_method(self):
        self._method = self.requests[0].split(' ')[0].lower()
        return self._method

    def _get_url(self):
        url = self.requests[0].split(' ')[1]
        if '?' in url:
            _url = url[:url.index('?')]
            [
                self._params.update(dict([i.split('=')])) for i in url[url.index('?') + 1:].split('&') if '=' in i and i[-1] != '='
            ]
        else:
            _url = url

        self._url += _url
        return self._url

    def __str__(self):
        return '\n'.join(self.requests)

    def _format(self, param):
        return str(param).replace(', \'', ',\n    \'').replace('{', '{\n    ').replace('}', '\n}')

    def parse(self):
        self._method = self._get_method()
        self._get_headers()
        self._url = self._get_url()

        template = 'import requests\n\nresponse = requests.{}(\n"{}", \nparams={}, \ndata={}, \nheaders={})'.format(
            self._method,
            self._url,
            self._format(self._params),
            self._format(self._data),
            self._format(self._headers),
        )

        return template

```
示例：

```
In [9]: raw = '''
   ...: GET / HTTP/1.1
   ...: Host: baidu.com
   ...: User-Agent: curl/7.54.0
   ...: Accept: */*
   ...: '''
   ...:

In [10]: print(string2requests(raw).parse())
import requests

response = requests.get(
"baidu.com/",
params={

},
data={

},
headers={
    'Host': 'baidu.com',
    'User-Agent': 'curl/7.54.0',
    'Accept': '*/*'
})
```
