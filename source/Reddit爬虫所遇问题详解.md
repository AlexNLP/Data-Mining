---
share: ture
---

# Reddit爬虫笔记

**Reddit** is a network of communities where people can dive into their interests, hobbies and passions. There's a community for whatever you're interested in on **Reddit**.

简单来讲，Reddit就是在特定子主题下用户自动发掘内容和分享的讨论社区，其中包含帖子、评论、投票、点赞、分享等操作，其操作自由度和交互性非常适合广大用户创造价值。

Reddit对于数据挖掘工作者非常友好。其一在于，目前已有非常多的Reddit公开数据集可以方便获取，如REDDITBINARY、REDDITMULTI5K等数据集被广泛用于顶会中系列模型的训练。另外，Reddit网站也提供了限制较少的API，可以简单上手获取到称心如意的一手研究数据。因此，研究Reddit数据集如何获取对于往后的研究工作是十分必要的。

## Reddit爬虫方式

###  Reddit API
有用的链接地址

1.  [官方 api 说明文档](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.reddit.com%2Fdev%2Fapi)
2.  [官方 api 使用规范](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Freddit-archive%2Freddit%2Fwiki%2FAPI)
3.  [reddit 授权说明](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Freddit-archive%2Freddit%2Fwiki%2FOAuth2)
4.  [reddit app 列表地址](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.reddit.com%2Fprefs%2Fapps%2F)
5.  [praw 文档](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fpraw.readthedocs.io%2Fen%2Flatest%2Findex.html)
6.  [python 采集 reddit 例子](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.storybench.org%2Fhow-to-scrape-reddit-with-python%2F)
 
 具体操作步骤：
- 注册Reddit账号
- 申请Reddit API
  首先在[Reddit app](https://www.reddit.com/prefs/apps) 以Script for personal use身份填写name, description, redirect url，然后create app得到以下的信息：
  **client_id**: The client ID is at least a 14-character string listed just under “personal use script” for the desired [developed application](https://www.reddit.com/prefs/apps/)
  **client_secret**: The client secret is at least a 27-character string listed adjacent to `secret` for the application.
  **password**: The password for the Reddit account used to register the application.
  **username**: The username of the Reddit account used to register the application.
  信息详细介绍请看：[Authenticating via OAuth — PRAW 7.6.1.dev0 documentation](https://praw.readthedocs.io/en/latest/getting_started/authentication.html)
- 利用以上信息开始填写代码
```code
# coding: UTF-8 
#!/use/bin/env python3 
import praw 
import pandas as pd 
import datetime as dt 

reddit = praw.Reddit( client_id='your-clientID', 
					client_secret='your secret',
					user_agent='your_platform:dev_tmp:v0.1 (by /u/dev_tmp)', 
					# redirect_uri='http://localhost:8080' 
					username='dev_tmp', 
					password='HGFhgf123'
) 
# print(reddit.auth.url(["identity"], "...", "permanent")) 
print(reddit.user.me()) 

# all 是<class 'praw.models.reddit.subreddit.Subreddit'>类型, 
# 具体使用见：https://praw.readthedocs.io/en/latest/code_overview/models/subreddit.html 
all = reddit.subreddit("all") 
print(type(all)) 

# submission 的类型是<class 'praw.models.reddit.submission.Submission'>， 
# 具体属性列表见：https://praw.readthedocs.io/en/latest/code_overview/models/submission.html 
messages = { "id": [], 
			"url": [], 
			"title": [], 
			"score": [], 
			"comms_num": [], 
			"body": [], 
			"created": [] 
} 

for submission in all.search("tiktok", limit=5): 
messages["id"].append(submission.id) 
messages["url"].append(submission.url) 
messages["title"].append(submission.title) 
messages["score"].append(submission.score) 
messages["comms_num"].append(submission.num_comments) 
messages["body"].append(submission.selftext) 
messages["created"].append(submission.created) 

# search 结果类型是<class 'praw.models.listing.generator.ListingGenerator'> 
# praw 中的其他类型文档为：
https://praw.readthedocs.io/en/latest/code_overview/other.html # res = 
all.search("tiktok") 
# print(type(res)) 

data = pd.DataFrame(messages) 
data.to_csv('data.csv', index=False)

```
  这样就能获得你想要的数据了
- reddit API持续开发|
  这部分可以详见Reddit给出的官方文档，按照文档即可得到相应的数据。 

- 但是可能会存在以下的问题，我已经给出了解决方法如下。
  - ***问题：目前Reddit 只能科学访问。如果按照正常的爬虫方式来获取网站数据会遇到
prawcore.exceptions.RequestException: error with request HTTPSConnectionPool(host='www.reddit.com', [[port=443]]): Max retries exceeded with url: /api/v1/access_token (Caused by ProxyError('Cannot connect to proxy.', OSError(0, 'Error')))>)***

  - ***解决:
  
  相关解决参考
  [# HTTPSConnectionPool(host='xxxxx', port=443): Max retries exceeded with url:xxxxxxxx (Caused by Ne...](https://blog.csdn.net/qq_39377418/article/details/102552822), 
  [python爬虫之requests.exceptions.ProxyError: HTTPSConnectionPool(host='www.xxxx.com', port=443): Max retries exceeded with url: / (Caused by ProxyError('Cannot connect to proxy.', timeout('_ssl.c:1108: Th - Eeyhan - 博客园 (cnblogs.com)](https://www.cnblogs.com/Eeyhan/p/14610998.html)
    - 使用VPN科学访问解决error with request HTTPSConnectionPool。l[[Clash for Windows]] 仅支持浏览器网页，其他程序类软件无法使用它进行科学上网。但是它提供了一个非常便捷的工具TUN Mode，可以使所有的程序都经过Proxy进行科学访问，这样就能解决ProxyError等问题。另外，最好选取GLobal下的美国服务器会更加稳定。如果想更加高效稳定爬取大量Reddit数据，可以选择将[[程序部署国外外网服务器爬取]]。
    - 关闭系统代理ProxyError。系统代理仍然会导致出现上述错误。
    ![[Pasted image 20220520174235.png]]

Reddit网站API爬虫示例可以参见：
[(36条消息) Reddit网站获赞最高文章/评论的爬取_小丫头い的博客-CSDN博客](https://blog.csdn.net/zm714981790/article/details/51326036)
[如何采集reddit - hgf doing - OSCHINA - 中文开源技术交流社区](https://my.oschina.net/hgfdoing/blog/4314811)


### Scrapy 爬虫
Scrapy爬虫的简单介绍可以参见：
[1. Scrapy框架介绍与安装 · GitBook (csdn.net)](https://edu.csdn.net/notebook/python/week09/1.html)
[Scrapy爬虫框架教程（一）-- Scrapy入门 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/24669128)
[(36条消息) Scrapy爬虫入门教程十三 Settings（设置）___静禅__的博客-CSDN博客](https://blog.csdn.net/Ka_Ka314/article/details/81114570)

- 创建项目 
```code
scrapy startproject reddit
```
就会得到这样的文件夹
```code
reddit/
    scrapy.cfg
    reddit/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
            ...
```
这些文件分别是:
 - scrapy.cfg: 项目的配置文件。
 - reddit/: 该项目的python模块。之后您将在此加入代码。 
 -  reddit/items.py: 项目中的item文件。 
 - reddit/pipelines.py: 项目中的pipelines文件。 
 - reddit/settings.py: 项目的设置文件。 
 - reddit/spiders/: 放置spider代码的目录。

- 在reddit/spiders目录下创建reddit.py文件
```
scrapy genspider reddit reddit.com
```
- 启动爬虫
```code
scrapy crawl reddit
```
这样就可以爬取reddit.com上的子社区的数据啦。

Reddit网站Scrapy爬虫示例可以参见：
[Article-for-Datartisan/基于Scrapy的Reddit爬取与分析.md at master · fibears/Article-for-Datartisan (github.com)](https://github.com/fibears/Article-for-Datartisan/blob/master/article/%E5%9F%BA%E4%BA%8EScrapy%E7%9A%84Reddit%E7%88%AC%E5%8F%96%E4%B8%8E%E5%88%86%E6%9E%90.md)
[Scraping Reddit - DataScienceCentral.com](https://www.datasciencecentral.com/scraping-reddit/)
[StrikingLoo/kitten-getter: A Spider that crawls reddit.com/r/cats (github.com)](https://github.com/StrikingLoo/kitten-getter)


### 伪装请求头爬虫
- request网页返回response
```code
import requests
import csv
import time
from bs4 import BeautifulSoup4

url = "https://old.reddit.com/r/datascience/"
# Headers to mimic a browser visit
headers = {'User-Agent': 'Mozilla/5.0'}
# Returns a requests.models.Response object
page = requests.get(url, headers=headers)

```

- 需要主要是这样的方式爬虫会导致UA和IP被封，因此，我们需要建立Random UA和IP池来动态管理爬虫进程的伪装，具体的操作可以详见：
  [(36条消息) 八、python爬虫伪装 [免费伪装ip伪装请求头]_袁六加.的博客-CSDN博客_python伪装ip](https://blog.csdn.net/weixin_51852924/article/details/120019488)
Reddit网站伪装请求头爬虫示例可以参见：
[使用Python和BeautifulSoup 4抓取Reddit - H5W3](https://www.h5w3.com/204097.html)


## 总结
至此，关于Reddit网站的爬虫方式已经介绍完毕，后续的使用将继续更新。