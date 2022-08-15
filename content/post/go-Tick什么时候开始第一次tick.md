---
title: go Tick什么时候开始第一次tick
date: 2019-04-07 21:02:56
tags: [Go]
---
ticker的第一次tick是立即运行呢，还是等待一定时间后运行呢？运行下面的代码测试
```
package main

import (
    "fmt"
    "time"
)

func main() {
	ticker := time.NewTicker(time.Duration(1) * time.Second)
	j := 0
	defer ticker.Stop()

	fmt.Println(j, time.Now())
	for tm := range ticker.C{
		j ++
		fmt.Println(j, tm)
		if j >= 10 {
			break
		}
	}
}
```

输出如下：
```
0 2019-04-07 21:10:29.1018538 +0800 CST m=+0.006997901
1 2019-04-07 21:10:30.1015869 +0800 CST m=+1.006731001
2 2019-04-07 21:10:31.1028936 +0800 CST m=+2.008037701
3 2019-04-07 21:10:32.1046246 +0800 CST m=+3.009768701
4 2019-04-07 21:10:33.1154541 +0800 CST m=+4.020598201
5 2019-04-07 21:10:34.1012874 +0800 CST m=+5.006431501
6 2019-04-07 21:10:35.1031226 +0800 CST m=+6.008266701
7 2019-04-07 21:10:36.104569 +0800 CST m=+7.009713101
8 2019-04-07 21:10:37.1064066 +0800 CST m=+8.011550701
9 2019-04-07 21:10:38.1094041 +0800 CST m=+9.014548201
10 2019-04-07 21:10:39.1118504 +0800 CST m=+10.016994501

Process finished with exit code 0
```
可以看到在j = 0和j = 1 期间耗时了1s，也就是一个tick 的duration之后才会产生第一个tick。如果设置10s超时，则应该设置条件为j >= 10跳出循环。如果j > 10,实际上超时时间是11s。
从源码中可以看到，下一个tick产生的时间由when()决定
```
func NewTicker(d Duration) *Ticker {
	if d <= 0 {
		panic(errors.New("non-positive interval for NewTicker"))
	}
	// Give the channel a 1-element time buffer.
	// If the client falls behind while reading, we drop ticks
	// on the floor until the client catches up.
	c := make(chan Time, 1)
	t := &Ticker{
		C: c,
		r: runtimeTimer{
			when:   when(d),
			period: int64(d),
			f:      sendTime,
			arg:    c,
		},
	}
	startTimer(&t.r)
	return t
}
```
