


## [golang 时间比对时，time.Now()与time.Parse()的使用注意](https://www.cnblogs.com/yylyhl/p/18058723 "发布于 2024-03-07 14:36")

在11:28时执行以下代码

```
nowTime := time.Now()
t1, err := time.Parse("2006-01-02 15:04", "2024-03-07 08:00:00")
result := nowTime.Before(t1)
```


本以为result应该是false，结果竟然是true。

  

调试下看看两者的区别发现：

time.Parse()是UTC时间，无时区信息，如：time.Time(2024-03-07T08:00:00Z){wall: 0, ext: 63845395200, loc: *time.Location nil}
time.Now()是本地时间，有时区信息，如：time.Now()：time.Time(2024-03-07T11:28:29+08:00, +11732401) {wall: 13939321407022326624, ext: 11732401, loc: *time.Location：{ name:"Local",zone:[{name: "CST", offset: 28800, isDST: false}] } }

两者做时间比较时，time.Now()的时间会去掉时区差，即减去8小时，变成2024-03-07 03:00:00。导致2024-03-07T11:28:29+08:00早于2024-03-07T08:00:00Z。

 

两种方式解决：

1. time.Now()的时间格式化掉时区信息。

```
nowTime := time.Now()
t1, errr := time.Parse("2006-01-02 15:04:05", nowTime.Format("2006-01-02 15:04:05"))
```


 

2.将time.Parse()改用time.ParseInLocation()设置时区得到本地时间。

```
local, _ := time.LoadLocation("Asia/Shanghai")//local, _ := time.LoadLocation("Local")
t1, err := time.ParseInLocation("2006-01-02 15:04:05", "2024-03-07 08:00:00", local) 
```


 



