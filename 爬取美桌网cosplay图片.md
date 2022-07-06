---
title: 爬取美桌网cosplay图片
tags:
  - 爬虫
categories:
  - Python
  - 爬虫
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp13.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp13.jpg'
abbrlink: e434
date: 2020-09-29 10:46:27
updated: 2020-09-29 10:46:32
---



**开发环境：**

①Python3.8.5		

②Pycharm-community-2020.2

③GoogleChrome 84.0.4147.135 

④XPath Helper 2.0.2

**目标网址：**【美桌网cosplay图片大全】http://www.win4000.com/meinvtag26_1.html

可以看到这个cosplay大全一共有5页，我们需要将每页的每个相册里的所有图片都爬取下来。

整个爬取过程可以分为如下四步：1.分析网页性质2.发送网络请求3.解析数据（筛选数据）4.将数据保存至本地。

**实验过程：**

- 打开目标网址后，我们发现url地址栏中下划线后面的数字表示这是该cosplay大全的第几页。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20200929113954.png)

---

- 在浏览器中选中相册右键选择检查，可以看到该模块对应的前端代码，我们的目标就是获取黄色方框的url地址，也就是每个相册所在的网页地址。于是我们可以使用XPath插件通过标签去一步步定位，可以看到results结果集里有24条数据，刚好对应每页24个相册。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214939.png)

对应的xpath为：

//div[@class="list_cont Left_list_cont  Left_list_cont2"]//div[@class="tab_tj"]//ul/li/a/@href

---

- 进入相册后，我们发现url地址栏中下划线后面的数字表示这是该相册的第几张图片。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215034.png)

---

- 右键检查图片，data-original后面的url地址即是我们最终需要的图片地址，可通过该地址下载图片。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215035.png)

对应的xpath为：

//div[@class="pic-meinv"]/a/img/@data-original

---

**代码实现：**

```python
import requests
import parsel
import time
from datetime import datetime

# cosplay大全共有5页，使用拼接url访问每一页
for page in range(1, 6):
    url = 'http://www.win4000.com/meinvtag26_{}.html'.format(str(page))
    headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36'}

    response = requests.get(url=url, headers=headers).text

    # 使用parsel的xpath解析获得每个相册所在的url地址集合，使用getall()方法
    selector = parsel.Selector(response)
    album_list = selector.xpath('//div[@class="list_cont Left_list_cont  Left_list_cont2"]//div[@class="tab_tj"]//ul/li/a/@href').getall()
    # print(album_list)

    # 每个相册十多张图片，使用拼接picture_url访问每一张图片的网页地址
    for album_url in album_list:
        # 每个相册的图片不超过20张
        for num in range(1, 20):
            picture_url = album_url.split('.')[0] + '.' + album_url.split('.')[1] + '.' + album_url.split('.')[2] + '_' + str(num) + '.' + album_url.split('.')[3]
            response_2 = requests.get(url=picture_url, headers=headers).text
            selector_2 = parsel.Selector(response_2)

            # 获得每张图片的下载地址，使用get()
            imgPath = selector_2.xpath('//div[@class="pic-meinv"]/a/img/@data-original').get()
            # print(imgPath)

            # 拼接的picture_url可能不存在该页面，解析获得的imgPath可能为null
            if not imgPath is None:
                # content是获取二进制文本数据
                img_data = requests.get(url=imgPath, headers=headers).content

                # 按时间命名图片，保正图片按相册排序
                file_name = datetime.now().strftime("%Y%m%d-%H%M%S") + '.jpg'
                # file_name = img_url.split('/')[-1]
                # print(file_name)

                with open("img\\" + file_name, mode='wb') as f:
                    f.write(img_data)
                    print("保存完成：" + file_name)
                # 阻塞1s，防止图片名重复导致覆盖了图片
                time.sleep(1)
```

---

**运行结果：**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214940.png)