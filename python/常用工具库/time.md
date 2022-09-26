# time

time 是 python 的内置模块，这里记录一些她的常用方法

- `time.time()` ：获取当前时间戳
- `time.localtime(time.time())`：获取当前时间
- `time.strftime("%Y-%m-%d %H:%M:%S", time.localtime()) `：格式化时间-格式化成2016-03-20 11:45:39形式
- `time.sleep(secs)`：推迟调用线程的运行，secs指秒数。