
## [golang使用redis锁（避免误解锁/死锁/过期引起并发）：go-redis, redigo](https://www.cnblogs.com/yylyhl/p/18072942 "发布于 2024-03-21 18:05")

【go-redis】简单实现方式，不会死锁/误解锁

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
package main

import (
    "context"
    "fmt"
    "sync"
    "time"

    redis2 "github.com/redis/go-redis/v9"
)

var mutex sync.Mutex

// redis加锁 sec:锁定秒数(避免死锁),value 锁唯一值(避免误解锁)
// import redis2 "github.com/redis/go-redis/v9"
// import "context"
func LockGoRedis(key string, val string, sec int) bool {
    mutex.Lock()
    defer mutex.Unlock()
    client := redis2.NewClient(&amp;redis2.Options{
        Addr:     "127.0.0.1",
        Password: "",
        DB:       0,
    })
    ctx := context.Background()
    res, err := client.SetNX(ctx, key, val, time.Second*time.Duration(sec)).Result()
    if err != nil {
        fmt.Print("Lock error;", err.Error())
    }
    return res
}

// redis解锁 key,value一致才可解锁
func UnlockGoRedis(key string, val string) {
    client := redis2.NewClient(&amp;redis2.Options{
        Addr:     "127.0.0.1",
        Password: "",
        DB:       0,
    })
    ctx := context.Background()
    if res, _ := client.Get(ctx, key).Result(); res == val {
        client.Del(ctx, key)
    }
}
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

缺点：不能自动展期，当业务处理时间超过锁定时长时，会被其他业务或客户端拿到锁，造成并发。
加个自动续期

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
func LockGoRedis(key string, val string, sec int, autoDelay bool) bool {
    mutex.Lock()
    defer mutex.Unlock()
    client := redis2.NewClient(&amp;redis2.Options{
        Addr:     "127.0.0.1",
        Password: "",
        DB:       0,
    })
    ctx := context.Background()
    ttl := time.Second * time.Duration(sec)
    res, err := client.SetNX(ctx, key, val, ttl).Result()
    if err != nil {
        fmt.Print("Lock error;", err.Error())
    } else if autoDelay {
        // 锁自动续期
        go func() {
            for {
                time.Sleep(ttl / 2) //时间过半时再续期
                if res, _ := client.Get(ctx, key).Result(); res == val {
                    client.Expire(ctx, key, ttl)
                } else {
                    break // value不一致停止续期
                }
            }
        }()
    }
    return res
}
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

 

【redigo】版

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
// 加锁 sec:锁定秒数(避免死锁),value 锁唯一值(避免误解锁)
// import "github.com/gomodule/redigo/redis"
func LockRedigo(key string, val string, sec int, autoDelay bool) bool {
    mutex.Lock()
    defer mutex.Unlock()
    var pool = &amp;redis.Pool{
        MaxIdle:     10,  //初始连接数量
        MaxActive:   0,   //连接池最大连接数量,（0自动设置）
        IdleTimeout: 300, //连接空闲时间 300秒
        Dial: func() (redis.Conn, error) {
            return redis.Dial("tcp", "127.0.0.1")
        },
    }
    client := pool.Get()

    ttl := time.Second * time.Duration(sec)
    reply, err := client.Do("setnx", key, val)
    if err != nil {
        fmt.Print("Lock error;", err)
        return false
    }
    if reply.(int64) == 1 {
        client.Do("expire", key, ttl.Seconds())
    }
    if autoDelay {
        // 锁自动续期
        go func() {
            for {
                time.Sleep(ttl / 2) //时间过半时再续期
                if res, _ := client.Do("get", key); res == val {
                    client.Do("expire", key, ttl.Seconds())
                } else {
                    break // value不一致停止续期
                }
            }
        }()
    }
    return true
}

// 解锁 key,value一致才可解锁
func UnlockRedigo(key string, val string) {
    var pool = &amp;redis.Pool{
        MaxIdle:     10,  //初始连接数量
        MaxActive:   0,   //连接池最大连接数量,（0自动设置）
        IdleTimeout: 300, //连接空闲时间 300秒
        Dial: func() (redis.Conn, error) {
            return redis.Dial("tcp", "127.0.0.1")
        },
    }
    client := pool.Get()
    if res, _ := redis.String(client.Do("get", key)); res == val {
        client.Do("del", key)
    }
}
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")
