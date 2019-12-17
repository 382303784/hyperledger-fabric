# 前言

时隔一年多，当联盟链重回大众视野后，我也在工作需求中重拾了联盟链，当然超级账本也就成了第一个要吃透的技术。废话不多说，以下开始我对超级账本的理解，不保证都对，遇到错误也麻烦大家指正，非常感谢，共同学习。

# 系统架构

## 系统架构

![avatar](https://upload-images.jianshu.io/upload_images/13765375-3c26678f09a02fe4.jpg)

- 应用程序层

  - API：fabric为应用程序开发提供了gRPC接口。
  - SDK：在API的基础上官方又针对不同语言封装了SDK，如node.js、go和java。
  - 事件：fabric采用异步通信的模式进行开发，可以再连码中定义事件进行监听，当某个事件被触发时我们就可以执行预先定义的回调函数了。

- 应用程序层与底层交互的媒介

  - 身份：身份系统是所有其他操作的拦路虎，一个不能被验明身份的请求是不会被底层系统认可的。任何请求的处理流程都是先通过请求方的签名验证，通过以后才会进行下一步操作。应用层的身份管理模块是依托于底层的成员服务。
  - 账本管理：账本管理其实是对交易区块的查询，fabric提供了很多种的查询方式，比如按照区块高度进行查询，还可以根据区块hash去查询，也可以根据交易ID来查询交易或区块，特写强调可以使用交易ID来查询区块。账本管理还可以根据通道的名称获取账本的一些信息，fabric中账本是通过通道进行隔离的，这种隔离不仅仅体现在逻辑上，在物理存储上也是分割的。比如区块存储，fabric的区块存储是以文件块的方式，根据通道名划分不同的文件夹，这样就做到了物理隔离。在存储状态数据的时候，因为使用了levelDB，而它是没有更多的空间可以选择的，不同账本的状态是通过组合key的方式划分的。账本管理模块都是提供的区块链查询方法。
  - 交易管理：上层的应用程序通过交易管理模块，将交易提交到交易背书节点，在获得到交易背书以后，他将交易提供给交易排序节点进行排序。排完序的交易会被打包成区块进行分发，这样交易就扩散到了整个网络。交易管理模块主要是对区块链底层进行修改的。
  - 智能合约：主要是做智能合约的安装、初始化和升级等操作。智能合约是函数的声明，交易是函数的调用。

- 区块链底层

  - 成员服务：成员服务就是通常意义上联盟链才有的准入控制。他利用PKI体系（公钥基础设施）和CA系统（证书机构）提供了一些诸如注册登录、身份验证这样的功能。前面所指的成员和注册登录并不是传统意义上的账号密码形式，也不是为每个应用程序在区块链上建立身份，而是指的能与区块链底层建立交互的身份管理。一个应用程序也许从头到尾只需要一个身份就可以完成所有与区块链交互的功能。这样的成员服务中，每一个成员都包含多个证书，比如用于用户身份的注册证书，交易签名的交易证书，加密传输所用的TLS证书。
  - 共识服务：整个共识服务分为三个阶段。首先是客户端，它向背书节点发送一个提案，随后交易背书节点模拟执行后，返回背书结果以及签名，随后客户端将背书好的交易交给排序节点进行排序，之后由排序节点生成区块，然后向全网进行广播。记账节点收到浙西广播后，先验证这些交易的正确性，都验证通过了在存入本地的账本中。这个过程用到了两种通信协议，排序节点与组织的锚节点之间使用了gRPC协议。在组织内部使用了gossip协议使区块扩散。这几个阶段加起来就构成了共识服务。

  - 链码服务：提供了安全的且可隔离的交易执行环境，用于确保用户数据的隔离，这里使用的是docker。也有几点不足：首先是每个链码都是独立的容器，这也就意味着要为每一个链码提供完整的环境，这样就造成打包镜像比较大。在一个就是现阶段来链码容器编译都是直接与docker进行通讯的，这就与容器编排工具例如K8S结合的时候，显得格格不入，不能被K8S管理，拓展性没那么好。
  - 安全及密码服务：主要定义一些公钥、私钥、签名、加密、解密这样的基础功能。默认实现一套国际通用的密码服务，在国内使用要研究切换到使用国密。

## 网络拓补

![avatar](https://upload-images.jianshu.io/upload_images/13765375-ae6c0846e51fc422.jpg)

- 客户端：应用程序/SDK/命令行工具。并不是fabric的底层节点，他是应用层与底层交互的工具。不能够独立存在，必须与peer节点和orderer建立连接才能发挥作用，比如连接到orderer节点进行通道创建，连接到peer节点进行交易模拟执行等等。
- Peer：Anchor/Endorser/Committer。一个组织能可能有多个peer节点但是只有一个Anchor节点。Anchor节点可以翻译成锚节点或者叫主节点，他是组织内部唯一与orderer节点通信的节点。主节点可以在网络启动的时候由最初始的配置指定。在示例项目中有一步骤就是更新组织的锚节点配置，就是在初始化网络的时候指定组织对外的主节点。如果组织中的主节点因为一些原因下线了，那么这个组织将会自动选举出新的主节点与orderer进行连接，这个过程是自动化的。Endorser节点也叫背书节点，可以理解为担保，这个节点的主要作用就是为交易做担保的。fabric中的共识分为三步，第一步就是客户端发送交易到背书节点进行交易模拟执行，背书节点执行完以后对执行结果签名背书。背书节点它不是一个固定的节点类型，他是跟fabric中智能合约绑定的，每一个智能合约被安装在区块链上的时候，都会设置其专有的背书策略，指定该智能合约的交易由哪些背书节点背书以后才能是有效的，也就是说在背书节点上才能运行智能合约。如果某个节点不是背书策略里的一员，那它就只能是一个普通的peer节点，也就是Committer节点，翻译是记账节点。所有的peer节点都是记账节点，无论它本身是主节点还是背书节点。记账节点的主要功能是用于验证从orderer节点接收的区块，验证区块的有效性以及交易的有效性，验证完以后记录到本地的账本中，如果交易有效同时更改区块链上的状态数据，这就是我们平时所理解的记账。peer节点的三种类型不是排斥的，一个节点他可能同时是主节点、背书节点和记账节点，他也可能是其中的两种角色，也可能是记账节点这一种。
- Orderer:排序节点。他主要有两个功能。第一个功能是从全网的客户端接收交易，然后将交易按照一定的规则排序。第二个功能是将排序好的交易按照固定的时间间隔打包成区块，然后分发给其他组织的主节点。官方现在实现了两种类型的排序，一种是solo，这种模式下整个网络只有一个排序节点，他收到交易的顺序就是整个网络中排好的顺序，这种模式仅仅适合开发和测试的时候使用，正式商用环境中是不行的，因为它的单点故障一点出现整个网络接瘫痪了。第二种是kafka，它将整个网络中的排序功能交给了kafka集群。在它的实现中每个orderer是kafka的生产者也是消费者。生产者把从客户端接收的交易发送给kafka集群，同时消费者又从kafka集群中获取交易，这样获得的交易就已经是排好序的了。但是这两种类型的排序都不是那么的区块链，一般情况下我们所说的区块链共识都是支持拜占庭容错的，但这两种都不支持。
- CA:证书颁发机构。主要作用是建立区块链的身份是否是有效的，是否是合法的。只有被CA认可的身份才可以在区块链上交易，否则就会被拒绝。在fabric中，CA机构是可以选择的，可要可不要，可以选用fabric提供的CA组件，也可以选用第三方的。

## 交易流程

![avatar](https://upload-images.jianshu.io/upload_images/13765375-61da602ef38032e6.jpg)

- 提交交提案：应用程序也就是客户端节点在本地构造好交易提案后，就选择背书节点进行提交。背书节点的选择是跟智能合约相关的，智能合约在安装的时候必须指定背书节点，如果一个智能合约要求两个节点进行背书，那么应用程序就要把交易发送到至少两个背书节点。如果因为网络或者其他什么原因，背书节点一直没有结果返回，那么就必须将交易发送给第三个背书节点，如果不这样该笔交易就是无效的。提交交提案的顺序一般是没有要求的，如果一切正常的话所有背书节点都会返回一样的结果，只是签名不一样。
- 模拟执行交易提案并签名：背书节点在收到前面发送的交易后会做一些验证检查方面的工作，比如对交易提案格式的验证、该交易以前是否被提交过、交易发送者的签名是否正确，该发送者是否有权利提交这笔交易等等。如果一切都通过以后，背书节点会把交易发送给智能合约隔离执行，执行完后会返回执行结果，背书节点对这个结果进行签名，也就是背书，然后返回给应用程序。需要注意的是这个过程是模拟执行的不会产生任何持久化操作。
- 返回模拟执行结果+提交交易+交易排序并结块+广播区块：背书节点将执行结果和签名返回个应用程序也就是客户端。客户端收到以后，还是先对消息进行验证，如果验证通过才会有后续的操作。一般会有两种处理结果。如果该交易是个查询交易，因为查询并不会对账本产生状态的影响，所以只要验证签名状态通过以后，客户端就会拿着这个返回值作为下一步业务逻辑的依据，可能会导致下一个交易的产生，此时交易流程就结束了。如果该交易是一个写交易，情况就会复杂很多。客户端节点必须收集到足够多的背书结果，然后将交易提案、背书结果再加上自己的签名合起来成为一个交易，然后将交易发送给排序节点进行排序，这一步比较简单，只需要确保所有的背书结果都一致。如果提交了背书结果不一致的交易，排序节点是不会发现的，因为排序节点并不会读取交易的内容，只是会接收网络中所有的交易，然后按照提前规定好的排序逻辑进行排序，将排好序的交易进行打包然后广播给其他组织的主节点。
- 保存区块更新世界状态：主节点收到交易以后，在记账的过程中回去验证这个比交易是否是有效的，如果无效会被标记成无效交易，这个交易不会被丢弃也会存在区块链账本中，并不会更新状态数据库。
- 同步区块+在组织内保存区块更新世界状态：同上。

# 共识机制

- Orderer
  - 交易排序
    - 目的：保证系统交易顺序一致性（有限状态机）
    - solo：单节点排序，所见即所得
    - kafka：外置消息队列，保证一致性
  - 区块分发
    - 中间状态区块（临时区块）
    - 有效交易和无效交易
  - 多通道数据隔离

![avatar](https://upload-images.jianshu.io/upload_images/13765375-a022ca2888ebd10a.jpg)

首先所有的应用程序都向排序节点提交交易，客户端进行交易提交的时候会指定一个通道，表明该交易是发往哪一个通道的，排序节点收到交易以后会按照通道进行拆分，然后进行排序。也就是说排序不是全局排序，而是按照每一个通道单独排序，最后分别组装成区块发往主节点。通道之间是相互隔离的，每个节点可以加入多个通道。

# 账本存储

## 概述

![avatar](https://upload-images.jianshu.io/upload_images/13765375-94d3ec387a0534f0.png)

文件系统+区块索引：左边的区块链，狭义上的区块存储，底层存储引擎是个文件系统，区块并不是存储在数据库里面，直接存储成为文件。使用文件系统查询区块就要用到右下角的文件索引。它是将区块的一些属性和文件存储的位置关联起来，比如说根据区块的哈希、区块的高度查询区块，也可以根据交易的ID查询区块。区块索引的实现使用了LevelDB，这是一个可以内嵌的开源数据库。没有区块索引是没有办法读取区块的。在fabric的实现里，并不是一个区块单独成一个文件，而是由N多个区块组合成一个文件块的形式进行存储。所以你就需要去表示每个区块在哪个文件块中，并且它的偏移量是多少，这样的一个映射过程就是区块索引所做的作用。

状态数据库：可以简单理解为区块链上的最新数据。状态数据库是随着交易的增加不断更新的，这个状态是可以根据区块重构的。如果我们把状态数据库丢失了，我们可以根据从创世区块以来所有的状态，按照交易的顺序一笔一笔的重新计算就可以重现整个状态数据库的当前内容。现阶段状态数据库支持LevelDB和CouchDB这两种数据库存储引擎。CouchDB比LevelDB更好的一点在于他可以存储结构化的数据，能够提供支持模糊查询的操作。

历史状态索引：一个数据的变动历史。在fabric当中他的设计很是巧妙，他不存储状态的具体变动结果，而只是告诉这个数据在哪一个交易中被更新了。这样做的好处显而易见的一点是节省了存储空间，我们只需要在查询的时候找到对应的交易ID，然后在根据区块索引模块查找相应的内容就可以了。

## 交易读写集

- 交易流程
  - 交易模拟->读写集（RWSet）
    - 读集：读取的已提交的状态键值（值）
    - 写集：将要更新的状态键值对
    - 写集：状态键值对删除标记
    - 写集：多次更新以最后一次为准
    - 版本号：二元组（区块高度、交易编号）
  - 交易排序
  - 交易验证->状态更新
    - 读集版本号==世界状态版本号（包括未提交的前序交易）
  
- 其他名词
  
  - 世界状态
    - 交易执行后的所有键的最新值
    - 显著提升链码执行的效率
    - 状态是所有交易日志的快照、可随时重构
    - LevelDB和CouchDB
  - 历史数据索引（可选）
    - 某键在某区块的某条交易中被改变
    - 只记录改变动作，不记录具体改变
    - 历史读取-->历史数据索引+区块读取
    - LevelDB组合键
  - 区块存储
    - 区块以文件块的形式存储（blockfile_xxxxxx）
    - 文件块大小：64M（硬编码）
    - 长辈最大容量：64M*1000000
  - 区块读取
    - 区块文件流（blockfileStream）
    - 区块流（blockStream）
    - 区块迭代器（blocksItr）
  - 区块索引
    - 快速定位区块
    - 索引建：区块高度/区块哈希/交易哈希...
    - 索引值：区块文件编号+文件内偏移量+区块数据长度
  - 区块提交
    - 保存区块文件
    - 更新世界状态
    - 更新历史状态（可选）

# 智能合约（链码）

- 智能合约

  - 区块链2.0：以太坊
  - 合约协议的数字化代码表达（合同用代码表达）
  - 分布式有限状态机（分布式系统中，起始状态一致，每个状态改变条件一样，那么得到的结果也是一样的）
  - 执行环境安全隔离、不受第三方干扰（EVM、Docker）

- 链码

  - Fabric应用层的基石（中间件）
  - 独立的Docker执行环境
  - 背书节点gRPC连接
  - 生命周期管理

- 生命周期

  - 打包（将编写的链码整合成一个文件，可以理解为智能合约的代码编译）
  - 安装（将刚才打包好的文件上传到背书节点）
  - 实例化（执行init方法，主要做一些初始化的操作。整个生命周期中，这个方法只会被执行一次）
  - 升级（前一版本有功能缺失，可以通过升级修复合约）
  - 交互（查询和写入）

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-16ae05bf6a95f9ce.jpg)

  首先应用程序发起一个交互请求到背书节点，背书节点收到这个消息后，首先会去调用容器管理模块看看链码容器是否正在运行，如果发现他没有启动，就回去编译和启动容器，启动后的容器会跟背书节点建立一个gRPC连接，连接建立好以后，背书节点会把交互请求转给合约进行执行，链码执行完成以后会返回一个执行结果，背书节点收到这个结果以后调用ESCC对这个结果进行签名背书，最终将背书结果返回给应用程序。第五步的签名，调用了ESCC，这是一种系统链码，主要完成一些系统功能，虽然是链码但是它是运行在节点进程中的，而不是一个链码容器的存在。

- 系统链码（了解即可）

  - LSCC（Lifecycle System Chaincode）
  - CSCC（Configuration System Chaincode）
  - QSCC（Query System Chaincode）
  - ESCC（Endorsement System Chaincode）
  - VSCC（Validation System Chaincode）

- 链码编程接口
    - Init()
    - Invoke() 
    
- 链码SDK接口
    - 参数解析
    - 交易信息
    - 状态操作
    - 链码互操作
    - 事件发送
    - 其他
    
- 链码编程禁忌
    - 分布式系统、多节点隔离执行
    - 随机函数
    - 系统时间
    - 不稳定的外部依赖

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

  从docker仓库下载所有需要的景象（官方景象和第三方景象）

## 步骤图解

- 启动六台节点

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-9271a876df3e2b07.jpg)

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-7440db5feef1fc72.png)

- 创建channel

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-1562662cdfac5970.jpg)

- 让每个peer加入channel

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-f9b200d0074d9dc8.jpg)

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-23a80a30d547ad25.png)

- 更新锚节点

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-b048ed7557668047.png)

- 链码代码

  ```js
  const shim = require('fabric-shim');
  const util = require('util');
  
  var Chaincode = class {
  
    // Initialize the chaincode
    async Init(stub) {
      console.info('========= example02 Init =========');
      let ret = stub.getFunctionAndParameters();
      console.info(ret);
      let args = ret.params;
      // initialise only if 4 parameters passed.
      if (args.length != 4) {
        return shim.error('Incorrect number of arguments. Expecting 4');
      }
  
      let A = args[0];
      let B = args[2];
      let Aval = args[1];
      let Bval = args[3];
  
      if (typeof parseInt(Aval) !== 'number' || typeof parseInt(Bval) !== 'number') {
        return shim.error('Expecting integer value for asset holding');
      }
  
      try {
        await stub.putState(A, Buffer.from(Aval));
        try {
          await stub.putState(B, Buffer.from(Bval));
          return shim.success();
        } catch (err) {
          return shim.error(err);
        }
      } catch (err) {
        return shim.error(err);
      }
    }
  
    async Invoke(stub) {
      let ret = stub.getFunctionAndParameters();
      console.info(ret);
      let method = this[ret.fcn];
      if (!method) {
        console.log('no method of name:' + ret.fcn + ' found');
        return shim.success();
      }
      try {
        let payload = await method(stub, ret.params);
        return shim.success(payload);
      } catch (err) {
        console.log(err);
        return shim.error(err);
      }
    }
  
    async invoke(stub, args) {
      if (args.length != 3) {
        throw new Error('Incorrect number of arguments. Expecting 3');
      }
  
      let A = args[0];
      let B = args[1];
      if (!A || !B) {
        throw new Error('asset holding must not be empty');
      }
  
      // Get the state from the ledger
      let Avalbytes = await stub.getState(A);
      if (!Avalbytes) {
        throw new Error('Failed to get state of asset holder A');
      }
      let Aval = parseInt(Avalbytes.toString());
  
      let Bvalbytes = await stub.getState(B);
      if (!Bvalbytes) {
        throw new Error('Failed to get state of asset holder B');
      }
  
      let Bval = parseInt(Bvalbytes.toString());
      // Perform the execution
      let amount = parseInt(args[2]);
      if (typeof amount !== 'number') {
        throw new Error('Expecting integer value for amount to be transaferred');
      }
  
      Aval = Aval - amount;
      Bval = Bval + amount;
      console.info(util.format('Aval = %d, Bval = %d\n', Aval, Bval));
  
      // Write the states back to the ledger
      await stub.putState(A, Buffer.from(Aval.toString()));
      await stub.putState(B, Buffer.from(Bval.toString()));
  
    }
  
    // Deletes an entity from state
    async delete(stub, args) {
      if (args.length != 1) {
        throw new Error('Incorrect number of arguments. Expecting 1');
      }
  
      let A = args[0];
  
      // Delete the key from the state in ledger
      await stub.deleteState(A);
    }
  
    // query callback representing the query of a chaincode
    async query(stub, args) {
      if (args.length != 1) {
        throw new Error('Incorrect number of arguments. Expecting name of the person to query')
      }
  
      let jsonResp = {};
      let A = args[0];
  
      // Get the state from the ledger
      let Avalbytes = await stub.getState(A);
      if (!Avalbytes) {
        jsonResp.error = 'Failed to get state for ' + A;
        throw new Error(JSON.stringify(jsonResp));
      }
  
      jsonResp.name = A;
      jsonResp.amount = Avalbytes.toString();
      console.info('Query Response:');
      console.info(jsonResp);
      return Avalbytes;
    }
  };
  
  shim.start(new Chaincode());
  ```

  

- 安装链码

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-d768d6358d0dffa8.jpg)

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-e595091bc44dc99f.png)

- 实例化链码

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-550ae95902ae5074.png)

- 执行链码

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-7e827d2aa64d3540.jpg)

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-471038caed4eab95.jpg)

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-c5f1ac62dba56408.jpg)

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

  OrdererOrgs:排序节点架构
  
  Domain：域名
  
  Specs：主机名
  
  PeerOrgs：节点架构
  
  Template：组织中拥有多少个peer
  
  Users：组织中拥有的用户数量
  
  一个组织中可以有多个peer同步记账，一个组织可以有一个或多个用户，一个组织中必须有一个admin管理员
  
  EnableNodeOUs：允许节点 OUS （out of service）
  
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

- 加密证书生成

  ```
  ../bin/cryptogen generate --config ./crypto-config.yaml
  ```
  

这个过程生成了crypto-config文件夹，里面包含了orderer机构信息和peer的机构信息，缩略图如下图所示：

![avatar](https://upload-images.jianshu.io/upload_images/13765375-3d57966d506816f4.png)

ca：证书

msp：类似于会员卡的意思

tls：下一代的互联网协议（lls）

详细内容如下：

```
.
├── ordererOrganizations
│   └── example.com
│       ├── ca
│       │   ├── 9a358c767605d638a9da1dd23e02c47bd85b93e443a48bc9b63c0e9aa08cde7b_sk
│       │   └── ca.example.com-cert.pem
│       ├── msp
│       │   ├── admincerts
│       │   ├── cacerts
│       │   │   └── ca.example.com-cert.pem
│       │   ├── config.yaml
│       │   └── tlscacerts
│       │       └── tlsca.example.com-cert.pem
│       ├── orderers
│       │   ├── orderer.example.com
│       │   │   ├── msp
│       │   │   │   ├── admincerts
│       │   │   │   ├── cacerts
│       │   │   │   │   └── ca.example.com-cert.pem
│       │   │   │   ├── config.yaml
│       │   │   │   ├── keystore
│       │   │   │   │   └── e395dbcc30c16d6429f7313f1a184f97c6f1eaf879ce0fe4fac906eedc33ef18_sk
│       │   │   │   ├── signcerts
│       │   │   │   │   └── orderer.example.com-cert.pem
│       │   │   │   └── tlscacerts
│       │   │   │       └── tlsca.example.com-cert.pem
│       │   │   └── tls
│       │   │       ├── ca.crt
│       │   │       ├── server.crt
│       │   │       └── server.key
│       │   ├── orderer2.example.com
│       │   │   ├── msp
│       │   │   │   ├── admincerts
│       │   │   │   ├── cacerts
│       │   │   │   │   └── ca.example.com-cert.pem
│       │   │   │   ├── config.yaml
│       │   │   │   ├── keystore
│       │   │   │   │   └── 78e7107e78a872f03658287faecd9cd08307147d08322ad82d224f0c15f40d76_sk
│       │   │   │   ├── signcerts
│       │   │   │   │   └── orderer2.example.com-cert.pem
│       │   │   │   └── tlscacerts
│       │   │   │       └── tlsca.example.com-cert.pem
│       │   │   └── tls
│       │   │       ├── ca.crt
│       │   │       ├── server.crt
│       │   │       └── server.key
│       │   ├── orderer3.example.com
│       │   │   ├── msp
│       │   │   │   ├── admincerts
│       │   │   │   ├── cacerts
│       │   │   │   │   └── ca.example.com-cert.pem
│       │   │   │   ├── config.yaml
│       │   │   │   ├── keystore
│       │   │   │   │   └── cbf99faab69480eb84ec8a22745efe043fcaaada3c5707b146bee127514e060a_sk
│       │   │   │   ├── signcerts
│       │   │   │   │   └── orderer3.example.com-cert.pem
│       │   │   │   └── tlscacerts
│       │   │   │       └── tlsca.example.com-cert.pem
│       │   │   └── tls
│       │   │       ├── ca.crt
│       │   │       ├── server.crt
│       │   │       └── server.key
│       │   ├── orderer4.example.com
│       │   │   ├── msp
│       │   │   │   ├── admincerts
│       │   │   │   ├── cacerts
│       │   │   │   │   └── ca.example.com-cert.pem
│       │   │   │   ├── config.yaml
│       │   │   │   ├── keystore
│       │   │   │   │   └── 04ddf63116297cdcf825d9cd764dd554646a77a1ab1abd7c1c7d4d50885c2152_sk
│       │   │   │   ├── signcerts
│       │   │   │   │   └── orderer4.example.com-cert.pem
│       │   │   │   └── tlscacerts
│       │   │   │       └── tlsca.example.com-cert.pem
│       │   │   └── tls
│       │   │       ├── ca.crt
│       │   │       ├── server.crt
│       │   │       └── server.key
│       │   └── orderer5.example.com
│       │       ├── msp
│       │       │   ├── admincerts
│       │       │   ├── cacerts
│       │       │   │   └── ca.example.com-cert.pem
│       │       │   ├── config.yaml
│       │       │   ├── keystore
│       │       │   │   └── d9f3aab206cf9de656215f5a6838cf24d9c7f372f474012c90e0fb6bdd0d2f46_sk
│       │       │   ├── signcerts
│       │       │   │   └── orderer5.example.com-cert.pem
│       │       │   └── tlscacerts
│       │       │       └── tlsca.example.com-cert.pem
│       │       └── tls
│       │           ├── ca.crt
│       │           ├── server.crt
│       │           └── server.key
│       ├── tlsca
│       │   ├── ad161eaa3207c26829d091f8982a21f9cd82e68e3c67ea1d06d2cb96f4bb7408_sk
│       │   └── tlsca.example.com-cert.pem
│       └── users
│           └── Admin@example.com
│               ├── msp
│               │   ├── admincerts
│               │   ├── cacerts
│               │   │   └── ca.example.com-cert.pem
│               │   ├── config.yaml
│               │   ├── keystore
│               │   │   └── 0a5a27103f92307ffaf5fff8454f67ad4320f16c0d139e6e63a110e2af7b3f0d_sk
│               │   ├── signcerts
│               │   │   └── Admin@example.com-cert.pem
│               │   └── tlscacerts
│               │       └── tlsca.example.com-cert.pem
│               └── tls
│                   ├── ca.crt
│                   ├── client.crt
│                   └── client.key
└── peerOrganizations
    ├── org1.example.com
    │   ├── ca
    │   │   ├── b4a8e99a2db4385b3c4b95afa99c9e4cd795e8eec08a1ac65b33d8613bb6072d_sk
    │   │   └── ca.org1.example.com-cert.pem
    │   ├── msp
    │   │   ├── admincerts
    │   │   ├── cacerts
    │   │   │   └── ca.org1.example.com-cert.pem
    │   │   ├── config.yaml
    │   │   └── tlscacerts
    │   │       └── tlsca.org1.example.com-cert.pem
    │   ├── peers
    │   │   ├── peer0.org1.example.com
    │   │   │   ├── msp
    │   │   │   │   ├── admincerts
    │   │   │   │   ├── cacerts
    │   │   │   │   │   └── ca.org1.example.com-cert.pem
    │   │   │   │   ├── config.yaml
    │   │   │   │   ├── keystore
    │   │   │   │   │   └── d447d4fca3ed19845e9c25bfeacb4010f05aa5a160b02834632a97ed32986c56_sk
    │   │   │   │   ├── signcerts
    │   │   │   │   │   └── peer0.org1.example.com-cert.pem
    │   │   │   │   └── tlscacerts
    │   │   │   │       └── tlsca.org1.example.com-cert.pem
    │   │   │   └── tls
    │   │   │       ├── ca.crt
    │   │   │       ├── server.crt
    │   │   │       └── server.key
    │   │   └── peer1.org1.example.com
    │   │       ├── msp
    │   │       │   ├── admincerts
    │   │       │   ├── cacerts
    │   │       │   │   └── ca.org1.example.com-cert.pem
    │   │       │   ├── config.yaml
    │   │       │   ├── keystore
    │   │       │   │   └── 4b4ee828029e344b6e8f03ad312c80694eeb70c2934fc8f09f93628a347bb106_sk
    │   │       │   ├── signcerts
    │   │       │   │   └── peer1.org1.example.com-cert.pem
    │   │       │   └── tlscacerts
    │   │       │       └── tlsca.org1.example.com-cert.pem
    │   │       └── tls
    │   │           ├── ca.crt
    │   │           ├── server.crt
    │   │           └── server.key
    │   ├── tlsca
    │   │   ├── aa33ce77e5525846ea9c162c069b83d26d1ebb4e5cf0af05bf80a1045e209ae0_sk
    │   │   └── tlsca.org1.example.com-cert.pem
    │   └── users
    │       ├── Admin@org1.example.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.org1.example.com-cert.pem
    │       │   │   ├── config.yaml
    │       │   │   ├── keystore
    │       │   │   │   └── 35a1fac79974418df860d5a31b0869e46428264a68453c278bd41d664e9c9ee4_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── Admin@org1.example.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.org1.example.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── server.crt
    │       │       └── server.key
    │       └── User1@org1.example.com
    │           ├── msp
    │           │   ├── admincerts
    │           │   ├── cacerts
    │           │   │   └── ca.org1.example.com-cert.pem
    │           │   ├── config.yaml
    │           │   ├── keystore
    │           │   │   └── fcf4077184bc0d901a1cc2d2127b684e3d5ea931f8d60dd5bb0f41101db9be15_sk
    │           │   ├── signcerts
    │           │   │   └── User1@org1.example.com-cert.pem
    │           │   └── tlscacerts
    │           │       └── tlsca.org1.example.com-cert.pem
    │           └── tls
    │               ├── ca.crt
    │               ├── client.crt
    │               └── client.key
    └── org2.example.com
        ├── ca
        │   ├── 1d916111e61f20e8c588f3879608f88c620086821113416a4e08f38e7742bd73_sk
        │   └── ca.org2.example.com-cert.pem
        ├── msp
        │   ├── admincerts
        │   ├── cacerts
        │   │   └── ca.org2.example.com-cert.pem
        │   ├── config.yaml
        │   └── tlscacerts
        │       └── tlsca.org2.example.com-cert.pem
        ├── peers
        │   ├── peer0.org2.example.com
        │   │   ├── msp
        │   │   │   ├── admincerts
        │   │   │   ├── cacerts
        │   │   │   │   └── ca.org2.example.com-cert.pem
        │   │   │   ├── config.yaml
        │   │   │   ├── keystore
        │   │   │   │   └── a964fa43639583a3f01f5aec0506140ed4c7547965445b3b46a0f7a9d4945981_sk
        │   │   │   ├── signcerts
        │   │   │   │   └── peer0.org2.example.com-cert.pem
        │   │   │   └── tlscacerts
        │   │   │       └── tlsca.org2.example.com-cert.pem
        │   │   └── tls
        │   │       ├── ca.crt
        │   │       ├── server.crt
        │   │       └── server.key
        │   └── peer1.org2.example.com
        │       ├── msp
        │       │   ├── admincerts
        │       │   ├── cacerts
        │       │   │   └── ca.org2.example.com-cert.pem
        │       │   ├── config.yaml
        │       │   ├── keystore
        │       │   │   └── 322ea700b2ca7002b13845a4555c9d9d457d7669903cdf5148aa03544c400eaf_sk
        │       │   ├── signcerts
        │       │   │   └── peer1.org2.example.com-cert.pem
        │       │   └── tlscacerts
        │       │       └── tlsca.org2.example.com-cert.pem
        │       └── tls
        │           ├── ca.crt
        │           ├── server.crt
        │           └── server.key
        ├── tlsca
        │   ├── df907bf13068bcc7063d3163bca2ca3677b1e736ede10bc52e8b10a5acf8eaca_sk
        │   └── tlsca.org2.example.com-cert.pem
        └── users
            ├── Admin@org2.example.com
            │   ├── msp
            │   │   ├── admincerts
            │   │   ├── cacerts
            │   │   │   └── ca.org2.example.com-cert.pem
            │   │   ├── config.yaml
            │   │   ├── keystore
            │   │   │   └── c9d2a4b539270657638a9a3f2c5265704c0505b73433fe5691677cef0d5bc356_sk
            │   │   ├── signcerts
            │   │   │   └── Admin@org2.example.com-cert.pem
            │   │   └── tlscacerts
            │   │       └── tlsca.org2.example.com-cert.pem
            │   └── tls
            │       ├── ca.crt
            │       ├── server.crt
            │       └── server.key
            └── User1@org2.example.com
                ├── msp
                │   ├── admincerts
                │   ├── cacerts
                │   │   └── ca.org2.example.com-cert.pem
                │   ├── config.yaml
                │   ├── keystore
                │   │   └── 1ac8047a9d066e07bf46edbab8a21871400e2fe10498b3e42aea82b8040cef36_sk
                │   ├── signcerts
                │   │   └── User1@org2.example.com-cert.pem
                │   └── tlscacerts
                │       └── tlsca.org2.example.com-cert.pem
                └── tls
                    ├── ca.crt
                    ├── client.crt
                    └── client.key

141 directories, 133 files

```



## 配置configtx.yaml文件

```
touch configtx.yaml
vim configtx.yaml
```
Organizations：整个区块链网络所有参与的组织信息



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
          - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=my-network_byfn
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

## 创建创世区块和创建通道

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

export CORE_PEER_ADDRESS=peer1.org1.example.com:8051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer channel join -b mychannel.block

export CORE_PEER_ADDRESS=peer0.org2.example.com:9051
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
peer channel join -b mychannel.block


export CORE_PEER_ADDRESS=peer1.org2.example.com:10051
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


export CORE_PEER_ADDRESS=peer1.org1.example.com:8051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node

export CORE_PEER_ADDRESS=peer0.org2.example.com:9051
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node


export CORE_PEER_ADDRESS=peer1.org2.example.com:10051
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

- 切换到basic-network目录
- 修改 basic-network的docker-compose.yml
- 修改 脚本 start.sh
- 启动脚本 'start.h'

## 链码架构

```js
const Chaincode = class {
    async Init(stub) {  // 初始化方法
         await stub.putState(key, Buffer.from(aStringValue)); //可以初始化一些世界状态
         return shim.success(Buffer.from('Initialized Successfully!'));
    }

	async Invoke(stub) {
   		let ret = stub.getFunctionAndParameters(); //获取函数名和参数
   		console.info(ret);
   		let method = this[ret.fcn]; //函数
   		let payload = await method(stub, ret.params); //调用函数
   		return shim.success(payload);
	}

	async xxx(stub, args) {//示例函数
    	return "xxx";
	}

};
shim.start(new Chaincode());

```

