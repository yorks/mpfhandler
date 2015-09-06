Overview
========
This module provides an additional log handler for Python's standard 
logging package (PEP 282). 
This handler will write log events to log file which is rotated at a 
certain time(day, hour, min) and  multiple processes supported.

这个模块提供了一个 Python logging handler 类。它会根据配置按时间来
轮转日志文件，并且支持多线程

Details
=======
The ``MultProcTimedRotatingFileHandler`` class is a drop-in replacement for
Python's standard log handler ``RotatingFileHandler``. This module uses file
locking so that multiple processes can concurrently log to a single file without
dropping or clobbering log events. This module provides a file rotation scheme
like with ``RotatingFileHanler``.  Extra care is taken to ensure that logs
can be safely rotated before the rotation process is started. 

This class ratating file at an exact time, such as every day(23:59:59) or 
every hour(xx:59:59) and so on. 

If you have multiple instances of a script (or multiple scripts) all running at
the same time and writing to the same log file, then *all* of the scripts should
be using this class.

This class require `portalocker` to deal with file locking.  Please be aware
that portalocker only supports Unix (posix) an NT platforms at this time, and
therefore this package only supports those platforms as well. But, only tested
on Linux.

基于标准的 `RotatingFileHandler` 类开发，支持多线程，多进程，
轮转方式是按照时间, 每天/每小时/每分钟 不依赖于进程的启动时间，而是根据系统时间
进行轮转.

如果有多个脚本或者实例都需要打日志到同一个文件，请一定都要用这个类
`MultProcTimedRotatingFileHandler`, 因为它是基于文件锁的.

这个类使用了 `portalocker` 去出来多进程之间的文件锁，所以它只支持 *nix 跟 NT 系统，
但是我只测试了在 Linux 下面使用。


Why
========
This class fork from `TimedRotatingFileHandler` and `ConcurrentLogHandler`.
For `TimedRotatingFileHandler` issues:
1. thread safe but not process safe
2. rotating time is accumulate, this means it deponding the process when to start up.
`ConcurrentLogHandler` is good, but it only rotating by the logfile size.


这个类其实是参考了 `TimedRotatingFileHandler`，`ConcurrentLogHandler` 这两个handler,
为什么要写这个呢？主要是上面提到的这两个都有一些小问题。
`TimedRotatingFileHandler`:
1. 虽然是线程安全，但是并不支持多进程
2. 轮转的时间是依赖于进程的启动时间，所以会出现轮转点并不是常规的59s.

`ConcurrentLogHandler` 如果只是实现了根据日志文件大小来轮转, 未提供按时间的.


Installation
============
Use the following command to install this package:

    pip install mpfhandler

If you are installing from source, you can use:

    python setup.py install

Or wget the single file to your python sys.path:

    wget https://github.com/yorks/mpfhandler/raw/master/src/mpfhandler.py -O\
        /usr/lib/python2.6/site-packages/mpfhandler.py



Examples
========

Simple Example
--------------
Here is a example demonstrating how to use this module directly
```python

    from logging import getLogger, INFO
    from mpfhandler import MultProcTimedRotatingFileHandler
    import os
    
    log = getLogger()
    # Use an absolute path to prevent file rotation trouble.
    logfile = os.path.abspath("mylogfile.log")
    # Rotate log every hour
    rotateHandler = MultProcTimedRotatingFileHandler(logfile,  when='h')
    log.addHandler(rotateHandler)
    log.setLevel(INFO)

    log.info("Here is a very exciting log message, just for you")
```
Django Example
--------------
```python
# settings.py
LOGGING={
 #...
 'handlers':{
        'custom_mpf_rotate':{
            'level': 'DEBUG',
            'class': 'mpfhandler.MultProcTimedRotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'logs/custom_mptf.log'),
            'when' : 'D', # day, or H for hour
            #'interval' : 1,  # TODO
            #'debug' : False, # handler own log
            'formatter': 'verbose'
        },
  },
 'loggers':{
        ...
        'customapp': {
            'handlers': ['console', 'custom_mpf_rotate'],
            'level': 'DEBUG',
            'propagate': False,
        },
        #...
  }

}

# app/view.py
import logging
log=logging.getLogger(__name__)
log.info('Here is a very exciting log message, just for you')


```

Attention
==========
1. interval, backupCount options is not working!
2. $logpath.lock is the lock file to handle multiple processes, donot delete it.
3. There is no stress-testing has been done, if it slow
   down for logging it might be the lock issue.


注意
=========
1. `interval`, `backupCount` 参数暂不支持，这也就是说，日志的压缩清理您需要额外处理。
2. `$logpath.lock` 这个文件请不要删除，是用来出来文件锁的。
3.  还没经过压力测试，如果担心会托慢您的程序，可能是因为文件锁引起.
