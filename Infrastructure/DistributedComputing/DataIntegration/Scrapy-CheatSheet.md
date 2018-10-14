# Scrapy CheatSheet

# 递归爬取

```py
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.selector import HtmlXPathSelector
from scrapy.spider import Spider
from scrapy.selector import Selector
from winkel.items import WinkelItem

class DmozSpider(CrawlSpider):
name = "dmoz"
    allowed_domains = ["mydomain.nl"]
    start_urls = [
        "http://www.mydomain.nl/Zuid-Holland"
    ]

    rules = (Rule(SgmlLinkExtractor(allow=('*Zuid-Holland*', )), callback='parse_winkel', follow=True),)

    def parse_winkel(self, response):
        sel = Selector(response)
        sites = sel.xpath('//ul[@id="itemsList"]/li')
        items = []

        for site in sites:
        item = WinkelItem()
        item['adres'] = site.xpath('.//a/text()').extract(), site.xpath('text()').extract(), sel.xpath('//h1/text()').re(r'winkel\s*(.*)')
        items.append(item)
        return items
```
