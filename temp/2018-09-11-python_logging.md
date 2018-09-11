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
import logging.handlers
# 生成并获取记录器，所有记录器默认继承于root记录器，使用相同名字多次获取的是同一个记录器，此处建议使用当前模块名称__name__  
logger = logging.getLogger('test')  
# 定义日志文件名称  
LOGFILE = 'test.log'  
# 定义处理器，处理器有多种类型，请自行查询  
handler = logging.handlers.RotatingFileHandler(filename=LOGFILE, maxBytes=10*1024*1024, backupCount=5)  
# 定义消息格式，格式中有一些内置属性，请自行查询
fmt = 'asctime: %(asctime)s - levelname: %(levelname)s - lineno: %(lineno)d - process: %(process)d - \n message: %(message)s'  
# 定义格式器  
formatter = logging.Formatter(fmt)  
# 把格式器安装到处理器上  
handler.setFormatter(formatter)  
# 把处理器安装到记录器上  
logger.addHandler(handler)  
# 设置记录器的消息级别，共有6个级别: notset、debug、info、warning、error、critical，只有消息级别高于记录器级别时才会被处理，notset只有root记录器才可以设置
logger.setLevel(logging.DEBUG)
# 打印debug日志
logger.debug('test debug')
# 打印info日志
logger.info('test info')
# 打印warning日志
logger.warning('test warning')
# 打印error日志
logger.error('test error')
# 打印critical日志
logger.critical('test critical')
```  

* 使用配置文件  
如果一个项目拥有多个模块，并且每个模块都需要一个单独的日志文件，那么上面的第一种方式可能需要在每个模块中都去定义一遍，这样比较麻烦，此时，可以使用自定义配置文件的方式去定义，如下(logger.conf):  
``` conf  
# 声明所有的记录器，root是系统默认的必须有，其他的自定义  
[loggers]
keys=root,testLogger1,testLogger2

# 声明所有的处理器, 全部自定义
[handlers]
keys=rootHandler,testHandler1,testHandler2

# 声明所有的格式器，全部自定义，因为我所有的日志格式都相同，所以这里只声明了一个，起名为all
[formatters]
keys=all

# 配置所有的记录器，指定消息级别和所使用的处理器，qualname属性为使用记录器时的真实名字  
[logger_root]
level=NOTSET
handlers=rootHandler

[logger_testLogger1]
level=DEBUG
handlers=testHandler1
qualname=test1

[logger_testLogger2]
level=DEBUG
handlers=testHandler2
qualname=test2

# 配置所有的处理器，指定处理器类型、参数和所使用的格式器  
[handler_rootHandler]
class=handlers.RotatingFileHandler
formatter=all
args=('/dev/null', 'a', 1024*1024, 5)

[handler_testHandler1]
class=handlers.RotatingFileHandler
formatter=all
args=('../log/test1.log', 'a', 1024*1024, 5)

[handler_testHandler2]
class=handlers.RotatingFileHandler
formatter=all
args=('../log/test2.log', 'a', 1024*1024, 5)

# 配置所有的格式器，指定具体格式
[formatter_all]
format='asctime: %(asctime)s - levelname: %(levelname)s - lineno: %(lineno)d - process: %(process)d - \n message: %(message)s'
```  
  
在程序启动时使用配置文件初始化logging  
> logging.config.fileConfig('logger.conf')  
  
在其他模块中调用时获取qualname属性指定名字的记录器，然后直接使用即可  
> logger = logging.getLogger('test1')  
> logger.debug('test1')  
