# study-note-1-
第一次记录自己学习过程中遇到的问题，并记录与分享。 《python网络数据采集》第95页，提到了将数据清洗后进行序列统计排序的问题，但未给出代码，并且他以字典方式实现我实在看不明白。。就试着自己写了种办法
```python
x = [[1, 2], [1, 2], [2, 3], [2, 3], [2, 3], [3, 4]]
y = [list(t) for t in set(tuple(_) for _ in x)]  # 列表嵌套去重
n = [x.count(i) for i in y]  # 统计频率
z = zip(y, n)
z = sorted(z, key=lambda t: t[1], reverse=True)  # 按频率倒排序
print(z)
```
输出结果
```
[([2, 3], 3), ([1, 2], 2), ([3, 4], 1)]
```
一开始对x直接使用set函数报错
```python
x = [[1, 2], [1, 2], [2, 3], [2, 3], [2, 3], [3, 4]]
set(x)
```
报错为
TypeError: unhashable type: ‘list’
查资料后发现set不能对列表去重，但是可以对元组去重，去重后再将其变为列表
```python
x = [[1, 2], [1, 2], [2, 3], [2, 3], [2, 3], [3, 4]]
y = [list(t) for t in set(tuple(_) for _ in x)] 
```
最后书中代码与此方法结合
```python
import requests
from bs4 import BeautifulSoup
import re
import string
from collections import OrderedDict


headers = {'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8',
           'Accept-Encoding': 'gzip, deflate, sdch, br',
           'Accept-Language': 'zh-CN,zh;q=0.9',
           'Connection': 'keep-alive',
           'referer': 'https://zh.wikipeida.org',
           'Upgrade-Insecure-Requests': '1',
           'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'
           }


def clean_input(str_input):
    str_input = re.sub('\n+', ' ', str_input)  # 去除换行
    str_input = re.sub(r'\[[0-9]*\]', '', str_input)  # 去除引用标记
    str_input = re.sub(' +', ' ', str_input)  # 去除多余空格
    str_input = bytes(str_input, 'utf-8')
    str_input = str_input.decode('ascii', 'ignore')
    str_input = str_input.split(' ')
    str_clean_input = []
    for item in str_input:
        # 去除标点符号
        item = item.strip(string.punctuation)
        # 去除单字符单词
        if len(item) > 1 or item.lower() == 'a' or item.lower() == 'i':
            str_clean_input.append(item)
    return str_clean_input


def n_grams(str_input, n):
    str_input = clean_input(str_input)
    str_output = [str_input[i:i+n] for i in range(len(str_input)-n+1)]
    set_str_output = [list(t) for t in set(tuple(_) for _ in str_output)]
    str_count = [str_output.count(output) for output in set_str_output]
    dict_output = zip(set_str_output, str_count)
    return dict_output


html = requests.get('https://en.wikipedia.org/wiki/Roger_Federer', headers=headers)
html.encoding = 'utf-8'
bs0bj = BeautifulSoup(html.text, 'lxml')
content = bs0bj.find('div', {'id': 'mw-content-text'}).get_text()
my_n_grams = n_grams(content, 2)
my_n_grams = sorted(my_n_grams, key=lambda t: t[1], reverse=True)
print(my_n_grams)
print('2-grams count is : ' + str(len(my_n_grams)))
```
结果为
```
[(['Roger', 'Federer'], 283), (['in', 'the'], 215), (['Novak', 'Djokovic'], 122), (['Rafael', 'Nadal'], 121), (['of', 'the'], 116), (['World', 'Tour'], 110)…………
2-grams count is : 15874
```
所以我牛还是和这两个人有莫大的羁绊啊。。
