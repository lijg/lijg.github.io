---
title: A Tour of Verification
date: 2012-03-22 20:44:41
tags:
- Verification
categories:
- ASIC
---

#1 验证流程
* 需求
* 测试点
* 环境
* 用例
* 覆盖率

#2 需求
* 设计人员 -> 可实现性
* 验证人员 -> 可验证性

#3 测试点分类
* 接口
* 功能
* 结构体
* 稳定性
* 可靠性

#4 测试点方法
* 等价类
* 边界值
* 错误推测
* 因果图
* 场景

#5 方式方法
* Formal
* Random
* Assertion

#6 验证语言与方法
* 验证语言
    * SystemVerilog / SystemC
* 验证方法
    * OVM(Mentor、Candence)
    * VMM(ARM、Synopsys)
    * UVM(OVM + VMM)

#7 环境组成
* DUT / DUV
* BFM (Driver / Monitor)
* Generator
* Reference Model
* Checker (ScoreBoard)
* Channel ( FIFO )
* ENV / SubENV
* TestCase

#8 覆盖率
* Function Coverage
* Code Coverage
    * line
    * cond
    * fsm
    * tgl
    * branch
    * assert
    * path
    