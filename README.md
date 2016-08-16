Stiphle
======

它是什么？
-----------

Stiphle is a little library to try and provide an easy way of throttling/rate limit requests, for those without fancy hardware etc.

Stiphle是一个用来节流限速的简单轻便的小工具，你不再需要一些花哨的大家伙来完成这个功能。

它是如何工作的？
-----------------

You create a throttle, and ask it how long you should wait. For example, given
that $identifier is some means of identifying whatever it is you're throttling,
and you want to throttle it to 5 requests per second:

创建一个throttle，然后告诉他需要等待多长时间。举个栗子，给这个$identifier一些被identifying的意义，管他是什么，反正是你需要限流的，然后假设你想要一秒钟只想有5个请求


```php
<?php

$throttle = new Stiphle\Throttle\LeakyBucket;
$identifier = 'dave';
while(true) {
    // the throttle method returns the amount of milliseconds it slept for
    //throttle方法返回的是他休眠的毫秒数
    echo $throttle->throttle($identifier, 5, 1000);
}
// 0 0 0 0 0 200 200....


```

Use combinations of values to provide bursting etc, though use carefully as it
screws with your mind
使用值得组合来提供突发[^footstep1]等等的保证，但要小心使用，别把你的脑子搅得一团糟

[^footstep1]:一些技术(包括ATM和帧中继)被认为是可突发的。这意味着用户数据可以超过为该连接正常保留的带宽，但是不能超过端口速率。这种情况的一个例子是T-1上的一个128Kb/s帧中继CIR—取决于销售商，有可能短时间超过128Kb/s速率进行发送

```php
<?php

$throttle = new Stiphle\Throttle\LeakyBucket;
$identifier = 'dave';
for(;;) {
    /**
     * Allow upto 5 per second, but limit to 20 a minute - I think
     * 允许每秒钟5次，但一分钟只能20次 - 大概是这样吧
     **/
    echo "a:" . $throttle->throttle($identifier, 5, 1000);
    echo " b:" . $throttle->throttle($identifier, 20, 60000);
    echo "\n";
}
//a:0 b:0
//a:0 b:0
//a:0 b:0
//a:0 b:0
//a:0 b:0
//a:199 b:0
//a:200 b:0
//a:199 b:0
//a:200 b:0
//a:200 b:0
//a:199 b:0
//a:200 b:0
//a:199 b:0
//a:200 b:0
//a:200 b:0
//a:199 b:0
//a:200 b:0
//a:200 b:0
//a:199 b:0
//a:200 b:0
//a:199 b:0
//a:200 b:2600
//a:0 b:3000
//a:0 b:2999



```


节流策略
-------------------

There are currently two types of throttles, [Leaky
Bucket](http://en.wikipedia.org/wiki/Leaky_bucket) and a simple fixed time
window. 

目前有两种throttle，[Leaky
Bucket](http://en.wikipedia.org/wiki/Leaky_bucket)和一个简单固定的TimeWindow


```php
/**
 * Throttle to 1000 per *rolling* 24 hours, e.g. the counter will not reset at
 * midnight
 * 限制每24小时1000个请求，这个计数器不会在你睡的正香的时候重置 - 有12点就睡觉的程序员吗？
 *
 */
$throttle = new Stiphle\Throttle\LeakyBucket;
$throttle->throttle('api.request', 1000, 86400000);

/**
 * Throttle to 1000 per calendar day, counter will reset at midnight
 * 每天1000个，都说了简单的，晚上12点就重置了哟
 */
$throttle = new Stiphle\Throttle\TimeWindow;
$throttle->throttle('api.request', 1000, 86400000);

```

__NB:__ The current implementation of the `TimeWindow` throttle will only work on 64-bit architectures!

__NB:__ 现在TimeWindow只能在64位的机器上使用！

Todo
----

* More Tests!				
	
	更多的测试！
* Decent *Unit* tests	

	合适的*单元*！测试
* More throttling methods
	
	更多的限流方法
* More storage adapters, the current ones are a little volatile, Redis, Mongo,
  Cassandra, MemcacheDB etc
  
  更多的存储适配，现在的是一个小Volatile不是很稳定，希望用上Redis，Mongo,Cassandra,MemcacheDB什么的
  

Copyright
---------

Copyright (c) 2011 Dave Marshall. See LICENCE for further details
