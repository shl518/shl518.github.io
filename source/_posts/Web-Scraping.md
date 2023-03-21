---
title: Web_Scraping
date: 2022-06-27 17:27:07
tags: 网络爬虫
---

urlparse模块主要是用于解析url中的参数 对url按照一定格式进行 拆分或拼接 

1.urlparse.urlparse

将url分为6个部分，返回一个包含6个字符串项目的元组：协议、位置、路径、参数、查询、片段。

```python
import urlparse
url_change = urlparse.urlparse('https://i.cnblogs.com/EditPosts.aspx?opt=1')
print url_change
```

输出结果为：

`ParseResult(scheme='https', netloc='i.cnblogs.com', path='/EditPosts.aspx', params='', query='opt=1', fragment='')`

其中 scheme 是协议 ,netloc 是域名服务器, path 相对路径 params是参数，query是查询的条件

```python
from urllib.request import urlopen
from bs4 import BeautifulSoup
from urllib.parse import urlparse
import re
import datetime
import random

pages = set()
random.seed(datetime.datetime.now())


def getInternalLinks(bs, includeUrl):
    includeUrl = '{}://{}'.format(urlparse(includeUrl).scheme, urlparse(includeUrl).netloc)
    internalLinks = []
    for link in bs.find_all('a', href=re.compile('^(/|.*)' + includeUrl + ')')):
        if link.attrs['href'] is not None:
            if link.attrs['href'] not in internalLinks:
                if link.attrs['href'].startwith('/'):
                    internalLinks.append(includeUrl + link.attrs['href'])
                else:
                    internalLinks.append(link.attrs['href'])
    return internalLinks


def getExternalLinks(bs, excludeUrl):
    externalLinks = []
    for link in bs.find_all('a', href=re.compile('^(http|www)((?!' + excludeUrl + ').)*$')):
        if link.attrs['href'] is not None:
            if link.attrs['href'] not in externalLinks:
                externalLinks.append(link.attrs['href'])
    return externalLinks


def getRandomExternalLink(startingPage):
    html = urlopen(startingPage)
    bs = BeautifulSoup(html, 'html.parser')
    externalLinks = getExternalLinks(bs, urlparse(startingPage).netloc)
    if len(externalLinks) == 0:
        print("No external Links")
        domain = '{}://{}'.format(urlparse(startingPage).scheme, urlparse(startingPage).netloc)
        internalLinks = getInternalLinks(bs, domain)
        return getRandomExternalLink(internalLinks[random.randint(0, len(internalLinks) - 1)])
    else:
        return externalLinks[random.randint(0, len(externalLinks) - 1)]


def followExternalOnly(startingPage):
    externalLink = getRandomExternalLink(startingPage)
    if not HasContainsUrl(externalLink):
        print('Random external Link is : {}'.format(externalLink))
        pages.add(externalLink)
    followExternalOnly(startingPage)


def HasContainsUrl(externalLink):
    global pages
    for page in pages:
        if (page == externalLink):
            return True
    return False

#main
followExternalOnly('https://www.py.cn/faq/python/15298.html')

```

