### 1. 时间date
* 获取指定时间n天后时间```date -d "20180423 30 days " "+%Y%m%d"```
* 获取指定时间n天前时间```date -d "20180423 -30 days " "+%Y%m%d"```

获取一个UUID：```cat /proc/sys/kernel/random/uuid```

查看文件中的重复行：```sort ./test.log | uniq -d ```