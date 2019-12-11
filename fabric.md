# 前言

时隔一年多，当联盟链重回大众视野后，我也在工作需求中重拾了联盟链，当然超级账本也就成了第一个要吃透的技术。废话不多说，以下开始我对超级账本的理解，不保证都对，遇到错误也麻烦大家指正，非常感谢，共同学习。

# 基础知识

## 概念名词

- hyperledger：由linux基金会孵化的区块链技术
- hyperledger fabric：第一个孵化出来的商用DLT（分布式账本技术）框架
- hyperledger fabric关键组件：peer（节点）， ordering service（排序服务），ca（证书），msp（会员管理者）
- chainCode：使用nodejs、go和java编写
- 授权网络
- 交易安全，交易可见性可控
- 无数字货币
- 可编程（chainCode）

## 数据完整性和安全性

hyperledger是一个分布式系统.没有单点故障, 没有单点的信息存放,每个节点都保存了全部的数据.所有的节点都保存了一致的区块链数据, 不可篡改.每个节点存放了所有的转账记录(账本) ,假设你的商业模式是使用hyperledger记录某个资产的所有者,因为某种原因, 你错误的登记了这个资产的所有人.在hyperledger里面你没法把这个错误的记录删除.你的做法只能是创建一个新的记录,标记之前的记录是错误的.在区块链系统里面,没有删除的概念, 所有对数据的添加和修改都会被记录.blockchain 记录了数据变化的过程.每一笔transaction都会导致数据的变化, 变化后的状态叫世界状态(world state)因为所有的node节点都保存了所有的transaction, 这些transaction都是一致的,所以所有的node节点的世界状态也是一致的.这个账本是由密码学签名保证.所以说hyperledger 区块链技术重建信任. 

## 授权网络和MSP和CA

每个operation在hyperledger网络 必须拥有数字签名.  update, 查询, insert, 或者获取metadata. 都需要数字签名.数字签名遵循x.509标准.  fabric ca是一个高质量的工具, 帮助我们自动生成证书. ca可以为不同的用户生成不同的证书, 每个用户可以拥有不同的attribute(属性), 在属性里面可以添加角色,账户id,account number等等.你可以添加任意的信息.hyperledger fabric 的chaincode可以获取用户的证书, 根据证书的类型决定某个智能合约是否可以执行.  比如说只有管理员可以安装链码, 普通用户只能查询. fabric ca就是生成ca和创建账户的工具. 可以根据自己的安全策略来管理用户.这个证书有效期是多久,分发策略是什么,都可以通过fabric ca来控制.fabirc ca支持链式继承,   假如某个证书被黑客攻陷, 上一级别的ca可以很容易作废这个证书.通过ca认证, 我们可以查询每一笔交易的参与者,并且参与者无法抵赖. 这个特性是密码学保证的hyperledger的组件是可插拔的, 你完全可以不使用fabric ca, 你可以自己建立一套认证体系,用于管理用户.设置用户属性,签名transaction.但是fabric ca是一个非常高质量,企业级的组件, 推荐使用.MSP只是一个接口，Fabric-CA是MSP接口的一种实现。

MSP是Membership Service Provider - 是可插拔的接口，它用于支持各种认证体系结构，为membership orchestration architecture提供抽象层。 MSP抽象提供：

具体的身份格式
用户证书验证
用户证书撤销
签名生成和验证
而 Fabric-CA 用于生成证书和密钥，以真正的初始化MSP。 Fabric-CA是用于身份管理的MSP接口的默认实现。
msp 定义
你是谁
你在什么网络
msp的证书是由fabric ca来颁发的
每个peer都需要msp的证书
每个order都需要msp的证书.
只有拥有相同msp的 peer才可以互相发现 互相通讯.MSP ID 是一个名字定义一组证书,说明你是谁你在哪个网络.使用hyperledger fabirc sdk的时候 经常需要指定mspid 

## Node和peer,client,orderer

- client 实例化transaction的(cli , node sdk, java sdk)
- peer 用来存储和同步ledger的数据
- orderer 用来排序分发transaction的

一个peer是ledger和blockchain存储的位置,是blockchain 的ledger存储的位置, 在生产环境中有多个peer, peer决定是否update的ledger.  一个peer会属于不同的channel.每个channel都在peer里面,但是是完全隔离的.一个peer可以控制多个channel,peer背书ledger的更新,最后强调一下peer是ledger和blockchain存储的位置,peer互相发现,互相同步

order提供排序服务. 在数据被提交到ledger之前, 必须先交给order服务, order服务创建block区块, 这些区块被签名和验证, 所有的transaction都在block里面,order做好了block之后,把数据发给peer, peer接收到block之后就把数据写入自己的ledger里面.

在hyperledger区块链中,挖矿的工作,共识的达成是有orderer节点来完成的,orderer负责避免双花,生成区块.

## Hyperledger的channel

每个channel可以理解成独立的hyperledger fabric的实例,channel是hyperledger fabirc里面一个非常重要的概念, 每个channel可以理解成独立的hyperledger fabric的实例, 所有的channel是完全独立的.一个channel不会依赖于其他的channel,不同的channel之间是不会交换数据的.不同的channel会有不同的规则,策略,智能合约,他们是完全独立的.channel 可以理解成private subnet, 有点类似微信的群.

## Chaincode

chaincode 就是智能合约  是应用程序,  是代码  是你的business logic, 他的作用是用来更新账本数据的.sdk发起一个transaction,  peer执行这个chaincode,chaincode可以用nodejs java和go编写, 在hyperledger里面,只要想读取和更新数据就必须通过chaincode的来完成

chaincode必须属于某个channel,因为ledger是属于某个channel的,chaincode操作的是ledger,当你执行一个操作的时候,你需要出示你的权限(ca), 明确在哪个channel, 执行哪个chaincode, 执行chaincode的什么函数, 函数的参数是什么.在一个channel里面可以定义一个chaincode,也可以定义多个chaincode, 看你们公司的业务逻辑.chaincode需要在某个channel的每个peer上安装,如果某个channel里面有3个peer , 你只在2个peer上安装chaincode, 最后会导致第三个peer的数据和其他peer的数据不一致.那第三个peer就会变成一个无效节点,无法搭乘共识,无法获取数据.

## Chaincode的生命周期

chaincode需要先安装, 然后必须要实例化,  实例化chaincode会启动docker容器. 在这个容器里面运行chaincode.

- 安装 install
- 实例化 init
- 调用 invoke

## Chaincode的背书策略

实例化chaincode需要指定背书策略. 背书策略是hpyerledger中一个很强大的功能.是所有的peer都要验证,还是大多数peer同意, 还是至少一个peer同意,  通过and或者or 这样的关键字定义背书策略.假设你的业务有10个不同的chaincode, 就可以有10个不同的背书策略. 有的代码要所有的节点同意才可以执行,有的逻辑需要至少有一个节点同意就可以执行.  这样就非常非常的灵活. 

## 工作流程

![avatar](https://upload-images.jianshu.io/upload_images/13765375-3d700b5f77b0b0d1.jpg)

​		这是最精简的一个案例, 一个组织, 3个peer节点, 在同一个网络, 同一个channel上.
sdk ,  nodejs java go , 命令行cli, 什么sdk都行. 
​		现在要执行一个更新账本的操作, 这个更新叫 transaction proposal , 一个修改账本的提案.peer 先接收到 proposal, 会模拟执行. 模拟执行.  拿着当前的versoion的ledger去模拟执行. 要注意的是每一个peer都有一个ledger(账本)的拷贝. 每个peer都有一份拷贝.这个数据在peer节点的硬盘里面存放着.模拟执行会产生一个读写集(read write set),具体读写集的内容为, 某个key要被update成什么, update后的version是什么. 这个模拟操作是由一个或者多个peer都会模拟执行的.每个peer会把模拟执行的结果 endorsement response 签名, 最后返回发送给sdk.sdk收集所有的背书响应.  invocation request,发送给orderer节点,order检查签名,背书策略,排序, 如果没有问题,就发送invocation.
​		背书策略非常重要,不同的链码可能对应不同的背书策略, 举个例子,假如背书策略是所有的节点都要同意.就需要检查是否有每个节点的背书, 如果背书策略是只要有一个同意, 那么只需要收集到一个背书即可.不合法的invocation request会被拒绝, 不会更新ledger的状态,(世界状态,读写集)但是transaction会被记录到blockchain.  方便以后排查检查问题. 专业属于叫审计. 方便以后审计.
​		ordering service还需要检查所有的背书对应的读写集是否一样, 只有一样的才会被接受.正常情况下所有的peer都保存了同一份账本的拷贝,执行相同的chaincode智能合约, 那么他们最终执行出来的结果也应该是一致的.如果数据不一致, 那么orderer节点就有理由认为,某个peer节点数据没有同步,或者peer节点被非法篡改. 不能更新ledger的数据,要把这次情况记录到blockchain. 如果数据一致, 那orderer服务就会发起invoke更新请求,  所有的节点都收到请求,更新自己的ledger状态.所有的peer数据还是保持一致的. 因为他们原始数据一致, 又都更新了相同的读写集.
​		另外orderer节点还担当了排序的角色, 因为在同一时刻,可能有来自于sdk的两个更新ledger的提案请求. 
先后顺序一定要排好.举个例子, 第一个proposal是张三花10块钱买大米, 在这完全的同一时刻,张三花10块钱买玉米的proposal也发了出来, 但是张三只有10块钱, orderer节点排序后, 第一个买大米的操作是合法的, 但是第二个买玉米的操作就不合法了.因为张三没有这么多钱了.  这也是hyperledger如何解决双花问题.在生产环境中, 一般会有多个组织, 多个组织之间的orderer节点 互相同步. 如果几个节点数据不同步, 就不会更新数据,  保证了order节点不会被攻破.黑客要同时攻击两个或者多个order是非常困难的. 这是一个非常非常强的安全模型. 
最后再回顾一下流程:

1. sdk发送transaction proposal给一个或者多个peer

2. peer模拟执行, 给出模拟执行的结果,读写集,key的version, 这些信息反馈给sdk

3. sdk收集背书信息,带着签名,发给orderer节点

4. orderer节点,检查数字签名,检查每个peer背书的读写集是否一致.排序.如果没有问题,就发出invocation 让每个peer去apply新的读写集.
  
   几个细节:
   orderer节点不是立刻处理每个invocation request.
   在orderer节点内部有一个消息队列 , 我们可以控制这个队列,控制队列的数量,容量大小,提交周期等参数
   通过调整参数调整hyperledger的处理效率.这些是hyperledger开发中比较高级的话题,
   关于性能调优, 后面我们会详细讨论.

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

![avatar](https://upload-images.jianshu.io/upload_images/13765375-94d3ec387a0534f0.png)

## 概述

## 交易读写集

# 智能合约（链码）

![avatar](https://upload-images.jianshu.io/upload_images/13765375-16ae05bf6a95f9ce.jpg)

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

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-9271a876df3e2b07.jpg)

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-7440db5feef1fc72.png)

- 创建channel

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-1562662cdfac5970.jpg)

- 让每个peer加入channel

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-f9b200d0074d9dc8.jpg)

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-23a80a30d547ad25.png)

- 更新锚节点

  ![avatar](https://upload-images.jianshu.io/upload_images/13765375-b048ed7557668047.png)

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

## 链码架构

