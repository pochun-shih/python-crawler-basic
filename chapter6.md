## Scrapy 簡介

Scrapy [官網](https://scrapy.org/) 及 [Github](https://github.com/scrapy/scrapy)
<br>
Scrapy 是一個 Python 的爬蟲框架 ， 提供許多強大的爬蟲功能。

## Scrapy Selectors

Scrapy 有兩種 [Selectors](https://docs.scrapy.org/en/latest/topics/selectors.html) ， [XPath](https://www.w3.org/TR/xpath/) 和 [css](https://www.w3.org/TR/selectors/)， 這邊只會簡介 css ， XPath 可用在 XML與HTML ， css 可用在 HTML。

```bash
scrapy shell "http://quotes.toscrape.com/"
```
scrapy shell 是一個可互動的 shell ， 可快速的用 python測試爬蟲程式碼 ， 並且不用創建專案及spider

```html
<div class="quote" itemscope="" itemtype="http://schema.org/CreativeWork">
    <span class="text" itemprop="text">“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”</span>
    <span>
        by
        <small class="author" itemprop="author">Albert Einstein</small>
        <a href="/author/Albert-Einstein">(about)</a>
    </span>
    <div class="tags">
        Tags:
        <meta class="keywords" itemprop="keywords" content="change,deep-thoughts,thinking,world">
        <a class="tag" href="/tag/change/page/1/">change</a>
        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
        <a class="tag" href="/tag/world/page/1/">world</a>
    </div>
</div>
```
上面是網頁中第一個引言的 HTML，而全部的引言都是這種格式，利用 Selectors 取得全部的引言 div (利用chrome devtools查看網頁原始碼 ， 用 network 標籤查看是不是向"http://quotes.toscrape.com/" 發 request 就能拿到目標 response)
```python
$ quotes = response.css('div.quote') # 取得全部 Class 為 quote 的 div
```
取得第一個div
```python
$ quote = quotes[0]
```
再取得引言(::text會取得 html標籤中的值)
```python
$ text = quotes.css('span.text::text')
```
這個時候 text 依然是一個 selector 物件 ， 利用 extract() 或 extract_first() 取得其中的值
```python
$ text.extract()[0] # 或
$ text.extract_first()
```
當 selector 的值超過一個時 extract() 會回傳一個陣列 extract_first() 會回傳陣列的第一個值
<br>
可以將上面的指令合併成
```python
$ text = response.css('div.quote span.text::text').extract_first()
```
取得標籤屬性值
```python
$ text = response.css('div.quote div.tags')[0].css('a.tag::attr(href)').extract()
# ['/tag/change/page/1/', '/tag/deep-thoughts/page/1/', '/tag/thinking/page/1/', '/tag/world/page/1/']
```
regex
```python
$ text = response.css('div.quote div.tags')[0].css('a.tag::attr(href)').re(r'/tag/\w+/page/1/')
# ['/tag/change/page/1/', '/tag/thinking/page/1/', '/tag/world/page/1/']
$ text = response.css('div.quote div.tags')[0].css('a.tag::attr(href)').re(r'/tag/(\w+)/page/1/')
# ['change', 'thinking', 'world']
$ text = response.css('div.quote div.tags')[0].css('a.tag::attr(href)').re(r'/(tag)/(\w+)/page/1/')
# ['tag', 'change', 'tag', 'thinking', 'tag', 'world']
# 兩個()就取兩個地方
```

## 創建 Scrapy 專案

```bash
$ scrapy startproject ds
# $ scrapy startproject projectName
$ cd ds
# $ cd projectName
```

專案最上層，一個資料夾與檔案，有 scrapy.cfg 檔 scrapy 就會判定為它是一個 scrapy 專案資料夾
```bash
projectName/
scrapy.cfg
```
下一層
```bash
__pycache__/    # 不要理它
spiders/        # spiders 為抓資料和分析資料的地方
_init_.py       # https://docs.python.org/3/tutorial/modules.html#packages
items.py        # 設定要傳給 pipelines 的參數
middlewares.py  # 預設給的中介層 可以增加刪除複寫 ， 也能再創一個新檔寫中介層
pipelines.py    # 預設給的pipelines 可以增加刪除複寫 ， 也能再創一個新檔寫pipelines ， 用來處理 spider 傳過來的 item(分析好的資料)
settings.py     # 專案設定檔 https://docs.scrapy.org/en/latest/topics/settings.html
```

![scrapy 流程](https://docs.scrapy.org/en/latest/_images/scrapy_architecture_02.png)
上圖為 scrapy專案爬蟲流程

## 建立 Spider

```bash
# $ scrapy genspider (-t templateName) spiderName example.com
#   Available templates:
#     basic
#     crawl
#     csvfeed
#     xmlfeed
#   default: basic
$ scrapy genspider qs "quotes.toscrape.com"
```
這時候會創立一個檔名為 qs 的爬蟲
```bash
__pycache__/
_init_.py
qs.py
```

## Spider 程式結構說明

[Spider](https://docs.scrapy.org/en/latest/topics/spiders.html)<br>

剛創建的qs
```python
# -*- coding: utf-8 -*-
import scrapy

class QsSpider(scrapy.Spider):
    name = 'qs'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        pass
```
qs 為這個 spider 的名子，利用下面這個指定利用 qs 進行爬取
```bash
$ scrapy crawl qs
```
start_urls 為要爬取的網站陣列 ， 可以一次很多個網站<br>
parse() 為爬取網站後預設處理 response 的 function<br>
start_urls 可以替代為 start_requests() ， 這樣較靈活
```python
def start_requests(self):
    c = ['love', 'life', 'books']
    for i in range(0, 3):
        yield scrapy.Request("http://quotes.toscrape.com/tag/%s/" % c[i], callback=self.parse_page)
```
上面將爬取三個網頁，callback 不指定就會利用預設的 parse() 處理 response ， 這邊有指定 callback=self.parse_page，所以要新增 parse_page()
```python
def parse_page(self, response):
    pass
```
這個時候檔案會變成
```python
# -*- coding: utf-8 -*-
import scrapy

class QsSpider(scrapy.Spider):
    name = 'qs'
    allowed_domains = ['quotes.toscrape.com']

    def start_requests(self):
        c = ['love', 'life', 'books']
        for i in range(0, 3):
            yield scrapy.Request("http://quotes.toscrape.com/tag/%s/" % c[i], callback=self.parse_page)

    def parse_page(self, response):
        pass
```
用自己定義好的 function (但如上面的callback沒指定又把 parse() 刪掉就會出錯，它會找不到要使用的預設 parse())
