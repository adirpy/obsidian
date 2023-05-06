## 问题描述

现场进行了三方（INFI/IM/EDN）的地址数据比较，发现三方数据都差距较大，INFI和IM的地址差距在几百万，IM和EDN的地址也差距了一百多万。

地址的不同，严重影响了业务开通的准确性。

## 问题分析
### 直接原因分析

检查检查im_infi_addr_sync_log表，发现表内积压了1399305条数据。
IM失败的数据数量为309183条。
EDN失败的数量为895条，未返回的为1397298条。

IM的失败原因主要集中在：
IM-RIVT-INFI-ADDR-0000001: 200
 Table 'oss_im.im_syn_covered_addr': 1926
 IM-OOM-OBJATTR-IS-REQUIRED: 101352
 Duplicate entry: 620
 IM-RIVT-ADDRESS-00014: 120
 Load balancer does not have available server for client: OSS-EDN-API: 175089
 IM-RIVT-INFI-ADDR-0000005:41
 IM-RIVT-ADDRESS-00007:2683
 IM-RIVT-INFI-ADDR-0000002: 2969
 Out of range value for column 'LATITUDE: 30
 java.lang.NullPointerException
	at com.ztesoft.zsmart.oss.ho.im.rivt.address.dao.mysql.AddressDAOMysqlImpl.swapAddress(AddressDAOMysqlImpl.java:90): 73
Data too long for column: 260
status 500 reading HoSyncEDNClient#queryHyperzoneByLngLat(String,String): 6
IM_STACK_TRACE = null: 12865

其中重点： 
* IM-OOM-OBJATTR-IS-REQUIRED: The Address 's attribute City is required:
	这里的原因是：
	
* Load balancer does not have available server for client
	这里的原因是：

而EDN最核心的原因是：
代码没有进行返回。导致无法分析错误。

### 其余问题

1. Google Pub/Sub无法保证顺序性，同时Infi也没有给出具体的流水号，这个是最根本的原因
2. 在途单如果存在，没有重试
3. 重复主键直接抛错
4. EDN长期没有返回
5. INFI同步性能问题，目前大约是200条一分钟，客户要求500条一分钟。


## 数据修复方案

1. 第一版数据对比，分析数据问题(AP:徐新凯).  IM-INFI , IM-EDN
2. 版本发布之前的定期数据比对(AP：张磊)

## 程序方案更新

### INFI地址同步方案更新

#### 内容
1、在Redis增加处理队列和等待队列，前面优先以单线程接收客户地址请求同步信息，并始终优先处理更改地址ID请求，再处理新增/编辑请求，最后处理删除请求，这样始终可以保持变更地址ID优先执行。

2、在单线程处理时，始终同一个地址ID的请求只有一个正在处理，其他的如果发现Redis有正在处理的则放入等待队列中，等待当前处理完成后再解挂执行。

3、地址同步始终以IM同步为主，IM和Infinity同步完成后，当前事物结束，再生产新的内部MQ消息同步到修改在途单、服务存量、EDN数据，把内部同步与外部同步事物拆分两个异步操作，减少相互之间影响，始终保障IM和Infinity实时同步且一致。

4、如果内部各个模块系统同步失败，则根据失败原因配置异常重新执行次数和间隔时间自动处理重新执行策略，降低人工处理过程。

5、日志记录以Infinity同步请求消息流水作为主ID，并分别记录到与IM、在途单、服务存量、EDN的同步日志中，并记录关联关系，方便定位问题和查询。

6、地址ID外部和内部同步都完成后，才允许讲相同地址ID的后一个同步消息由等待队列解挂到当前执行队列，以保障地址同步次序的正确性。

7、提供监控页面可以监控Infinity、IM、在途单、服务存量、EDN的同步日志，并提供人工Redo或者忽略成功的操作，方便客户和交付运维。

8、设定每天或者每周定期发送地址同步统计结果给客户负责人和现场交付负责人，可以及时处理同步异常数据，保障数据同步完整性。


常见错误：
IM-RIVT-INFI-ADDR-0000002： SWAP UPRN不存在，IM处理方式改为放入等待队列，待存在后再进行处理。EDN的处理方式改为




#### 解决问题
1. 减少swap时地址ID不存在时的错误
2. 在途单直接处理，减少因为存在在途单的错误
3. 提高性能








方案描述：  
解决同步性能问题 
进行地址数据的全量比对  
1. 地址实例数据UPRN ID以Infinity为准  
2. Rollout Status以IM的为准 
3. Zone ID以EDN为准


AP：彭漾
1. Redis 队列内排序
2. Redo
3. updateHyperzone移出(另起进程去进行异步处理)
4. 不再判断在途单，直接修改

执行层面：
1. 监控界面（彭漾）
2. 定期检查，客户沟通（张磊）


数据问题：
1. 第一版数据对比，分析数据问题(AP:徐新凯).  IM-INFI , IM-EDN
2. 版本发布之前的定期数据比对(AP：张磊)