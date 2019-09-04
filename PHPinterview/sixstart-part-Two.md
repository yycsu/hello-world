# Part Two

收集自：公众号——PHP 开源社区

## 2019-8-28

### 一、Redis如何做内存优化？

尽可能使用散列表（hashes），散列表（是说散列表里面存储的数少）使用的内存非常小，所以你应该尽可能的将你的数据模型抽象到一个散列表里面。

比如你的web系统中有一个用户对象，不要为这个用户的名称，姓氏，邮箱，密码设置单独的key，而是应该把这个用户的所有信息存储到一张散列表里面。

### 二、Redis回收进程如何工作的？

一个客户端运行了新的命令，添加了新的数据。

Redi检查内存使用情况，如果大于maxmemory的限制， 则根据设定好的策略进行回收。

一个新的命令被执行，等等。

所以我们不断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下。

如果一个命令的结果导致大量内存被使用（例如很大的集合的交集保存到一个新的键），不用多久内存限制就会被这个内存使用量超越。

### 2019-8-29

以下代码输出的结果是：（`array_merge`的区别）

```php
$a = [0,1,2,3];
$b = [1,2,3,4,5];
$a+=$b;
echo json_encode($a)
```

A. `[0,1,2,3]`   B. `[1,3,5,7,5]`  C. `[1,2,3,4,5]`  ***D. `[0,1,2,3,5]`***

参考答案：D
答案解析：考的是数组+和array_merge的区别 当下标为数值时，array_merge()不会覆盖掉原来的值，但array＋array合并数组则会把最先出现的值作为最终结果返回，而把后面的数组拥有相同键名的那些值“抛弃”掉（不是覆盖）. 当下标为字符时，array＋array仍然把最先出现的值作为最终结果返回，而把后面的数组拥有相同键名的那些值“抛弃”掉，但array_merge()此时会覆盖掉前面相同键名的值.

### 2019-8-30

#### 关于缓存雪崩事前事中事后的解决方案正确的有？

***A、事前：进行系统压力测试，在负载均衡层做限流处理，过载丢弃请求或者进入队列***

***B、事前：`redis` 高可用，主从+哨兵，`redis cluster`，避免全盘崩溃***

***C、事中：本地`ehcache` 缓存+`hystrik` 限流&降级，避免`MySQL`被打死***

***D、事后：`redis` 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。***

P.S. 缓存雪崩参考https://www.cnblogs.com/hadley/p/9979785.html

缓存雪崩的事前事中事后的解决方案如下：

- 事前：进行系统压力测试，在负载均衡层做限流处理，过载丢弃请求或者进入队列 

- 事前：redis 高可用，主从+哨兵，redis cluster，避免全盘崩溃。 

- 事中：本地 ehcache 缓存 + hystrix 限流&降级，避免 MySQL 被打死。 

- 事后：redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。

### 2019-8-31

#### Redis 内存淘汰机制有哪些？

***A、noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错***
***B、allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key***
***C、volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key***
D、allkeys-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key。

P.S. redis 内存淘汰参考 [https://www.cnblogs.com/alsf/p/9399011.html](https://www.cnblogs.com/alsf/p/9399011.html)

答案解析：
redis 内存淘汰机制有以下几个：

- noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错，这个一般没人用。 allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）。 

- allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个 key，这个一般也没人用，为啥要随机，是把最近最少使用的 key 给干掉。 

- volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key（这个一般不太合适）。 

- volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key。 

- volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除。

### 2019-9-3

以下代码输出的结果是？

```php
if('xuexi' == 0){
    echo 1;
}else{
    echo 2;
}
```

***A. 1***  B. 2 C. 3 D. 4

答案解析：

`if($str==0)`，字符串和数字比较是否相等， 相当于 把`$str` 字符串隐性转换为数字，然后再比较，相当于 `if( intval($str) == 0 ) `。if(\$str==0) 判断 和 `if( intval($str) == 0 )` 是等价的，而和 `if ($str)` 是不一样的。`if ($str)` 可以判断 `$str`值有没有被初始化。有没有付值，只要付值，就返回`true`。 当然你也可以使用 `$str="字符串"`;`if($str===0){ echo "返回了true.";}` ，就是 判断 `$str`的数据类型 和值 都和0的值 数据类型一样，才可以返回`true`