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

再上面的项目中做下列事情

```
cd fabric-samples
```



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

- 配置生成创世区块

  ```
  ../bin/cryptogen generate --config ./crypto-config.yaml
  ```
  

## 配置configtx.yaml文件

```
touch configtx.yaml
vim configtx.yaml
```

```
---
Organizations:

    - &OrdererOrg
        
        Name: OrdererOrg
        ID: OrdererMSP

        MSPDir: crypto-config/ordererOrganizations/example.com/msp

        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"

    - &Org1

        Name: Org1MSP

        ID: Org1MSP

        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp

        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1MSP.admin')"

        AnchorPeers:

            - Host: peer0.org1.example.com
              Port: 7051

    - &Org2

        Name: Org2MSP

        ID: Org2MSP

        MSPDir: crypto-config/peerOrganizations/org2.example.com/msp

        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.peer', 'Org2MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org2MSP.admin')"

        AnchorPeers:

            - Host: peer0.org2.example.com
              Port: 9051

Capabilities:

    Channel: &ChannelCapabilities

        V1_4_3: true

        V1_3: false

        V1_1: false

    Orderer: &OrdererCapabilities
        
        V1_4_2: true

        V1_1: false

    Application: &ApplicationCapabilities
        
        V1_4_2: true
        
        V1_3: false
       
        V1_2: false
       
        V1_1: false

Application: &ApplicationDefaults

    Organizations:

    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    Capabilities:
        <<: *ApplicationCapabilities

Orderer: &OrdererDefaults

    OrdererType: solo

    Addresses:
        - orderer.example.com:7050

    BatchTimeout: 2s

    BatchSize:

        MaxMessageCount: 10

        AbsoluteMaxBytes: 99 MB

        PreferredMaxBytes: 512 KB

    Kafka:

        Brokers:
            - 127.0.0.1:9092

    EtcdRaft:

        Consenters:
            - Host: orderer.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
            - Host: orderer2.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
            - Host: orderer3.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
            - Host: orderer4.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
            - Host: orderer5.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt

    Organizations:

    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        # BlockValidation specifies what signatures must be included in the block
        # from the orderer for the peer to validate it.
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"

Channel: &ChannelDefaults

    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    Capabilities:
        <<: *ChannelCapabilities

Profiles:

    TwoOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
    TwoOrgsChannel:
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
            Capabilities:
                <<: *ApplicationCapabilities

    SampleDevModeKafka:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: kafka
            Kafka:
                Brokers:
                - kafka.example.com:9092

            Organizations:
            - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
            - <<: *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                - *Org1
                - *Org2

    SampleMultiNodeEtcdRaft:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: etcdraft
            EtcdRaft:
                Consenters:
                - Host: orderer.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
                - Host: orderer2.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                - Host: orderer3.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
                - Host: orderer4.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
                - Host: orderer5.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
            Addresses:
                - orderer.example.com:7050
                - orderer2.example.com:7050
                - orderer3.example.com:7050
                - orderer4.example.com:7050
                - orderer5.example.com:7050

            Organizations:
            - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
            - <<: *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                - *Org1
                - *Org2

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

- 创建base相关文件

  ```
  mkdir base
  cd base
  ```

  - 创建docker-compose-base.yaml文件

    ```
    touch docker-compose-base.yaml
    vim docker-compose-base.yaml
    ```

    ```
    # Copyright IBM Corp. All Rights Reserved.
    #
    # SPDX-License-Identifier: Apache-2.0
    #
    
    version: '2'
    
    services:
    
      orderer.example.com:
        container_name: orderer.example.com
        extends:
          file: peer-base.yaml
          service: orderer-base
        volumes:
            - ../channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
            - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
            - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls
            - orderer.example.com:/var/hyperledger/production/orderer
        ports:
          - 7050:7050
    
      peer0.org1.example.com:
        container_name: peer0.org1.example.com
        extends:
          file: peer-base.yaml
          service: peer-base
        environment:
          - CORE_PEER_ID=peer0.org1.example.com
          - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
          - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
          - CORE_PEER_CHAINCODEADDRESS=peer0.org1.example.com:7052
          - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
          - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.example.com:8051
          - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
          - CORE_PEER_LOCALMSPID=Org1MSP
        volumes:
            - /var/run/:/host/var/run/
            - ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp
            - ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls:/etc/hyperledger/fabric/tls
            - peer0.org1.example.com:/var/hyperledger/production
        ports:
          - 7051:7051
    
      peer1.org1.example.com:
        container_name: peer1.org1.example.com
        extends:
          file: peer-base.yaml
          service: peer-base
        environment:
          - CORE_PEER_ID=peer1.org1.example.com
          - CORE_PEER_ADDRESS=peer1.org1.example.com:8051
          - CORE_PEER_LISTENADDRESS=0.0.0.0:8051
          - CORE_PEER_CHAINCODEADDRESS=peer1.org1.example.com:8052
          - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:8052
          - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.example.com:8051
          - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
          - CORE_PEER_LOCALMSPID=Org1MSP
        volumes:
            - /var/run/:/host/var/run/
            - ../crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp:/etc/hyperledger/fabric/msp
            - ../crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls:/etc/hyperledger/fabric/tls
            - peer1.org1.example.com:/var/hyperledger/production
    
        ports:
          - 8051:8051
    
      peer0.org2.example.com:
        container_name: peer0.org2.example.com
        extends:
          file: peer-base.yaml
          service: peer-base
        environment:
          - CORE_PEER_ID=peer0.org2.example.com
          - CORE_PEER_ADDRESS=peer0.org2.example.com:9051
          - CORE_PEER_LISTENADDRESS=0.0.0.0:9051
          - CORE_PEER_CHAINCODEADDRESS=peer0.org2.example.com:9052
          - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:9052
          - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org2.example.com:9051
          - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org2.example.com:10051
          - CORE_PEER_LOCALMSPID=Org2MSP
        volumes:
            - /var/run/:/host/var/run/
            - ../crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp:/etc/hyperledger/fabric/msp
            - ../crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls:/etc/hyperledger/fabric/tls
            - peer0.org2.example.com:/var/hyperledger/production
        ports:
          - 9051:9051
    
      peer1.org2.example.com:
        container_name: peer1.org2.example.com
        extends:
          file: peer-base.yaml
          service: peer-base
        environment:
          - CORE_PEER_ID=peer1.org2.example.com
          - CORE_PEER_ADDRESS=peer1.org2.example.com:10051
          - CORE_PEER_LISTENADDRESS=0.0.0.0:10051
          - CORE_PEER_CHAINCODEADDRESS=peer1.org2.example.com:10052
          - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:10052
          - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org2.example.com:10051
          - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org2.example.com:9051
          - CORE_PEER_LOCALMSPID=Org2MSP
        volumes:
            - /var/run/:/host/var/run/
            - ../crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/msp:/etc/hyperledger/fabric/msp
            - ../crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls:/etc/hyperledger/fabric/tls
            - peer1.org2.example.com:/var/hyperledger/production
        ports:
          - 10051:10051
    
    ```

  - 创建peer-base.yaml文件

    ```
    touch peer-base.yaml
    vim peer-base.yaml
    ```

    ```
    # Copyright IBM Corp. All Rights Reserved.
    #
    # SPDX-License-Identifier: Apache-2.0
    #
    
    version: '2'
    
    services:
      peer-base:
        image: hyperledger/fabric-peer:$IMAGE_TAG
        environment:
          - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
          # the following setting starts chaincode containers on the same
          # bridge network as the peers
          # https://docs.docker.com/compose/networking/
          - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_byfn
          - FABRIC_LOGGING_SPEC=INFO
          #- FABRIC_LOGGING_SPEC=DEBUG
          - CORE_PEER_TLS_ENABLED=true
          - CORE_PEER_GOSSIP_USELEADERELECTION=true
          - CORE_PEER_GOSSIP_ORGLEADER=false
          - CORE_PEER_PROFILE_ENABLED=true
          - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
          - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
          - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
        working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
        command: peer node start
    
      orderer-base:
        image: hyperledger/fabric-orderer:$IMAGE_TAG
        environment:
          - FABRIC_LOGGING_SPEC=INFO
          - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
          - ORDERER_GENERAL_GENESISMETHOD=file
          - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
          - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
          - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
          # enabled TLS
          - ORDERER_GENERAL_TLS_ENABLED=true
          - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
          - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
          - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
          - ORDERER_KAFKA_TOPIC_REPLICATIONFACTOR=1
          - ORDERER_KAFKA_VERBOSE=true
          - ORDERER_GENERAL_CLUSTER_CLIENTCERTIFICATE=/var/hyperledger/orderer/tls/server.crt
          - ORDERER_GENERAL_CLUSTER_CLIENTPRIVATEKEY=/var/hyperledger/orderer/tls/server.key
          - ORDERER_GENERAL_CLUSTER_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
        working_dir: /opt/gopath/src/github.com/hyperledger/fabric
        command: orderer
    ```

    ```
    cd ../
    ```

## 创建通道

```
../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
 ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
 ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org2MSP
```



## 启动网络

```
export IMAGE_TAG=1.4.3
docker-compose -f docker-compose-cli.yaml up -d
```



## 创建通道

```
docker exec -it cli bash
```



```
CORE_PEER_LOCALMSPID="Org1MSP" CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin\@org1.example.com/msp peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```



## 加入通道

```
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer channel join -b mychannel.block

export CORE_PEER_ADDRESS=peer1.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer channel join -b mychannel.block

export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
peer channel join -b mychannel.block


export CORE_PEER_ADDRESS=peer1.org2.example.com:7051
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
peer channel join -b mychannel.block
```



## 安装链码

```
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node


export CORE_PEER_ADDRESS=peer1.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node

export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node


export CORE_PEER_ADDRESS=peer1.org2.example.com:7051
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node

```



## 实例化链码

```
peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```



## 查询链码

```
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
```



## 调用及查询链码

```
peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}' --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

```
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
```



# 链码的编写

## 链码开发环境搭建

## 链码架构

