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

  

## 步骤图解

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

- 创建项目文件夹并进入这个文件夹中

  ```
  mkdir my-network
  cd my-network
  ```

  

## 创建crypto-config.yaml

- 创建crypto-config.yaml并打开编辑

  ```
  touch crypto-config.yaml
  vim crypto-config.yaml
  ```

  

- 复制并粘贴以下内容

  ```
  OrdererOrgs:
    - Name: Orderer
      Domain: example.com
      EnableNodeOUs: true
      Specs:
        - Hostname: orderer
        - Hostname: orderer2
        - Hostname: orderer3
        - Hostname: orderer4
        - Hostname: orderer5
  
  PeerOrgs:
    - Name: Org1
      Domain: org1.example.com
      EnableNodeOUs: true
      Template:
        Count: 2
      Users:
        Count: 1
    - Name: Org2
      Domain: org2.example.com
      EnableNodeOUs: true
      Template:
        Count: 2
      Users:
        Count: 1
  
  ```

  

## 创世区块配置和生成

- 配置生成尝试区块

  ```
  export PATH=/root/fabric-samples/bin:$PATH
  cryptogen generate --config ./crypto-config.yaml
  ```

  

## 配置docker compose文件和基础文件的加入

- 创建docker-compose-cli.yaml文件

  ```
  touch docker-compose-cli.yaml
  vim docker-compose-cli.yaml
  ```

- 配置docker-compose-cli.yaml文件

  ```
  version: '2'
  
  volumes:
    orderer.example.com:
    peer0.org1.example.com:
    peer1.org1.example.com:
    peer0.org2.example.com:
    peer1.org2.example.com:
  
  networks:
    byfn:
  
  services:
  
    orderer.example.com:
      extends:
        file:   base/docker-compose-base.yaml
        service: orderer.example.com
      container_name: orderer.example.com
      networks:
        - byfn
  
    peer0.org1.example.com:
      container_name: peer0.org1.example.com
      extends:
        file:  base/docker-compose-base.yaml
        service: peer0.org1.example.com
      networks:
        - byfn
  
    peer1.org1.example.com:
      container_name: peer1.org1.example.com
      extends:
        file:  base/docker-compose-base.yaml
        service: peer1.org1.example.com
      networks:
        - byfn
  
    peer0.org2.example.com:
      container_name: peer0.org2.example.com
      extends:
        file:  base/docker-compose-base.yaml
        service: peer0.org2.example.com
      networks:
        - byfn
  
    peer1.org2.example.com:
      container_name: peer1.org2.example.com
      extends:
        file:  base/docker-compose-base.yaml
        service: peer1.org2.example.com
      networks:
        - byfn
  
    cli:
      container_name: cli
      image: hyperledger/fabric-tools:$IMAGE_TAG
      tty: true
      stdin_open: true
      environment:
        - SYS_CHANNEL=$SYS_CHANNEL
        - GOPATH=/opt/gopath
        - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
        #- FABRIC_LOGGING_SPEC=DEBUG
        - FABRIC_LOGGING_SPEC=INFO
        - CORE_PEER_ID=cli
        - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
        - CORE_PEER_LOCALMSPID=Org1MSP
        - CORE_PEER_TLS_ENABLED=true
        - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
        - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
        - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
        - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
      working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
      command: /bin/bash
      volumes:
          - /var/run/:/host/var/run/
          - ./../chaincode/:/opt/gopath/src/github.com/chaincode
          - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
          - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
          - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
      depends_on:
        - orderer.example.com
        - peer0.org1.example.com
        - peer1.org1.example.com
        - peer0.org2.example.com
        - peer1.org2.example.com
      networks:
        - byfn
  
  ```

  

## 创建通道

## 加入通道

## 安装链码

## 实例化链码

## 查询链码

## 调用及查询链码

# 链码的编写

## 链码开发环境搭建

## 链码架构

