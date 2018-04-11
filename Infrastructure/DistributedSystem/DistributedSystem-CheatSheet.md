[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# 分布式系统概述与理论基础

分布式架构是数据库发展的大势所趋。分布式架构显著提升大容量数据存储和管理能力，既保障面对大量用户的高并发需求，又保障了面对业务变化的弹性增长能力。分布式数据库的使用成本，也远低于传统数据库。由于分布式架构主要使用 PC 服务器与内置盘，因此几乎全部新型分布式数据库均使用多副本技术来保障数据的可靠性与安全性。

现代服务端架构，都可以称为分布式系统

微服务，分布式存储，分布式计算

Fallacies of Distributed Computing

网络是稳定的。网络传输的延迟是零。网络的带宽是无穷大。网络是安全的。网络的拓扑不会改变。只有一个系统管理员。传输数据的成本为零。整个网络是同构的。

# CAP

CAP 定理是分布式系统设计中最基础，也是最为关键的理论。它指出，分布式数据存储不可能同时满足以下三个条件。一致性（Consistency）：每次读取要么获得最近写入的数据，要么获得一个错误。可用性（Availability）：每次请求都能获得一个（非错误）响应，但不保证返回的是最新写入的数据。分区容忍（Partition tolerance）：尽管任意数量的消息被节点间的网络丢失（或延迟），系统仍继续运行。也就是说，CAP 定理表明，在存在网络分区的情况下，一致性和可用性必须二选一。而在没有发生网络故障时，即分布式系统正常运行时，一致性和可用性是可以同时被满足的。这里需要注意的是，CAP 定理中的一致性与 ACID 数据库事务中的一致性截然不同。

对于大多数互联网应用来说（如门户网站），因为机器数量庞大，部署节点分散，网络故障是常态，可用性是必须要保证的，所以只有舍弃一致性来保证服务的 AP。而对于银行等，需要确保一致性的场景，通常会权衡 CA 和 CP 模型，CA 模型网络故障时完全不可用，CP 模型具备部分可用性。

CA (consistency + availability)，这样的系统关注一致性和可用性，它需要非常严格的全体一致的协议，比如“两阶段提交”（2PC）。CA 系统不能容忍网络错误或节点错误，一旦出现这样的问题，整个系统就会拒绝写请求，因为它并不知道对面的那个结点是否挂掉了，还是只是网络问题。唯一安全的做法就是把自己变成只读的。
CP (consistency + partition tolerance)，这样的系统关注一致性和分区容忍性。它关注的是系统里大多数人的一致性协议，比如：Paxos 算法 (Quorum 类的算法)。这样的系统只需要保证大多数结点数据一致，而少数的结点会在没有同步到最新版本的数据时变成不可用的状态。这样能够提供一部分的可用性。
AP (availability + partition tolerance)，这样的系统关心可用性和分区容忍性。因此，这样的系统不能达成一致性，需要给出数据冲突，给出数据冲突就需要维护数据版本。Dynamo 就是这样的系统。

# 数据一致性

## 一致性分类

## 一致性协议

### Paxos

### Raft

## 多阶段提交

## MVCC

# 分布式应用（微服务）架构模式

# 分布式计算

## 批计算

## 流计算

# Introduction

# 分布式事务

```
分布式事务往往和本地事务进行对比分析，以支付宝转账到余额宝为例，假设有：

支付宝账户表：A（id，userId，amount）
```

余额宝账户表：B（id，userId，amount）

用户的 userId=1；

从支付宝转账 1 万块钱到余额宝的动作分为两步：

1）支付宝表扣除 1 万：update A set amount=amount-10000 where userId=1;

2）余额宝表增加 1 万：update B set amount=amount+10000 where userId=1;

```sql
Begin transaction
         update A set amount=amount-10000 where userId=1;
         update B set amount=amount+10000 where userId=1;
End transaction
commit;
```

```
在Spring中使用一个@Transactional事务也可以搞定上述的事务功能：
```

```java
@Transactional(rollbackFor=Exception.class)
    public void update() {
        updateATable(); //更新A表
        updateBTable(); //更新B表
    }
```

```
如果系统规模较小，数据表都在一个数据库实例上，上述本地事务方式可以很好地运行，但是如果系统规模较大，比如支付宝账户表和余额宝账户表显然不会在同一个数据库实例上，他们往往分布在不同的物理节点上，这时本地事务已经失去用武之地。
```

## 两阶段提交协议

```
两阶段提交协议（Two-phase Commit，2PC）经常被用来实现分布式事务。一般分为协调器C和若干事务执行者Si两种角色，这里的事务执行者就是具体的数据库，协调器可以和事务执行器在一台机器上。
```

![](http://images0.cnblogs.com/blog2015/522490/201508/091642197846523.png)

1） 我们的应用程序（client）发起一个开始请求到 TC；

2） TC 先将<prepare>消息写到本地日志，之后向所有的 Si 发起<prepare>消息。以支付宝转账到余额宝为例，TC 给 A 的 prepare 消息是通知支付宝数据库相应账目扣款 1 万，TC 给 B 的 prepare 消息是通知余额宝数据库相应账目增加 1w。为什么在执行任务前需要先写本地日志，主要是为了故障后恢复用，本地日志起到现实生活中凭证 的效果，如果没有本地日志（凭证），容易死无对证；

3） Si 收到<prepare>消息后，执行具体本机事务，但不会进行 commit，如果成功返回<yes>，不成功返回<no>。同理，返回前都应把要返回的消息写到日志里，当作凭证。

4） TC 收集所有执行器返回的消息，如果所有执行器都返回 yes，那么给所有执行器发生送 commit 消息，执行器收到 commit 后执行本地事务的 commit 操作；如果有任一个执行器返回 no，那么给所有执行器发送 abort 消息，执行器收到 abort 消息后执行事务 abort 操作。

注：TC 或 Si 把发送或接收到的消息先写到日志里，主要是为了故障后恢复用。如某一 Si 从故障中恢复后，先检查本机的日志，如果已收到<commit >，则提交，如果<abort >则回滚。如果是<yes>，则再向 TC 询问一下，确定下一步。如果什么都没有，则很可能在<prepare>阶段 Si 就崩溃了，因此需要回滚。

现如今实现基于两阶段提交的分布式事务也没那么困难了，如果使用 java，那么可以使用开源软件 atomikos([http://www.atomikos.com/](http://www.atomikos.com/))来快速实现。

不过但凡使用过的上述两阶段提交的同学都可以发现性能实在是太差，根本不适合高并发的系统。为什么？

1）两阶段提交涉及多次节点间的网络通信，通信时间太长！

2）事务时间相对于变长了，锁定的资源的时间也变长了，造成资源等待时间也增加好多！

## 消息队列方式

```
比如在北京很有名的姚记炒肝点了炒肝并付了钱后，他们并不会直接把你点的炒肝给你，往往是给你一张小票，然后让你拿着小票到出货区排队去取。为什么他们要将付钱和取货两个动作分开呢？原因很多，其中一个很重要的原因是为了使他们接待能力增强（并发量更高）。
```

还是回到我们的问题，只要这张小票在，你最终是能拿到炒肝的。同理转账服务也是如此，当支付宝账户扣除 1 万后，我们只要生成一个凭证（消息）即可，这个凭证（消息）上写着“让余额宝账户增加 1 万”，只要这个凭证（消息）能可靠保存，我们最终是可以拿着这个凭证（消息）让余额宝账户增加 1 万的，即我们能依靠这个凭证（消息）完成最终一致性。

### 业务与消息耦合的方式

```
支付宝在完成扣款的同时，同时记录消息数据，这个消息数据与业务数据保存在同一数据库实例里（消息记录表表名为message）；
```

```sql
Begin transaction
         update A set amount=amount-10000 where userId=1;
         insert into message(userId, amount,status) values(1, 10000, 1);
End transaction
commit;
```

```
上述事务能保证只要支付宝账户里被扣了钱，消息一定能保存下来。当上述事务提交成功后，我们通过实时消息服务将此消息通知余额宝，余额宝处理成功后发送回复成功消息，支付宝收到回复后删除该条消息数据。
```

### 业务与消息解耦方式

```
上述保存消息的方式使得消息数据和业务数据紧耦合在一起，从架构上看不够优雅，而且容易诱发其他问题。为了解耦，可以采用以下方式。
```

1）支付宝在扣款事务提交之前，向实时消息服务请求发送消息，实时消息服务只记录消息数据，而不真正发送，只有消息发送成功后才会提交事务；

2）当支付宝扣款事务被提交成功后，向实时消息服务确认发送。只有在得到确认发送指令后，实时消息服务才真正发送该消息；

3）当支付宝扣款事务提交失败回滚后，向实时消息服务取消发送。在得到取消发送指令后，该消息将不会被发送；

4）对于那些未确认的消息或者取消的消息，需要有一个消息状态确认系统定时去支付宝系统查询这个消息的状态并进行更新。为什么需要这一步骤，举个例子：假设在第 2 步支付宝扣款事务被成功提交后，系统挂了，此时消息状态并未被更新为“确认发送”，从而导致消息不能被发送。

优点：消息数据独立存储，降低业务系统与消息系统间的耦合；

缺点：一次消息发送需要两次请求；业务处理服务需要实现消息状态回查接口。

### 消息重复投递

```
还有一个很严重的问题就是消息重复投递，以我们支付宝转账到余额宝为例，如果相同的消息被重复投递两次，那么我们余额宝账户将会增加2万而不是1万了。		 	为什么相同的消息会被重复投递？比如余额宝处理完消息msg后，发送了处理成功的消息给支付宝，正常情况下支付宝应该要删除消息msg，但如果支付宝这时候悲剧的挂了，重启后一看消息msg还在，就会继续发送消息msg。

解决方法很简单，在余额宝这边增加消息应用状态表（message_apply），通俗来说就是个账本，用于记录消息的消费情况，每次来一个消息，在真正执行之前，先去消息应用状态表中查询一遍，如果找到说明是重复消息，丢弃即可，如果没找到才执行，同时插入到消息应用状态表（同一事务）。
```

\``` sql list

for each msg in queue

Begin transaction

```
select count(*) as cnt from message_apply where msg_id=msg.msg_id;
if cnt==0 then
  update B set amount=amount+10000 where userId=1;
  insert into message_apply(msg_id) values(msg.msg_id);
```

End transaction

commit;

\```

# 分布式锁

在很多互联网产品应用中，有些场景需要加锁处理，比如：秒杀，全局递增 ID，楼层生成等等。大部分是解决方案基于 DB 实现的，Redis 为单进程单线程模式，采用队列模式将并发访问变成串行访问，且多客户端对 Redis 的连接并不存在竞争关系。

> 分布式锁是控制分布式系统之间同步访问共享资源的一种方式。在分布式系统中，常常需要协调他们的动作。如果不同的系统或是同一个系统的不同主机之间共享了一个或一组资源，那么访问这些资源的时候，往往需要互斥来防止彼此干扰来保证一致性，在这种情况下，便需要使用到分布式锁。

简单的理解就是：分布式锁是一个在很多环境中非常有用的原语，它是不同的系统或是同一个系统的不同主机之间互斥操作共享资源的有效方法。

## Redis

1、为避免特殊原因导致锁无法释放, 在加锁成功后, 锁会被赋予一个生存时间(通过 lock 方法的参数设置或者使用默认值), 超出生存时间锁将被自动释放.

2、锁的生存时间默认比较短(秒级, 具体见 lock 方法), 因此若需要长时间加锁, 可以通过 expire 方法延长锁的生存时间为适当的时间. 比如在循环内调用 expire

3、系统级的锁当进程无论因为任何原因出现 crash，操作系统会自己回收锁，所以不会出现资源丢失。

4、但分布式锁不同。若一次性设置很长的时间，一旦由于各种原因进程 crash 或其他异常导致 unlock 未被调用，则该锁在剩下的时间就变成了垃圾锁，导致其他进程或进程重启后无法进入加锁区域。

```
<?php

require_once 'RedisFactory.php';

/**
* 在 Redis 上实现的分布式锁
*/
class RedisLock {

//单例模式
    private static $_instance = null;
    public static function instance() {
        if(self::$_instance == null) {
            self::$_instance = new RedisLock();
        }
        return self::$_instance;
    }


//redis对象变量
    private $redis;

//存放被锁的标志名的数组
    private $lockedNames = array();

    public function __construct() {

//获取一个 RedisString 实例
        $this->redis = RedisFactory::instance()->getString();
    }


/**

* 加锁

*

* @param string 锁的标识名

* @param int 获取锁失败时的等待超时时间(秒), 在此时间之内会一直尝试获取锁直到超时. 为 0 表示失败后直接返回不等待

* @param int 当前锁的最大生存时间(秒), 必须大于 0 . 如果超过生存时间后锁仍未被释放, 则系统会自动将其强制释放

* @param int 获取锁失败后挂起再试的时间间隔(微秒)

*/
    public function lock($name, $timeout = 0, $expire = 15, $waitIntervalUs = 100000) {
        if(empty($name)) return false;

        $timeout = (int)$timeout;
        $expire = max((int)$expire, 5);
        $now = microtime(true);
        $timeoutAt = $now + $timeout;
        $expireAt = $now + $expire;

        $redisKey = "Lock:$name";
        while(true) {
            $result = $this->redis->setnx($redisKey, (string)$expireAt);
            if($result !== false) {

//对$redisKey设置生存时间
                $this->redis->expire($redisKey, $expire);

//将最大生存时刻记录在一个数组里面
                $this->lockedNames[$name] = $expireAt;
                return true;
            }


//以秒为单位，返回$redisKey 的剩余生存时间
            $ttl = $this->redis->ttl($redisKey);

// TTL 小于 0 表示 key 上没有设置生存时间(key 不会不存在, 因为前面 setnx 会自动创建)

// 如果出现这种情况, 那就是进程在某个实例 setnx 成功后 crash 导致紧跟着的 expire 没有被调用. 这时可以直接设置 expire 并把锁纳为己用
            if($ttl < 0) {
                $this->redis->set($redisKey, (string)$expireAt, $expire);
                $this->lockedNames[$name] = $expireAt;
                return true;
            }


// 设置了不等待或者已超时
            if($timeout <= 0 || microtime(true) > $timeoutAt) break;


// 挂起一段时间再试
            usleep($waitIntervalUs);
        }

        return false;
    }


/**

* 给当前锁增加指定的生存时间(秒), 必须大于 0

*

* @param string 锁的标识名

* @param int 生存时间(秒), 必须大于 0

*/
    public function expire($name, $expire) {
        if($this->isLocking($name)) {
            if($this->redis->expire("Lock:$name", max($expire, 1))) {
                return true;
            }
        }
        return false;
    }


/**

* 判断当前是否拥有指定名称的锁

*

* @param mixed $name

*/
    public function isLocking($name) {
        if(isset($this->lockedNames[$name])) {
            return (string)$this->lockedNames[$name] == (string)$this->redis->get("Lock:$name");
        }
        return false;
    }


/**

* 释放锁

*

* @param string 锁的标识名

*/
    public function unlock($name) {
        if($this->isLocking($name)) {
            if($this->redis->deleteKey("Lock:$name")) {
                unset($this->lockedNames[$name]);
                return true;
            }
        }
        return false;
    }


/** 释放当前已经获取到的所有锁 */
    public function unlockAll() {
        $allSuccess = true;
        foreach($this->lockedNames as $name => $item) {
            if(false === $this->unlock($name)) {
                $allSuccess = false;
            }
        }
        return $allSuccess;
    }
}
```
