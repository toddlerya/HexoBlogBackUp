---
title: Golang学习（一）
date: 2017-12-24 12:11:49
lastmod:
tags: [Golang, 工具]
categories:
 - 造轮子

typora-root-url: ..
typora-copy-images-to: ../images
---

# 闲扯

去北京参加Top100学习到现在一个多月过去了，时间过得好快，转眼2017年只剩下最后一周了，不由得感叹时间就像指缝的流沙。
再加上最近中年程序员不堪重负的各种新闻，感觉自己距离那一天也没多远了，在此之前努力提高自己的姿势水平吧，要没时间了。

在Top100听到了很多讲师提到了Golang，结合之前看过的左耳朵耗子的文章[《GO语言、DOCKER 和新技术》](https://coolshell.cn/articles/18190.html)，决定要学习了解下Golang了。
最近半个月断断续续看了Golang的一些教程[《GO 语言简介（上）— 语法》](https://coolshell.cn/articles/8460.html)、[《GO 语言简介（下）— 特性》](https://coolshell.cn/articles/8489.html)、无闻老师的[《Go 编程基础》](https://github.com/Unknwon/go-fundamental-programming)，只是对Golang有了一点点初步的了解。
学习一门语言最重要的还是要撸代码啊，要动手写个小工具试试手。
恰巧昨天加班时帮同事写了个小工具，感觉用Python的性能不够好，而且想要高性能还要依赖gevent这种第三方库，不方便部署，于是想到用Golang可以来试下，写完编译下给同事用就好啦！昨晚回到家吃过饭陪女朋友玩了会，从十点开始撸代码到凌晨一点，终于写出一个小demo，特此记录下年轻人的第一个Golang小程序。

---
# 正文
工具需求是有一个样例zip包，zip包里面有一些bcp数据、xml数据、bjson数据、图片、视频什么的。
zip包的命名规范如下：
​    AAA-BBB-1514090969-CCC_DDD_21234.zip
我们要将zip包的1514090969【绝对秒数】与21234【随机序列】进行替换，随机生成大量的zip包副本，发送到某个ETL输入目录，暂且不管zip包里面的内容（其实zip包内的文件也要随机生成）。
之前同事用Shell写的，没看他代码怎么实现的，不过肯定有问题，一小时才生成了三万个样例zip包，远远达不到压测的要求。
于是我花了十几分钟调用Python的gevent模块帮他重新用Python写了一遍，性能瞬间爆炸。
但是部署gevent比较麻烦（公司和客户都是内网环境，pip是没法用的，只能手动安装所需的第三方包，而且公司大部分操作系统还是Redhat AS6U3，默认的Python是2.6版本，各种不方便），就想到了如果用Golang写一遍是否性能很好，编译后提供二进制文件直接运行就可以了，于是就有了下面的代码。
# 代码
```Golang
/*
 * User: toddlerya
 * Date: 2017/12/23
 * ds接入模块加压工具
 */

package main

import (
	"flag"
	"fmt"
	"io"
	"math/rand"
	"os"
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

func copyFile(src, des string) (w int64, err error) {
	srcFile, err := os.Open(src)
	if err != nil {
		fmt.Println(err)
	}
	defer srcFile.Close()

	desFile, err := os.Create(des)
	if err != nil {
		fmt.Println(err)
	}
	defer desFile.Close()

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
	srcFilePath := flag.String("s", "/home", "输入原始文件路径")
	dstFilePath := flag.String("d", "/tmp", "目标输出路径")
	renameFormat := flag.String("f", "abc-{random1}-456_780_{random2}.zip", "参数原始文件替换格式")
	random1Start := flag.Int("r1s", 100, "随机参数1的起始值")
	random1End := flag.Int("r1e", 999, "随机参数1的截止值")
	random2Start := flag.Int("r2s", 100000, "随机参数2的起始值")
	random2End := flag.Int("r2e", 999999, "随机参数2的截止值")
	generateFileNumber := flag.Int("g", 10000, "需要生成的文件个数")
	// taskQueueSize := flag.Int("t", 100, "每次任务队列生成文件个数")
	// interval := flag.Int("i", 3, "任务运行间隔[单位秒]")
	flag.Parse()
	fmt.Printf("[srcFilePath]          :%s\n", *srcFilePath)
	fmt.Printf("[dstFilePath]          :%s\n", *dstFilePath)
	fmt.Printf("[renameFormat]         :%s\n", *renameFormat)
	fmt.Printf("[random1Start]         :%v\n", *random1Start)
	fmt.Printf("[random1End]           :%v\n", *random1End)
	fmt.Printf("[random2Start]         :%v\n", *random2Start)
	fmt.Printf("[random2End]           :%v\n", *random2End)
	fmt.Printf("[generateFileNumber]   :%v\n", *generateFileNumber)
	// fmt.Printf("[taskQueueSize]        :%v\n", *taskQueueSize)
	// fmt.Printf("[interval](second)     :%v\n", *interval)
	fmt.Println("=======================================")
	if judgeExists(*srcFilePath) {
		fmt.Println("[+] 开始随机生成目标数据, 请注意", *dstFilePath, "目录是否有数据生成\t", "计划生成", *generateFileNumber, "个文件Orz")
		count := 0
		for {
			if count >= *generateFileNumber {
				fmt.Println("[+] 已经完成目标: 共计生成", *generateFileNumber, "个文件")
				return
			}
			random1Val := generateRandomNumber(*random1Start, *random1End)
			random2Val := generateRandomNumber(*random2Start, *random2End)
			temp := strings.Replace(*renameFormat, "{random1}", strconv.Itoa(random1Val), -1)
			newFileName := strings.Replace(temp, "{random2}", strconv.Itoa(random2Val), -1)
			dstJoinList := []string{*dstFilePath, newFileName}
			newDstFilePath := strings.Join(dstJoinList, "/")
			fmt.Println("[*] 输出随机生成的文件: ", newDstFilePath)
			copyFile(*srcFilePath, newDstFilePath)
			count++
		}
	} else {
		fmt.Println("[-] 原始文件不存在, 请检查: ", *srcFilePath)
	}
}
```
# 使用效果如下
## 测试运行
![](/images/snipaste_20171224_011006.png)

## 异常处理
![](/images/snipaste_20171224_011006.png)

## 遗留BUG 
但是在阿里云的测试中发现了程序的BUG，df看磁盘剩余空间还有很多，但是df -i查看发现磁盘Inode被耗尽了导致无法复制文件，可是是程序并没有停止复制，异常退出，这个BUG以后再修改：
![](/images/snipaste_20171224_134323.png)

# 后续
因为还没学到Golang的并发，所以这个demo只是简单的循环执行，后续学会了并发再来更新T_T

-----

2017年12月24日 于 南京  
[Email](toddlerya@qq.com)  
[GitHub](https://github.com/toddlerya)