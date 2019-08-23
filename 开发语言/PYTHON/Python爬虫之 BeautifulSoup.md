# Python 爬虫之 BeautifulSoup

#### 简介

Beautiful Soup提供一些简单的、python式的函数用来处理导航、搜索、修改分析树等功能。它是一个工具箱，通过解析文档为用户提供需要抓取的数据，因为简单，所以不需要多少代码就可以写出一个完整的应用程序。Beautiful Soup自动将输入文档转换为Unicode编码，输出文档转换为utf-8编码。你不需要考虑编码方式，除非文档没有指定一个编码方式，这时，Beautiful Soup就不能自动识别编码方式了。然后，你仅仅需要说明一下原始编码方式就可以了。

Beautiful Soup已成为和lxml、html6lib一样出色的python解释器，为用户灵活地提供不同的解析策略或强劲的速度。

#### 安装

```python
pip install BeautifulSoup4
或
easy_install BeautifulSoup4
```

#### 创建BeautifulSoup对象

首先应该导入BeautifulSoup类库 

`from bs4 import BeautifulSoup`

下面开始创建对像，在开始之前为了方便演示，先创建一个html文本，如下：

```html	
<html><head><title>The Dormouse's story</title></head>

<body>

<p class="title" name="dromouse"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were

<!-- Elsie -->,

Lacie and

Tillie;

and they lived at the bottom of a well.</p>

<p class="story">...</p>

```

创建对象：soup=BeautifulSoup(html,’lxml’),这里的lxml是解析的类库，目前来说个人觉得最好的解析器了，一直在用这个，安装方法：

`pip install lxml`

####　Tag

Tag就是html中的一个标签，用BeautifulSoup就能解析出来Tag的具体内容，具体的格式为soup.name,其中name是html下的标签，具体实例如下：

```bash
print soup.title  # 输出title标签下的内容，包括此标签，这个将会输出<title>The Dormouse's story</title>

print soup.head

```

#### 注意：

这里的格式只能获取这些标签的第一个，后面会讲到获取多个标签的方法。其中对于Tag有两个重要的属性name和attrs,分别表示名字和属性,介绍如下：

- name:对于Tag，它的name就是其本身，如soup.p.name就是p
- attrs是一个字典类型的，对应的是属性-值，如print soup.p.attrs,输出的就是{‘class’: [‘title’], ‘name’: ‘dromouse’},当然你也可以得到具体的值，如print soup.p.attrs[‘class’],输出的就是[title]是一个列表的类型，因为一个属性可能对应多个值,当然你也可以通过get方法得到属性的，如：print soup.p.get(‘class’)。还可以直接使用print soup.p[‘class’]

#### get

get方法用于得到标签下的属性值，注意这是一个重要的方法，在许多场合都能用到，比如你要得到`<img src=”#”>`标签下的图像url,那么就可以用`soup.img.get(‘src’)`,具体解析如下：

```python
print soup.p.get("class") #得到第一个p标签下的src属性
```

#### string

得到标签下的文本内容，只有在此标签下没有子标签，或者只有一个子标签的情况下才能返回其中的内容，否则返回的是None具体实例如下：

```python
print soup.p.string #在上面的一段文本中p标签没有子标签，因此能够正确返回文本的内容
print soup.html.string  #这里得到的就是None,因为这里的html中有很多的子标签
```

#### get_text()

可以获得一个标签中的所有文本内容，包括子孙节点的内容，这是最常用的方法



#### 搜索文档树

`find_all( name , attrs , recursive , text , **kwargs )`

find_all是用于搜索节点中所有符合过滤条件的节点

1. name参数：是Tag的名字，如p,div,title …..

`soup.find_all("p")` 查找所有的p标签，返回的是`[<b>The Dormouse's story</b>]`，可以通过遍历获取每一个节点，如下：

```python
ps=soup.find_all("p")
for p in ps:
    print p.get('class')   #得到p标签下的class属性
```

传入正则表达式：`soup.find_all(re.compile(r’^b’)`查找以b开头的所有标签，这里的body和b标签都会被查到

传入类列表：如果传入列表参数,BeautifulSoup会将与列表中任一元素匹配的内容返回.下面代码找到文档中所有`<a>`标签和`<b>`标签

`soup.find_all(["a", "b"])`

2. KeyWords参数，就是传入属性和对应的属性值，或者一些其他的表达式

- soup.find_all(id='link2'),这个将会搜索找到所有的id属性为link2的标签。传入正则表达式soup.find_all(href=re.compile("elsie")),这个将会查找所有href属性满足正则表达式的标签
- 传入多个值：soup.find_all(id='link2',class_='title') ,这个将会查找到同时满足这两个属性的标签，这里的class必须用class_传入参数，因为class是python中的关键词
- 有些属性不能通过以上方法直接搜索，比如html5中的data-*属性，不过可以通过attrs参数指定一个字典参数来搜索包含特殊属性的标签，如下：

```bash
# [<div data-foo="value">foo!</div>]
data_soup.find_all(attrs={"data-foo": "value"})   #注意这里的atts不仅能够搜索特殊属性，亦可以搜索普通属性
soup.find_all("p",attrs={'class':'title','id':'value'})  #相当与soup.find_all('p',class_='title',id='value')
```

3. text参数：通过 text 参数可以搜搜文档中的字符串内容.与 name 参数的可选值一样, text 参数接受 字符串 , 正则表达式 , 列表, True

```python
soup.find_all(text="Elsie")
#[u'Elsie']
soup.find_all(text=["Tillie", "Elsie", "Lacie"])
#[u'Elsie', u'Lacie', u'Tillie']
soup.find_all(text=re.compile("Dormouse"))
[u"The Dormouse's story", u"The Dormouse's story"]
```

4. limit参数：find_all() 方法返回全部的搜索结构,如果文档树很大那么搜索会很慢.如果我们不需要全部结果,可以使用 limit 参数限制返回结果的数量.效果与SQL中的limit关键字类似,当搜索到的结果数量达到 limit 的限制时,就停止搜索返回结果.

文档树中有3个tag符合搜索条件,但结果只返回了2个,因为我们限制了返回数量,代码如下：

```python
soup.find_all("a", limit=2)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

5. recursive 参数:调用tag的 `find_all()` 方法时,BeautifulSoup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 `recursive=False`

`find( name , attrs , recursive , text , **kwargs )`

它与 find_all() 方法唯一的区别是 find_all() 方法的返回结果是值包含一个元素的列表,而 find() 方法直接返回结果,就是直接返回第一匹配到的元素，不是列表，不用遍历，如soup.find("p").get("class")

#### 通过标签名查找
```python
print soup.select('title')
#[<title>The Dormouse's story</title>]
print soup.select('a')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

#### 通过类名查找
```python
print soup.select('.sister')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
#### 通过id名查找
```python
print soup.select('#link1')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
#### 组合查找

学过 css 的都知道 css 选择器，如 p #link1 是查找 p 标签下的 id 属性为 link1 的标签
```python
print soup.select('p #link1')    #查找p标签中内容为id属性为link1的标签
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]

print soup.select("head > title")   #直接查找子标签
#[<title>The Dormouse's story</title>]
```
#### 属性查找

查找时还可以加入属性元素，属性需要用中括号括起来，注意属性和标签属于同一节点，所以中间不能加空格，否则会无法匹配到。
```python
print soup.select('a[class="sister"]')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]


print soup.select('a[href="http://example.com/elsie"]')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
同样，属性仍然可以与上述查找方式组合，不在同一节点的空格隔开，同一节点的不加空格,代码如下：
```python
print soup.select('p a[href="http://example.com/elsie"]')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
以上的 select 方法返回的结果都是列表形式，可以遍历形式输出，然后用 get_text() 方法来获取它的内容
```python
soup = BeautifulSoup(html, 'lxml')
print type(soup.select('title'))
print soup.select('title')[0].get_text()

for title in soup.select('title'):
    print title.get_text()
```
