
最近工作上写了个爬虫，要爬取国家标准网上的一些信息，这自然离不了 Python，而在解析 HTML 方面，`xpath` 则可当仁不让的成为兵器谱第一。


你可能之前听说或用过其它的解析方式，像 `Beautiful Soup`，用的人好像也不少，但 `xpath` 与之相比，语法更简单，解析速度更快，就像正则表达式一样，刚上手要学习一番，然而用久了，那些规则自然而然的就记住了，熟练之后也很难忘记。


## 安装 lxml


`xpath` 只是解析规则，其背后是要有相应的库来实现功能的，就像正则表达式只是规则，而 Python 内置的 `re` 库，则是提供了解析功能。在 Python 中，`lxml` 就是 xpath 解析的实现库。


安装 `lxml` 非常简单，`pip install lxml` 就搞定了。


下面我们来看一下，在我这次真实的项目中，该如何发挥出它的威力。


## 加载 HTML 内容，应该用 etree.parse()、etree.fromstring() 还是 etree.HTML() ?


首先，把 lxml 库导进来：`from lxml import etree`。


HTML 内容的加载，是通过 etree 的方法载入的，具体有 3 个方法：parse()、fromstring() 和 HTML()。


* parse() 是从文件加载。
* fromstring() 是从字符串加载。
* HTML() 也是从字符串加载，但是以 HTML 兼容的方式加载进来的。


那我们应该选哪个方法呢？别犹豫，选 `etree.HTML()`，即使你的 HTML 内容来自文件。这是为何？


首先要说的一点是，HTML 也是 XML 的一种，而 XML 的标准规定，其必须拥有一个根标签，否则，这段 XML 就是非法的。而我们加载进来的 HTML 内容，可能本身就不是完整的，只是个片段，且没有根标签；或是加载进来的 HTML 从头到脚看起来都是完整的，但是中间的节点，有的缺少结束标签，这些情况，其实都是非法的 XML。那么，在用 parse() 或 formstring() 加载这种缺胳膊少腿的 HTML 的时候，就会报错；而用 `etree.HTML()` 则不会。


这是因为 etree.HTML() 加载方式，有很好的 HTML 兼容性，它会补全缺胳膊少腿的 HTML，把它变成一个完整的、合法的 HTML。


下面是一个从文件加载 HTML 的例子：



```
from lxml import etree

with open('test.html', 'r') as f:
    html = etree.HTML(f.read())
    print(html, type(html))

```

打印出来的结果是：，加载进来的 HTML 字符串，已经变成了 `Element` 对象。


后面我们通过 xpath 找 HTML 节点，全都是在这个 `Element` 对象上操作的。


## 找到你需要的 HTML 节点


下面是我想要找的 HTML 节点


![](https://img2024.cnblogs.com/blog/111619/202411/111619-20241127164843342-86092689.png)


在这个 table 表格中，第一个 tbody 是表头，第二个 tbody 是表内容，我们要如何定位到第二个 tbody ？


我们通常是调用上面获得的 `Element` 对象的 `xpath()` 方法，通过传入的 xpath 路径查找的。而路径有两种写法：一种是 `/` 开头，从 `html` 根标签，沿着子节点一个个找下来；另一种是 `//` 开头，即不论我们要找的节点在什么位置，找到就算，这种方式是最常用的。


比如，我们现在要找的 `tbody` 节点，它在 `table` 节点下，我们就可以这样写：`html.xpath('//table/tbody')`。这里的 `html` 是上面获得的 `Element` 对象，然后去找 HTML 内容中的、不管在任何位置的所有的 `table`，找到后再继续找它们下面的直接子节点 `tbody`，于是就匹配出来了。


可是这里有 2 个 `tbody`，我需要的是第二个，我们可以在 `[]` 中写条件表达式：`html.xpath('//table/tbody[2]')`，注意这里的序号是从 `1` 开始的。


## 强大的属性选择器


你可能有个疑问，如果 HTML 内容中不只有一个 `table` 表格，那我们通过 `html.xpath('//table/tbody[2]')` 岂不是找到了 2 个 `table` 里的第二个 `tbody`，而我需要的只是其中之一。没错，是存在这样一个问题。此时，我们就可以用属性选择器，来更精确的定位元素。


![](https://img2024.cnblogs.com/blog/111619/202411/111619-20241128144932661-64492573.png)


观察一下上面的 HTML 结构，`table` 表格的最外层有一个 `div`，它还有个 class 属性：`table-responsive`，假设这个 div 的 class 属性是整个 HTML 里独一无二的，那么我们就可以很放心的去查找 `div.table-responsive` 下的 `table`，进而精确定位我们想要的元素。


那么，要怎样写 `class = "table-responsive"` 这个条件呢？看看上面写条件表达式的 `[]`，那里面除了可以写数字来指定位置以外，也可以写其它各式各样的条件，比如：


`html.xpath('//div[@class="table-responsive"]/table/tbody[2]')`，这里我们就把 `class = "table-responsive"` 这个条件写进去了，从而定位到想要的元素。注意，在 xpath 中，所有的 HTML 属性匹配都是以 `@` 打头的，比如有这样一个 `[Click Me](#)` 元素，我们想要通过 id 定位它，可以这样写：`//a[@id="show_me"]`，是不是很简单。


假设很遗憾，我们这里的 `table-responsive` 不是唯一的，可能还有其它地方的 div 的 class \= "table\-responsive"，这该怎么办？没关系，我们可以找其它具有唯一 class 值的元素，比如：最外层 div 下的 `table.result_list` 这个元素，这个是唯一的。好了，下面开始写定位代码：`html.xpath('//table[@class="result_list"]/tbody[2]')`，但是运行后，发现找不到元素，这是为什么？


![](https://img2024.cnblogs.com/blog/111619/202411/111619-20241128145134204-75909122.png)


其实仔细观察一下就能发现，这个元素的 class 里不只有 `result_list`，它还包括其它一长串的内容：`class = "table result_list table-striped table-hover"`，所以匹配失败了。那要如何指定 class 包含某个属性呢？其实可以在条件表达式中，用 `contains()` 函数，无需精确匹配，而是模糊匹配，只要包含指定的字符串就可以了。比如：`html.xpath('//table[contains(@class, "result_list")]/tbody[2]')` 这样就可以实现了。


需要提一点的是，xpath 定位到的元素，不管是不是全局唯一的，它的返回值都是一个列表，需要通过下标获取其中的元素。


## 相对定位


我最终的目标，是要遍历表格中所有的内容行，获取其中的标准号和标准名称，于是我初步完成了如下代码：



```
from lxml import etree

with open('test.html', 'r') as f:
    html = etree.HTML(f.read())
    rows = html.xpath('//table[contains(@class, "result_list")]/tbody[2]/tr')

    for row in rows:
        td_list = row.xpath('...')

```

现在我能够成功地定位到每一行，下面需要再基于每一行，找到我需要的列：


![](https://img2024.cnblogs.com/blog/111619/202411/111619-20241128150410529-1761300852.png)


此时，我在 for 循环的内部，已经拿到了每一行 `row`，再通过 `row.xpath('//td')` 继续往下定位 `td` 就好了。


可是，当你运行这段代码的时候，你会发现不对劲，一行里面总共只有 8 个 td，为什么出来了 80 个【一行 8 个，总共 10 行】？这是把 HTML 中所有的 td 都找出来了吧，可是我明明是用上面获取的 `row` 对象来查找的呀，不是应该只基于当前行往下找吗？


这就牵扯到了 `绝对定位` 和 `相对定位`。


其实，我们上面讲到的 `/` 和 `//`，都是绝对定位，也就是从 HTML 内容的根节点往下查找。一个 HTML 内容的根节点是什么呢，它是 `html`，再往下是 `body`，再再往下才是自定义的标签。所以，上面代码的执行结果是那种情况，也就不足为奇了，因为它不是在当前所在的 `row` 节点查找的，而是从根节点 `/html/body/xxx/xxx/td` 往下查找的呀。


所以，在这里不能用 `绝对定位` 了，要用 `相对定位`，那要如何用？很简单，用 `.` 和 `..` 即可，这个我们可太熟悉了，`.` 就代表了当前节点 `row`，而 `..` 则代表了当前节点的上一层父节点 `tbody`。


好了，我们修正上面的代码：



```
from lxml import etree

with open('test.html', 'r') as f:
    html = etree.HTML(f.read())
    rows = html.xpath('//table[contains(@class, "result_list")]/tbody[2]/tr')

    for row in rows:
        td_list = row.xpath('./td')

        i = 1
        for td in td_list:
            if i == 2:
                pass
            elif i == 4:
                pass
            i += 1

```

这样就可以正常地找到每一行里面的 8 个 `td`，然后再单独处理第 2 个和第 4 个单元格，获取其中的信息就好了。


## 通过已知节点获取属性和文本


到目前为止，我们能拿到第 2 个和第 4 个 `td` 节点了，只要再获取里面的 `a` 标签的属性和文本就可以了。


![](https://img2024.cnblogs.com/blog/111619/202411/111619-20241128160132993-2017592865.png)


我们先获取 `onclick` 属性，通过 `td.xpath('./a')`，可以找到此 `td` 节点下面的 `a` 标签，然后调用 `a` 节点的 `get()` 方法，即可获得对应的属性值，代码如下：



```
a1 = td.xpath('./a')[0]
onclick = a1.get('onclick')

```

注意哦，`xpath()` 方法的返回值，始终是一个列表，所以我们用下标 `[0]` 先把它从列表中取出来，然后再获取其属性。 至于属性内的值，我实际想取的是里面的一串 ID 字符串，这个再用正则表达式取一下就可以了。


要获取节点内的文本，也很简单，获得到的节点有一个 `text` 属性，可以直接得到节点的文本内容：`a1.text`。


## 好用的兄弟节点选择器


上面的代码逻辑有点挫，我们先是获取到一行里的所有 `td`，然后循环遍历它，在遍历的过程当中，只取其中的 2 个 `td`，着实有些浪费。假设一行里有 1000 个 `td`，那这里岂不是要循环 1000 次，就只为了取 2 个？


虽然从实际运行速度上来讲，影响微乎其微，但对于有代码洁癖和强迫症的人来说，是不可接受的，所以，我们要改造它。


重新观察一下 HTML 结构，我发现第 4 个单元格有个明显的特征，它的 `class = "mytxt"`：


![](https://img2024.cnblogs.com/blog/111619/202411/111619-20241128175135369-1509677151.png)


我们可以很容易地找到它：`title_td = tr.xpath('./td[@class="mytxt"]')[0]`，然后再基于刚找到的 `title_td`，查找从它往上数第 2 个兄弟节点，这样就省略了一个循环，只要查找两次就完成了。


那么，怎么查找上面的兄弟节点呢？用 `preceing-sibling`，比如：`title_td.xpath('./preceding-sibling::td[2]')`，这就代表要查找 `title_td` 上面的、从它这里往上数、排在第 2 位的 `td` 节点。


除了 `preceding-sibling` 之外，还有 `following-sibling`，顾名思意，是往下查找兄弟节点。


以上我只介绍了这 2 个，其实还有很多类似的选择器，具体可以参考下面的速查手册。


最后，我改造的代码如下：



```
from lxml import etree

with open('test.html', 'r') as f:
    html = etree.HTML(f.read())
    rows = html.xpath('//table[contains(@class, "result_list")]/tbody[2]/tr')

    for row in rows:
        title_td = row.xpath('./td[@class="mytxt"]')[0]
        title_link = title_td.xpath('./a')[0]
        title_onclick = title_link.get('onclick')

        print(title_onclick, title_link.text)

        id_td = title_td.xpath('./preceding-sibling::td[2]')[0]
        id_link_text = id_td.xpath('./a/text()')[0]

        print(id_link_text)

```

## 速查手册


xpath 的规则并不复杂，常用的也就那些，用熟了自然就记住了。但像正则表达式一样，它还有许多不常用却很好用的特性，你还是需要偶尔查一下具体的作用和用法。


这里有一个非常好的速查手册，虽然里面的内容看起来不够丰富、很简单，但是可以一目了然，并且它用 css 的语法来作类比，就能够更好地理解每一个 xpath 规则的实际用途。


速查手册：[https://devhints.io/xpath](https://github.com):[slower加速器](https://chundaotian.com)


