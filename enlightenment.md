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

		这是最精简的一个案例, 一个组织, 3个peer节点, 在同一个网络, 同一个channel上.
sdk ,  nodejs java go , 命令行cli, 什么sdk都行. 
		现在要执行一个更新账本的操作, 这个更新叫 transaction proposal , 一个修改账本的提案.peer 先接收到 proposal, 会模拟执行. 模拟执行.  拿着当前的versoion的ledger去模拟执行. 要注意的是每一个peer都有一个ledger(账本)的拷贝. 每个peer都有一份拷贝.这个数据在peer节点的硬盘里面存放着.模拟执行会产生一个读写集(read write set),具体读写集的内容为, 某个key要被update成什么, update后的version是什么. 这个模拟操作是由一个或者多个peer都会模拟执行的.每个peer会把模拟执行的结果 endorsement response 签名, 最后返回发送给sdk.sdk收集所有的背书响应.  invocation request,发送给orderer节点,order检查签名,背书策略,排序, 如果没有问题,就发送invocation.
		背书策略非常重要,不同的链码可能对应不同的背书策略, 举个例子,假如背书策略是所有的节点都要同意.就需要检查是否有每个节点的背书, 如果背书策略是只要有一个同意, 那么只需要收集到一个背书即可.不合法的invocation request会被拒绝, 不会更新ledger的状态,(世界状态,读写集)但是transaction会被记录到blockchain.  方便以后排查检查问题. 专业属于叫审计. 方便以后审计.
		ordering service还需要检查所有的背书对应的读写集是否一样, 只有一样的才会被接受.正常情况下所有的peer都保存了同一份账本的拷贝,执行相同的chaincode智能合约, 那么他们最终执行出来的结果也应该是一致的.如果数据不一致, 那么orderer节点就有理由认为,某个peer节点数据没有同步,或者peer节点被非法篡改. 不能更新ledger的数据,要把这次情况记录到blockchain. 如果数据一致, 那orderer服务就会发起invoke更新请求,  所有的节点都收到请求,更新自己的ledger状态.所有的peer数据还是保持一致的. 因为他们原始数据一致, 又都更新了相同的读写集.
		另外orderer节点还担当了排序的角色, 因为在同一时刻,可能有来自于sdk的两个更新ledger的提案请求. 
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