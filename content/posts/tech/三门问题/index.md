---
title: "三门问题的贝叶斯解释以及JAVA复现方式"
date: 2024-03-01T19:00:00+08:00
lastmod: 2024-03-01T19:00:00+08:00
author: ["realMarkYang"]
keywords: 
- 贝叶斯
- JAVA
categories:   #类别 
- Java
- tech
- 贝叶斯
tags: 
- Java
- tech
- 贝叶斯
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true
reward: false # 打赏
mermaid: false # 是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true # 顶部显示路径
cover:
    image: "" # 图片路径：posts/tech/123/123.png
    caption: "" # 图片底部描述
    alt: ""
    relative: false
---


### 一.前言

在上个世纪90年代，一个名为《三门问题（Monty Hall problem）》亦称为《蒙提霍尔悖论》的概率学问题引发了广泛的讨论，在90年代的十年间，40多种学术刊物发表了关于这一问题超过75篇论文。然而时隔多年再次看待这一问题依然十分有趣，同时我将以JAVA语言的方式复现这一实验并验证结果。本文将要解决的问题如下：

* 如何从贝叶斯理论视角解释三门问题
* 如何使用JAVA验证推断结果
* 人们的直觉为什么会出错

### 二.三门问题的贝叶斯解释

> 三门问题（Monty Hall problem大致出自美国的电视游戏节目Let’s Make a Deal。问题名字来自该节目的主持人蒙提·霍尔（Monty Hall）。参赛者会看见三扇关闭了的门，其中一扇的后面有一辆汽车，选中后面有车的那扇门可赢得该汽车，另外两扇门后面则各藏有一只山羊。当参赛者选定了一扇门，但未去开启它的时候，节目主持人开启剩下两扇门的其中一扇，露出其中一只山羊。主持人其后会问参赛者要不要换另一扇仍然关上的门。

**问题是：换另一扇门会否增加参赛者赢得汽车的概率？**

在解决这个问题之前我们先复习一下贝叶斯公式以及一些基本的概率计算公式

概率乘法公式 ：$$ P({B \over A}) = { P(AB) \over P(A) }$$

贝叶斯公式：$$P({B \over A}) = {P({A \over B})P(B) \over P(A)} $$

全概率公式：$$ P(B)=\sum_{i=1}^{n} P(A_i)P({B \over A_i}) $$

---

推导过程如下：

设有门：1                  2                         3

1.若玩家选择1号门，1号内若是汽车那么主持人打开2号门，3号门概率为$${1 \over 2}$$

2.若1号门内不是汽车，则主持人打开2，3号门概率为0与1（2，3号门内有一辆汽车，这里根据题设主持人是知道门里的内容的）

以下标为i表示玩家选择的门

  * 设事件A为第i扇门后有汽车，事件B主持人选择3号门

$$P(A_i) = P(A_1)=P(A_2)=P(A_3)={1 \over 2} $$

$$P(B)=P(A_1)P({B \over A_1})+P(A_2)P({B \over A_2})+P(A_3)P({B \over A_3})$$   全概率公式

$$ P({B \over A_1})={1 \over 2} $$

$$ P({B \over A_2})={1} $$

$$ P({B \over A_3})={0} $$

$$P(B)={1 \over 6}+{1 \over 3}={1 \over 2}$$       这时求出了主持人选择3号门的概率，接下来运用贝叶斯公式进行求解每号门背后有小汽车的概率

故 当主持人打开3号门后，i号门有汽车的概率为：

$$ P({A_1 \over B})= {{P({B \over A_1})P(A_1)} \over P(B) }= {1 \over 3}$$

$$ P({A_2 \over B})= {{P({B \over A_2})P(A_2)} \over P(B) }= {2 \over 3}$$

至此关于三门问题的贝叶斯理论推导已完成，得出选手换门获奖概率为$$2 \over 3$$，不换门为$$1 \over 3$$

那么，这个结果究竟对不对呢，我们来用JAVA模拟这一过程

### 三. 三门问题的JAVA复现

``` java
import java.util.Random;
import java.util.Scanner;

public class identify {
    private static final int NUM_DOORS = 3;// 定义门的数量
    private static Random random = new Random();// 随机数生成器
    private static final int percent = 100;// 定义百分比

    public static void main(String[] args) {
        int stayWinCar = 0;// 定义不更换门得到车的游戏次数
        int switchWinCar = 0;// 定义更换门后得到车的游戏次数
        System.out.print("请输入实验次数:");
        Scanner scanner = new Scanner(System.in);
        int totalGames  = scanner.nextInt();
        for (int i = 0; i < totalGames; i++) {
            int carDoor = random.nextInt(NUM_DOORS);// 随机一扇门后有汽车
            int firstChoice = random.nextInt(NUM_DOORS);// 参赛者第一次选择的门

            // 主持人打开另一扇有羊的门
            int openedDoor;
            do {
                openedDoor = random.nextInt(NUM_DOORS);
                // 主持人不能打开用户选择的门和有汽车的门
            } while (openedDoor == firstChoice || openedDoor == carDoor);

            // 计算不能换门，参赛者可以获得小汽车的次数
            if (firstChoice == carDoor) {
                stayWinCar++;
            }

            // 主持人打开另一扇有羊的门，独立事件——与第一次打开门无关
            int secondChoice;
            do {
                secondChoice = random.nextInt(NUM_DOORS);
                // 主持人不能打开用户选择的门和有汽车的门
            } while (secondChoice == firstChoice || secondChoice == openedDoor);

            // 计算换门后，参赛者可以获得小汽车的次数
            if (secondChoice == carDoor) {
                switchWinCar++;
            }
        }

        // 将次数转换为概率，输出结果
        double stayWinRate = stayWinCar * 1.0 / totalGames;
        double switchWinRate = switchWinCar * 1.0 / totalGames;
        System.out.println("不更换门获得小汽车的概率为：" + stayWinRate * percent + "%");
        System.out.println("更换门后获得小汽车的概率为：" + switchWinRate * percent + "%");
    }

}

```

```
请输入实验次数:100000
不更换门获得小汽车的概率为：33.096%
更换门后获得小汽车的概率为：66.904%
```

在进行了 10万次的实验后，换门得到小汽车的概率趋近于$$2 \over 3$$,这与我们在贝叶斯理论框架下推导的完全一致

### 四.人们的直觉为什么会出错

在刚开始看到这个问题的时候很多人会下意识地觉得，主持人排除一个答案后，嘉宾选中车的概率变为$$ {1 \over  2}$$,其实关于$$ {2 \over 3}$$为什么反直觉，以及为什么结果不是$$ {1 \over  2}$$之间的争论从未停止。这里我按照我所理解的三门问题从两个方面来尝试回答这个问题。

##### 1.预测行为发生的时间线对概率产生的影响

当我们使用贝叶斯预测一个事件发生的概率时，其实是一种由果溯因的过程，本例在推导过程中一直利用了主持人会选中3号门这一果，从而倒推原因 ，也就是选中三号门的概率。这种由果溯因的思路一般称为贝叶斯推断，同时预测概率这件事并不是发生在整个事件经行时，也就是说我们是站在一个静态的时间点去预测一个动态的事件。这本身就是一个很反直觉的事情，同时在人们的日常思考过程中也是只对一个事件去做出一个大致的判断和预测，而不是去预测这个事件发生的具体概率（因为预测概率需要成千上万的事件作为观测样本）。因此在单一样本中，直觉和客观概率产生了较大的偏差。

##### 2.思考角度的差异

当我们使用贝叶斯推导的时候，我们的目光始终聚焦在换门后，其余的门后有小汽车的概率。而当我们使用直觉判断的时候，很容易进入一个2选1的(两个门里那个有)思维误区，思考问题的方向也从换不换门对概率的影响转变为剩下的两扇门中有汽车的概率。而进入21世纪以来由于机器学习的出现和高级语言在数据处理方面的应用，使得我们可以快速且客观的验证这些问题，从而为这场争执划上一个完美的句号。



> 引用[根据贝叶斯公式求解三门问题 - 模糊计算士 - 博客园 (cnblogs.com)](https://www.cnblogs.com/fanlumaster/p/13723146.html)