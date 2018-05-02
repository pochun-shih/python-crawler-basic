## quotes to scrapy

[http://quotes.toscrape.com/](http://quotes.toscrape.com/)
<br>
接續 [chapter6](chapter6.md)
### items<span>.py
開起專案中的 items，它應該會長這樣
```python
# -*- coding: utf-8 -*-

import scrapy

class DsItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
```
目標為作者與引言，所以加入目標欄位，變成：
```python
# -*- coding: utf-8 -*-

import scrapy

class DsItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    author = scrapy.Field()
    quotes = scrapy.Field()
```

<hr>

### spiders/ 裡的 qs<span>.py

[spider文件](https://docs.scrapy.org/en/latest/topics/spiders.html)<br>
利用之前練習的 Selectors 獲得目標文字<br>
修改 parse_page()
```python
def parse_page(self, response):
    quotes = response.css('div.quote')
    for quote in quotes:
        _quote = quote.css('span.text::text').extract_first()
        author = quote.css('small.author::text').extract_first()
```
接下來將目標文字塞進 items 並回傳(就像流程圖畫的，傳 items 就會丟給 pipelines處理，傳 request 就會給 SCHEDULER 排行程)<br>
import item
```python
from ds.items import DsItem # import 剛寫好的 item
# from porjectName.fileName import className
```
將資料塞進 item並回傳
```python
def parse_page(self, response):
    item = DsItem()
    quotes = response.css('div.quote')
    for quote in quotes:
        # item 的 key值為 items.py裡設定好的欄位
        item['quotes'] = quote.css('span.text::text').extract_first()
        item['author'] = quote.css('small.author::text').extract_first()
        yield item
```
spiders/qs.py整體
```python
# -*- coding: utf-8 -*-
import scrapy
from ds.items import DsItem

class QsSpider(scrapy.Spider):
    name = 'qs'
    allowed_domains = ['quotes.toscrape.com']

    def start_requests(self):
        c = ['love', 'life', 'books']
        for i in range(0, 3):
            yield scrapy.Request("http://quotes.toscrape.com/tag/%s/" % c[i], callback=self.parse_page)

    def parse_page(self, response):
        item = DsItem()
        quotes = response.css('div.quote')
        for quote in quotes:
            item['quotes'] = quote.css('span.text::text').extract_first()
            item['author'] = quote.css('small.author::text').extract_first()
            yield item
```

<hr>

### pipelines<span>.py

[pipelines文件](https://docs.scrapy.org/en/latest/topics/item-pipeline.html)<br>
在 class 中新增兩個 function，open_spider() 與 close_spider()
```python
class DsPipeline(object):
    # 開啟時，只會執行一次
    def open_spider(self, spider):
        pass

    # 處理 item，回傳 item繼續給下一個 pipeline處理
    # 所以可以將功能切成很多個 pipeline
    def process_item(self, item, spider):
        return item

    # 關閉時，只會執行一次
    def close_spider(self, spider):
        pass
```
開檔、關檔、創立物件
```python
class DsPipeline(object):
    def open_spider(self, spider):
        self.f = open('quotes.json', 'w', encoding='utf-8')
        self.quotes = {'quote': []}

    def process_item(self, item, spider):
        return item

    def close_spider(self, spider):
        self.f.close()
```
存 item到物件，轉成 json，寫檔
```python
import scrapy
import json

class DsPipeline(object):
    def open_spider(self, spider):
        self.f = open('quotes.json', 'w', encoding='utf-8')
        self.quotes = {'quote': []}

    def process_item(self, item, spider):
        self.quotes['quote'].append(dict(item))
        return item

    def close_spider(self, spider):
        self.f.write(json.dumps(self.quotes, ensure_ascii=False))
        self.f.close()
```

<hr>

### setting<span>.py

[Scrapy setting](https://docs.scrapy.org/en/latest/topics/settings.html)<br>
啟用 Pipeline，將預設好的註解拿掉
```python
ITEM_PIPELINES = {
    'ds.pipelines.DsPipeline': 300,
}
```
設定下載延遲(單位：秒)
```python
DOWNLOAD_DELAY = 1
```
不遵守對方給的 robot.txt
```python
ROBOTSTXT_OBEY = False
```
修改預設發送 request裡的 header
```python
DEFAULT_REQUEST_HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.162 Safari/537.36'
}
```
不修改的話發送的 User-Agent會是
```python
User-Agent: Scrapy/1.5.0 (+https://scrapy.org)
```

<hr>

### 執行爬蟲

```bash
$ scrapy crawl qs
# scrapy crawl spiderName
```
它存檔的位置為你執行爬蟲的位置