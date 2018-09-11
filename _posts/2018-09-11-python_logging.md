---  
layout: post  
title: Python 内置的日志系统  
date: 2018-09-11  
tags: [Python]  
---  
  
最近在项目中用到了 Python 内置的日志系统，感觉还蛮好用的，在这里简单记录下使用方法  

### Logging 的四大组件  
Python 的日志系统由四大基本组件构成，包括:
* logger : 记录器，负责输出日志信息，以及设置消息级别  
* handler : 处理器，负责指定日志输出位置，标准输出或指定具体文件，同时还可以指定日志使用的格式器  
* formatter : 格式器，负责格式化日志消息  
* filter : 过滤器，负责使用指定的规则过滤日志信息，不过我这里没有用到，也不是很了解，以后待补充  

### Logging 两种配置使用方式  
* 在代码中直接配置使用  
``` Python  
# 导入 logging 包
import logging  
# 生成并获取记录器，使用相同名字多次获取的是同一个记录器，此处建议使用当前模块名称__name__  
logger = logging.getLogger('test')  
# 定义日志文件名称  
LOGFILE = 'test.log'
handler = logging.handlers.RotatingFileHandler(filename=LOGFILE, maxBytes=10*1024*1024, backupCount=5)

fmt = 'asctime: %(asctime)s - levelname: %(levelname)s - lineno: %(lineno)d - process: %(process)d - ' \
      '\n message: %(message)s'
formatter = logging.Formatter(fmt)

handler.setFormatter(formatter)

logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

```  

* 使用配置文件