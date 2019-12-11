---
title: Golang学习（二）
date: 2017-12-31 16:13:51
lastmod:
tags: [Golang, 工具]
categories:
 - 造轮子
typora-root-url: ..
typora-copy-images-to: ../images
---
# 闲扯
2017年的最后一天啦！今天是最后一批90后（1999年12月31日出生）的18岁生日，祝他们生日快乐！明天就是2018年啦，意味着90后已经全部成年，逐步成为社会的中流砥柱啦！
顺便吐槽下，被朋友圈的18岁照片刷屏啦，岁月是把杀猪刀，18岁的少年们都去哪啦！小伙伴们不要只顾着工作，也注意下身体啊，要变强不要变秃啊！不要变成油腻的中年胖子啊！

祝自己在2018年能更加努力，锻炼好身体！还有很多很多事情等我去做！Fight！

言归正传，在[Golang学习（一）]({{< relref "Golang学习(一).md" >}})那篇博客中我们写了一个小工具，当时还没学习到并发，正好元旦假期学习了下并发，来实践改进下我们的工具。

----

# 正文
## 遗留Bug修复
文件复制过程中异常报错，程序没有退出的问题已经修复，创建文件和打开文件时加入了panic

```go
srcFile, err := os.Open(src)
if err != nil {
    fmt.Println(err)
    panic("打开文件错误!")
}
defer srcFile.Close()
desFile, err := os.Create(des)
if err != nil {
    fmt.Println(err)
    panic("创建文件错误!")
}
defer desFile.Close()
```

## 使用Golang的gorutine来并发，提高性能
创建一个容量与期望生成文件个数大小相当的布尔型的<b>chan</b>，然后循环执行任务，每次向<b>chan</b>写入一个值，并通过读取<b>chan</b>来阻塞<b>main</b>函数的结束，伪代码如下

```go
func main() {
    c := make(chan bool, *generateFileNumber)
    for count := 0; count < *generateFileNumber; count++ {
        dosomething(c)
    }
    <-c
}
func dosomething(c) {
    do
    c <- true
}
```

# 代码

```Golang
/*
 * User: toddlerya
 * Date: 2017/12/23
 * Update: 2017/12/31
 * ds接入模块加压工具
 */

package main

import (
	"flag"
	"fmt"
	"io"
	"math/rand"
	"os"
	"runtime"
	"strconv"
	"strings"
	"time"
)

func judgeExists(name string) bool {
	if _, err := os.Stat(name); err != nil {
		if os.IsNotExist(err) {
			return false
		}
	}
	return true
}

func copyFile(c chan bool, src, des string) (w int64, err error) {
	srcFile, err := os.Open(src)
	if err != nil {
		fmt.Println(err)
		panic("打开文件错误!")
	}
	defer srcFile.Close()

	desFile, err := os.Create(des)
	if err != nil {
		fmt.Println(err)
		panic("创建文件错误!")
	}
	defer desFile.Close()

	c <- true

	return io.Copy(desFile, srcFile)
}

func generateRandomNumber(start int, end int) int {
	if end < 0 || start < 0 || (end-start) <= 0 {
		fmt.Println("[-] 随机数起始值[start]必须大于等于0, 截至值[end]必须大于起始值!")
		fmt.Printf("[-] 请检查配置是否正确: start=%v, end=%v\n", start, end)
		panic("随机数参数错误")
	}
	rand.Seed(time.Now().UnixNano())
	num := rand.Intn((end - start)) + start
	return num
}

func main() {
	runtime.GOMAXPROCS(runtime.NumCPU())
	srcFilePath := flag.String("s", "/home", "输入原始文件路径")
	dstFilePath := flag.String("d", "/tmp", "目标输出路径")
	renameFormat := flag.String("f", "abc-{random1}-456_780_{random2}.zip", "参数原始文件替换格式")
	random1Start := flag.Int("r1s", 100, "随机参数1的起始值")
	random1End := flag.Int("r1e", 999, "随机参数1的截止值")
	random2Start := flag.Int("r2s", 100000, "随机参数2的起始值")
	random2End := flag.Int("r2e", 999999, "随机参数2的截止值")
	generateFileNumber := flag.Int("g", 10000, "需要生成的文件个数")
	flag.Parse()
	fmt.Printf("[srcFilePath]          :%s\n", *srcFilePath)
	fmt.Printf("[dstFilePath]          :%s\n", *dstFilePath)
	fmt.Printf("[renameFormat]         :%s\n", *renameFormat)
	fmt.Printf("[random1Start]         :%v\n", *random1Start)
	fmt.Printf("[random1End]           :%v\n", *random1End)
	fmt.Printf("[random2Start]         :%v\n", *random2Start)
	fmt.Printf("[random2End]           :%v\n", *random2End)
	fmt.Printf("[generateFileNumber]   :%v\n", *generateFileNumber)
	fmt.Println("=======================================")
	t1 := time.Now()
	if judgeExists(*srcFilePath) {
		c := make(chan bool, *generateFileNumber)
		fmt.Println("[+] 开始随机生成目标数据, 请注意", *dstFilePath, "目录是否有数据生成\t", "计划生成", *generateFileNumber, "个文件Orz")
		for count := 0; count < *generateFileNumber; count++ {
			random1Val := generateRandomNumber(*random1Start, *random1End)
			random2Val := generateRandomNumber(*random2Start, *random2End)
			temp := strings.Replace(*renameFormat, "{random1}", strconv.Itoa(random1Val), -1)
			newFileName := strings.Replace(temp, "{random2}", strconv.Itoa(random2Val), -1)
			dstJoinList := []string{*dstFilePath, newFileName}
			newDstFilePath := strings.Join(dstJoinList, "/")
			// fmt.Println("[*] 输出随机生成的文件: ", newDstFilePath)
			go copyFile(c, *srcFilePath, newDstFilePath)
		}
		<-c
		fmt.Println("[+] 已经完成目标: 共计生成", *generateFileNumber, "个文件")
	} else {
		fmt.Println("[-] 原始文件不存在, 请检查: ", *srcFilePath)
	}
	elapsed := time.Since(t1)
	fmt.Println("运行耗时: ", elapsed)
}
```

# 使用效果如下
## 没有使用gorutine之前
![](images/snipaste_20171231_163523.png) 

## 使用gorutine后的并发效果
![](images/snipaste_20171231_163646.png) 

## 异常处理
![](images/snipaste_20171231_164046.png) 


第一次自己写的代码遇到这个文件打开数过多的问题，还有点小激动呢！查看下系统当前的**ulimit open files**配置值
![](images/snipaste_20171231_164447.png)



果断手动改大些（个人笔记本，不会经常进新大量IO操作，就临时改下吧）**ulimit -n 65535**
![](images/snipaste_20171231_164748.png) 



修改后的使用效果就是上面的并发测试那个图啦！

# 后续
其实还有一个使用了sync的版本，但是测试效果和没有使用并发的测试结果一样，甚至还要差一些，还是不懂的太多，留个坑，等搞明白了再来补充。
![](images/snipaste_20171231_165138.png) 



相关伪代码逻辑如下

```go
func main() {
    wg := sync.WaitGroup{}
	wg.Add(*generateFileNumber)
    for count := 0; count < *generateFileNumber; count++ {
        dosomething(&wg)
    }
    wg.Wait()
}
func dosomething(wg *sync.WaitGroup) {
    do
    wg.Done()
}
{% endcodeblock %}
```

-----

2017年12月31日 于 南京  
[Email](toddlerya@qq.com)  
[GitHub](https://github.com/toddlerya)