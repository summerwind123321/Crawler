# 梨视频多进程爬取视频
- 更新时间：2021.04.17
- 需要用的库：
  - requests
  - lxml
  - random
  - multiprocessing
  - re
  - os

"""
import requests
from lxml import etree
import random
from multiprocessing.dummy import Pool
import re
import os
if not os.path.exists("./vedio"):
    os.mkdir("./vedio")
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'
}
url = "https://www.pearvideo.com/category_5"
page_text = requests.get(url= url, headers = headers).text
tree = etree.HTML(page_text)
li_list = tree.xpath('//*[@id="listvideoListUl"]/li')
urls = []
for li in li_list:
    detail_url = "https://www.pearvideo.com/"+li.xpath('./div/a/@href')[0]
    vedio_name = li.xpath("./div/a/div[2]/text()")[0]+".mp4"
    print(vedio_name,detail_url)
    detail_text = requests.get(url = detail_url,headers = headers).text
    headers = {
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36',
        "Referer":detail_url
    }
    cont_id = re.findall(r"_(.+)",detail_url)[0]
    params = {
    "contId": cont_id,
    "mrd":str(random.random())
    }
    url = "https://www.pearvideo.com/videoStatus.jsp"
    response = requests.get(url = url,headers = headers, params = params).json()
    src_url = response["videoInfo"]["videos"]["srcUrl"]
    src_url_list = src_url.split("/")
    src_url_list_last = src_url_list[-1]
    last_list = src_url_list_last.split("-")
    last_list[0] = "cont-"+cont_id
    last = "-".join(last_list)
    src_url_list[-1] = last
    new_src_url = "/".join(src_url_list)
    dic = {
        "name":vedio_name,
        "url":new_src_url
    }
    urls.append(dic)
def get_data(dic):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36',
    }
    url = dic["url"]
    data = requests.get(url = url,headers = headers).content
    with open("./vedio/"+dic["name"],"wb") as fp:
        fp.write(data)
        print("下载成功")
pool = Pool(4)
pool.map(get_data,urls)
pool.close()
pool.join()
"""
