# 前言

时隔一年多，当联盟链重回大众视野后，我也在工作需求中重拾了联盟链，当然超级账本也就成了第一个要吃透的技术。废话不多说，以下开始我对超级账本的理解，不保证都对，遇到错误也麻烦大家指正，非常感谢，共同学习。

# 基础知识

## 基础简介

## 概念名词

## 工作流程

# 创建我的第一个网络

## 搞定环境

- 本篇文章所有操作均在ubuntu18.04下完成
- 安装cURL
- 安装git
- 安装docker-ce
- 安装docker compose

## 预制代码

- 安装hyperledger的docker镜像

  ```
  curl -sSL http://bit.ly/2ysbOFE | bash -s
  ```

  

- 切换到fabirc-sample/first-network

  ```
  cd fabirc-sample/first-network
  ```

  

- 生成创始区块,channel的配置文件

  ```
  ./byfn.sh -m generate
  ```

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-9b7a6e34d7186e63.png)

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-d788485282501098.png)

## 开始

- 启动fabric的测试网络

  ```
  ./byfn.sh -m up
  ```

  

## 步骤详解

- 启动六台节点

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-7440db5feef1fc72.png)

- 创建channel

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-1562662cdfac5970.jpg)

- 让每个peer加入channel

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-23a80a30d547ad25.png)

- 更新锚节点

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-b048ed7557668047.png)

- 安装链码

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-e595091bc44dc99f.png)

- 实例化链码

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-550ae95902ae5074.png)

- 执行链码

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-f4bd841e7542a395.png)

- 结束

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-e7f034bc0a20fdb5.png)

- 其他常用命令

  ```
  ./byfn.sh -m down
  ./byfn.sh -m up
  ```

  

# 手动搭建我的第一个网络

## 新建项目

## 创建crypto-config.yaml

## 创世区块配置和生成

## 配置docker compose文件和基础文件的加入

## 创建通道

## 加入通道

## 安装链码

## 实例化链码

## 查询链码

## 调用及查询链码

# 链码的编写

## 链码开发环境搭建

## 链码架构

