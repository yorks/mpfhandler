Overview
========
This module provides an additional log handler for Python's standard 
logging package (PEP 282). 
This handler will write log events to log file which is rotated at a 
certain time(day, hour, min) and  multiple processes supported.

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

This package require `portalocker`_ to deal with file locking.  Please be aware
that portalocker only supports Unix (posix) an NT platforms at this time, and
therefore this package only supports those platforms as well. But, only tested
on Linux.

Why
========
This class fork from `TimedRotatingFileHandler` and `ConcurrentLogHandler`.
For `TimedRotatingFileHandler` issues:
1. thread safe but not process safe
2. rotating time is accumulate, this means it deponding the process when to start up.
`ConcurrentLogHandler` is good, but it only rotating by the logfile size.




Installation
============
Use the following command to install this package:

    pip install mpfhandler

If you are installing from source, you can use:

    python setup.py install


Examples
========

Simple Example
--------------
Here is a example demonstrating how to use this module directly (from within
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

Attention
==========
1. interval, backupCount options is not working!
2. $logpath.lock is the lock file to handle multiple processes, donot delete it.
3. There is no stress-testing has been done, if it slow
   down for logging it might be the lock issue.
