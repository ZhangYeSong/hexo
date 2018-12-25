---
title: 使用python requests爬取Gerrit代码提交数据
date: 2018-12-25 19:00:44
categories:
 - 爬虫
tags:
 - python
 - 爬虫
---
> 组长要求把gerrit上所有的提交和comment数据都收集起来写到excel里面。现学python把这事给自动化了。之前连python语法都只知道一点点，大部分是现学，连中午饭都没时间吃。写完只有一个字：爽！于是晚上沙县加了个鸡腿。

### 大致思路
先说下，这个python脚本是根据BrainZou在github上的代码改写的https://github.com/BrainZou/PythonSpider/tree/master/Gerrit，非常感谢这位仁兄，不然我今天肯定下不了班了- -
<!-- more -->

用chrome打开gerrit，F12打开开发者工具点击Network->XHR可以看到页面的HTTP请求，没有就刷新一下，如下图：
![查找页面请求](https://upload-images.jianshu.io/upload_images/5586297-2b240b7905be7b60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后用python的第三方库requests来进行请求，得到结果后进行处理，处理完成后写入csv文件，csv可以用任何类似excel的软件打开。
运行环境：安装python3，再安装pip3，然后运行pip3 install requests来安装第三方库requests，最后使用python3 gerrit.py运行
通过requests可以爬取所有的前后端分离的网站，不分离的网站只能爬网页源码了。

### 完整代码
代码比较简单，都加了注释所以就直接上代码了，注意加上自己公司gerrit网站的cookie。
```
#coding:utf-8
import requests
import json
import re
import csv

#gerrit的默认url是下面这个url，然后下一页按钮的url是这个url加上这一页最后一项的_sortkey(形如0049b98c0000f3b8)
UNMERGE_URL = 'https://gerrit-sin.parrot.biz/changes/?n=25&O=1'
MERGED_URL = 'https://gerrit-sin.parrot.biz/changes/?q=status:merged&n=25&O=1'

#用来判断是哪个项目组添加的comment
TS_IDS = [9,10,11,12,13,16,22]
PARROT_IDS = [2,5]



# 请求访问接口，获取到json格式数据，稍加处理，写入表格。
def requesst(url,limit_time,start,merged):
    continue_flag = 1
    headers = {
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36',
        'Cookie': 'GerritAccount=aHyuzmj3aQisN6CyELrhcU1JnxOEcJK; XSRF_TOKEN=aHyuS4.LW.5ZOwZizn7DMLUV7IfZ-Pi'
    }
    request_url = url + "&S=" + str(start)
    data = requests.get(request_url, headers=headers, verify=True).text
    # content = data.text #你要的数据,JSON格式的
    #去掉多余的几个垃圾字符
    remove = re.compile('\)\]\}\'')
    # data = data.replace(')]}\'',"")
    data = re.sub(remove, "", data)
    data_json = json.loads(data)
    if(len(data_json) <=0):
        continue_flag = 0

    #打开csv文件，准备写入
    with open("gerrit_commits.csv", "a",newline='') as csvfile:
        writer = csv.writer(csvfile)
        #遍历所有commit数据
        for one in data_json:
            one["created"] = time_format(one["created"])
            if (time_cmp(one["created"], limit_time) < 0):
                print("已经发现有超过时间的数据")
                continue_flag = 0
                break
            else:
                project = one["project"];
                owner = get_owner(one["owner"]["_account_id"])

                #获取未merge代码的预估review时间
                if(merged != True):
                    advice_url = "https://gerrit-sin.parrot.biz/changes/"+str(one["_number"])+"/revisions/1/reviewassistant~advice"
                    advice_data = requests.get(advice_url, headers=headers, verify=True).text
                    advice_data = re.sub(remove, "", advice_data)
                    estimated = advice_data[advice_data.index("\\u003cstrong\\u003e")+len("\\u003cstrong\\u003e"):advice_data.index("\\u003c/strong\\u003e reviewing")]

                #获取commit的comment信息
                comment_url = "https://gerrit-sin.parrot.biz/changes/"+str(one["_number"])+"/comments"
                comment_data = requests.get(comment_url, headers=headers, verify=True).text
                comment_data = re.sub(remove, "", comment_data)
                comment_dict = json.loads(comment_data)


                ts_size, parrot_size = get_comment_size(comment_dict)
                print("comment size = " + str(ts_size) + ":" + str(parrot_size))

                #将数据写入表格中
                writer.writerow([
                    owner, 
                    project[project.rfind("/"):],
                    one["subject"],
                    "https://gerrit-sin.parrot.biz/"+str(one["_number"]),
                    one["insertions"],
                    one["deletions"],
                    "NA" if merged==True else estimated,
                    parrot_size + ts_size,
                    parrot_size,
                    ts_size,
                    merged
                ])

                #打印日志
                print(
                    owner, 
                    project[project.rfind("/"):],
                    one["subject"],
                    "https://gerrit-sin.parrot.biz/"+str(one["_number"]),
                    one["insertions"],
                    one["deletions"],
                    "NA" if merged==True else estimated,
                    parrot_size + ts_size,
                    parrot_size,
                    ts_size,
                    merged
                )
    return continue_flag, start+len(data_json)

#通过id获取owner姓名
def get_owner(account_id):
    return {
        1: 'Florent-Xavier Perrin',
        5: 'Omar Akrout',
        9: 'Wenping Wang',
        10: 'Wang Kui',
        11: 'Guoruyi',
        12: 'Hongliang',
        13: 'Zhangwei',
        16: 'Di Jin',
        22: 'Fengyou'
    }.get(account_id, 'error')

#解析comment json获取comment个数
def get_comment_size(comment_dict):
    ts_size = 0
    parrot_size = 0

    for k,v in comment_dict.items():
        for comment in v:
            _account_id = comment["author"]["_account_id"]
            if(_account_id in TS_IDS):
                ts_size += 1
            if(_account_id in PARROT_IDS):
                parrot_size += 1
        
    return ts_size, parrot_size


#对时间"2017-12-11 02:07:27.000000000"格式化为20171211020727
def time_format(time):
    time = time.replace('.000000000', "")
    time = time.replace(' ', "")
    time = time.replace(':', "")
    time = time.replace('-', "")
    return time

#两个字符串转int 相减计算大小
def time_cmp(first_time, second_time):
    print(first_time)
    print(second_time)
    return int(first_time) - int(second_time)

def main():
    limit_time = input("请输入截止时间  格式：20171211000000\n")
    url = UNMERGE_URL
    continue_flag = 1
    start = 0
    #写标题
    with open("gerrit_commits.csv", "w",newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(["Owner", "Module", "Title", "Gerrit link", "Insertions", "Deletions",
            "Gerrit estimated review time", "Total comments", "Total Parrot comments",
            "Total TS comments", "Merged"])
    # 循环爬下一页
    while (continue_flag):
        continue_flag, start = requesst(url,limit_time,start,False)

    url = MERGED_URL
    continue_flag = 1
    start = 0
    while (continue_flag):
        continue_flag, start = requesst(url,limit_time,start,True)

if __name__ == '__main__':
    main()
```

