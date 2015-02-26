---
layout: post
title: "linux skill"
description: ""
category: Linux
tags: [Linux]
---
{% include JB/setup %}

## 历史命令操作篇

* 最基本的查看历史命令 history

        history
* !n 编号为n的历史命令

        !767
* !-n  倒数第n个历史命令
* !!  上一条命令
* !keyword  查找包含该keyword的历史命令
* Ctrl + R  反向查找命令
* !$  上一条命令的最后一条参数
* !^  上一条命令的第一个参数
* :n  第n个参数
* magic-space  让历史记录表达式和参数符号立即显出原形
        
        bind Space:magic-space
* 命令前加空格，使之不计入history
   