---
title: supervisor + scrapy + python
date: 2020-11-26 14:42:57
categories:
- 项目经验
tags:
- 爬虫
- python
---
在ubuntu16.04下
最近在做一个项目，它是在服务器部署，首先我们的人员请求爬取新浪微博的话题和关键词，然后会返回爬到的微博用户id，bid，和图像url。之后再把这些图像送到深度学习所做的图像侵权检测系统，判断是否侵权我们的素材，返回相关的信息。

1.图像侵权检测

2.python 开启http服务

3.scrapy 爬取新浪微博

4.supervisor 进程管理

其实123慢慢调好了，这周调的最多的是怎么把1234放在一起，碰到了相当多的问题。


### 开启supervisor 执行的命令可以是虚拟环境的命令，需要加上路径

比如

>[program:ibaotu-image-match]
command=**/usr/local/data/anaconda3/envs/snakes/bin/python** /usr/local/data/www/weibo_search_master/http_server_GET.py
directory=/usr/local/data/www/weibo_search_master
autostart=true
autorestart=true
stdout_logfile=/usr/local/data/www/log/out.log
stderr_logfile=/usr/local/data/www/log/err.log

执行命令的时候就知道是执行哪里的python了

还可以web查看管理supervisor
>[inet_http_server] ; inet (TCP) server disabled by default
port=*:9001 ; (ip_address:port specifier, *:port for all iface)
;username=admin ; (default is no username (open server))
;password=admin ; (default is no password (open server))


### supervisor命令

查看状态
supervisorctl status
更新
supervisorctl update
重置
supervisorctl reload
结束
supervisorctl stop all
开始
supervisorctl start all
更新配置
supervisord -c /etc/supervisord/supervisord.conf

### 遇到的报错

>unix:///var/run/supervisor.sock no such file

1.sudo touch /var/run/supervisor.sock

2.sudo chmod 777 /var/run/supervisor.sock

3.sudo service supervisor restart


>unix:///var/run/supervisor/supervisor.sock refused connection

supervisord -c /etc/supervisord/supervisord.conf
启动supervisord并使用配置

>The 'supervisor==3.2.0' distribution was not found and is required by the application

如果默认的python是python2,应该不会报错。如果是python3，就要修改
和terminator一样也是python版本引起的，编辑/usr/bin/supervisord将#!/usr/bin/python修改为#!/usr/bin/python2即可
貌似启动supervisor 只能用python2

>supervisord不能正常地话，查看它的log

supervisorctl tail ibaotu-image-match stderr

>supervisorctl 启动起来，一直在报错重启， OSError: [Errno 98] Address already in use

1.
netstat -tunlp
kill -9 6153
这个是每次都要这样，有点烦
2.
find / -name supervisor.sock
unlink /name/supervisor.sock

>root@ibaotu-algo:/usr/bin# scrapy
Traceback (most recent call last):
  File "/usr/local/bin/scrapy", line 7, in <module>
    from scrapy.cmdline import execute
  File "/usr/local/lib/python3.5/dist-packages/scrapy/__init__.py", line 12, in <module>
    from scrapy.spiders import Spider
  File "/usr/local/lib/python3.5/dist-packages/scrapy/spiders/__init__.py", line 22
    name: Optional[str] = None
        ^
SyntaxError: invalid syntax

在虚拟环境是好的，但是放在一起跑就出问题了。我研究了好久，才发现还是命令路径的问题。
在终端输入scrapy，虚拟环境是ok的，在普通的就报错。两个scrapy版本是一样的，
只不过虚拟环境的python是3.6.5，普通的python是3.5.2，不知道是不是和这个有关。
把scrapy改成/usr/local/data/anaconda3/envs/snakes/bin/scrapy

>scrapy是命令行，怎么在代码里面添加？

```
#cmdline.execute(["scrapy", "crawl", "search", "-a", "start_date=%s"%(start_date), "-a", "end_date=%s"%(end_date), "-a", "keyword_list=%s"%(keyword)])
cmd = "/usr/local/data/anaconda3/envs/snakes/bin/scrapy crawl search -a " + "start_date=%s"%(start_date) + " -a " + "end_date=%s"%(end_date) + " -a " + "keyword_list=%s"%(keyword)
os.system(cmd)
```
>开启的http服务，客户请求爬虫，怎么把数据返回？

一开始的想法是能不能搞个全局变量。怎么搞也搞不过去，
python跨文件的全局变量，后面知道scrapy是新开的进程，数据根本到不了。
后面把数据存到csv或者json文件，爬完再读取，再返回。

> no module named ...

因为路径的问题，也搞了好久，明明都添加了，还是报错，sys.path.append()
最后因为没有使用到全局变量，这个报错也就不了了之了。

>遍历字典，写入csv，一个数据固定的写入一行

writer.writerow([item['weibo'][key] for key in item['weibo'].keys() if key == 'id' or key == 'bid' or key == 'pics'])

>是csv转成json，还是直接爬取pipeline到json

我选择了后者，爬取了json格式的文件下来，包括id，bid，picurl组成的json格式的数据。
一开始json格式还写不进去，应该是格式问题，我弄了data格式，之后ok了。

```
with codecs.open(jsonfile_path, 'a+') as f:
    data = {
    'id': item['weibo']['user_id'],
    'bid': item['weibo']['bid'],
    'screen_name': item['weibo']['screen_name'],
    'pics': item['weibo']['pics']
    }

    lines = json.dumps(data, ensure_ascii=False)
    f.write(lines + "\n")
```

>然后也出现supervisor一直重启的问题

因为配置中restart:默认为true，然后一直有htpp进程开着，报错OSError: [Errno 98] Address already in use；还有个原因就是原来的程序不是死循环，一会运行结束，又重启了。


### 最后这个1234基本就能愉快地在一起玩耍了。
