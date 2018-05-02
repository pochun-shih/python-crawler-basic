## 我甚麼都不想管只要跟我講怎麼做就好了的懶人包

安裝python3(記得裝 pip)<br>
[下載頁](https://www.python.org/downloads/)<br><br>

安裝 pipenv
```bash
$ pip install pipenv
```

建資料夾
```bash
$ mkdir scrapy-project
$ cd scrapy-project
```

建虛擬環境
```bash
$ pipenv install
```

進虛擬環境
```bash
$ pipenv shell
```

安裝套件
```bash
$ pipenv install scrapy
$ pipenv install pypiwin32
$ pipenv install Pillow
```

創建 scrapy專案
```bash
$ scrapy startproject pixiv
$ cd pixiv
```

新增 spider
```
$ scrapy genspider img "pixiv.net"
```

修改 items<span>.py
```python
# -*- coding: utf-8 -*-

import scrapy

class PixivItem(scrapy.Item):
    url = scrapy.Field()
    image_paths = scrapy.Field()
```

修改 img<span>.py
```python
# -*- coding: utf-8 -*-
import scrapy
from pixiv.items import PixivItem

class ImgSpider(scrapy.Spider):
    name = 'img'
    allowed_domains = ['pixiv.net']

    payload = {
        'pixiv_id': 'pixiv_id',
        'password': 'password',
        'source': 'pc',
        'return_to': 'https://www.pixiv.net/'
    }

    # 登入頁
    def start_requests(self):
        yield scrapy.Request("https://accounts.pixiv.net/login?lang=zh_tw/", callback=self.login)

    # 登入
    def login(self, r):
        # post_key = r.css('input[name=post_key]::attr(value)').extract_first()
        # self.payload['post_key'] = post_key
        # 因 from_response所以不用抓 post_key
        yield scrapy.FormRequest.from_response(r, formdata=self.payload, callback=self.go_gallery, dont_filter = True)

    # 圖片列表
    def go_gallery(self, r):
        yield scrapy.Request("https://www.pixiv.net/member_illust.php?id=366411", callback=self.go_image, dont_filter = True)

    # 觀看每張圖
    def go_image(self, r):
        head = 'https://www.pixiv.net'
        li = r.css('li.image-item a::attr(href)').extract()
        for url in li:
            yield scrapy.Request(head + url, callback=self.extract_url, dont_filter = True)

    # 取得大圖連結
    def extract_url(self, r):
        src = r.css('img.original-image::attr(data-src)').extract()
        item = PixivItem()

        for url in src:
            item['url'] = url
            yield item
```

修改 pipelines<span>.py
```python
# -*- coding: utf-8 -*-

import scrapy
from scrapy.pipelines.images import ImagesPipeline
from scrapy.exceptions import DropItem

class PixivImagesPipeline(ImagesPipeline):

    def file_path(self, request, response=None, info=None):
        image_guid = request.url.split('/')[-1]
        return '%s' % (image_guid)

    def get_media_requests(self, item, info):
        yield scrapy.Request(item['url'], headers={'referer': item['url']}, dont_filter = True)

    def item_completed(self, results, item, info):
        image_paths = [x['path'] for ok, x in results if ok]
        if not image_paths:
            raise DropItem("Item contains no images")
        item['image_paths'] = image_paths
        return item
```

修改 settings<span>.py ， 新增/修改下面規則
```python
ROBOTSTXT_OBEY = False
DEFAULT_REQUEST_HEADERS = {
  'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.162 Safari/537.36'
}
DOWNLOAD_DELAY = 3
ITEM_PIPELINES = {
   'pixiv.pipelines.PixivImagesPipeline': 300,
}
IMAGES_STORE = 'images'
```

爬圖片
```bash
$ scrapy crawl img
```

離開虛擬環境
```bash
$ exit
```