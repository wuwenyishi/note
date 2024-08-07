
在Redis内部，每当我们设置一个键的过期时间时，Redis就会将该键带上过期时间存放到一个**过期字典**中。当我们查询一个键时，Redis便首先检查该键是否存在过期字典中，如果存在，那就获取其过期时间。然后将过期时间和当前系统时间进行比对，比系统时间大，那就没有过期；反之判定该键过期。  

# 过期删除策略  

## 定时删除    

在设置key的过期时间的同时，创建一个定时器，让定时器在key的过期时间来临时，立即执行对key的删除操作；  

优点：定时删除对内存是最友好的，能够保存内存的key一旦过期就能立即从内存中删除。  

缺点：对CPU最不友好，在过期键比较多的时候，删除过期键会占用一部分 CPU 时间，对服务器的响应时间和吞吐量造成影响。  



## 惰性删除    

设置该key 过期时间后，我们不去管它，当需要该key时，我们在检查其是否过期，如果过期，我们就删掉它，反之返回该key。  

优点：对 CPU友好，我们只会在使用该键时才会进行过期检查，对于很多用不到的key不用浪费时间进行过期检查。

缺点：对内存不友好，如果一个键已经过期，但是一直没有使用，那么该键就会一直存在内存中，如果数据库中有很多这种使用不到的过期键，这些键便永远不会被删除，内存永远不会释放。从而造成内存泄漏。  



## 定期删除    

每隔一段时间，我们就对一些key进行检查，删除里面过期的key。  

优点：可以通过限制删除操作执行的时长和频率来减少删除操作对 CPU 的影响。另外定期删除，也能有效释放过期键占用的内存。  

缺点：

* 难以确定删除操作执行的时长和频率。
* 如果执行的太频繁，定期删除策略变得和定时删除策略一样，对CPU不友好。
* 如果执行的太少，那又和惰性删除一样了，过期键占用的内存不会及时得到释放。
* 另外最重要的是，在获取某个键时，如果某个键的过期时间已经到了，但是还没执行定期删除，那么就会返回这个键的值，这是业务不能忍受的错误。



# Redis过期删除策略  

单一使用某一策略都不能满足实际需求，可组合使用。

Redis的过期删除策略就是：惰性删除和定期删除两种策略配合使用。

* **惰性删除**  

Redis的惰性删除策略由 db.c/expireIfNeeded 函数实现，所有键读写命令执行之前都会调用 expireIfNeeded 函数对其进行检查，如果过期，则删除该键，然后执行键不存在的操作；未过期则不作操作，继续执行原有的命令。

* **定期删除**  

由redis.c/activeExpireCycle 函数实现，函数以一定的频率运行，每次运行时，都从一定数量的数据库中取出一定数量的随机键进行检查，并删除其中的过期键。

> 注意：并不是一次运行就检查所有的库，所有的键，而是随机检查一定数量的键。  

定期删除函数的运行频率，在Redis2.6版本中，规定每秒运行10次，大概100ms运行一次。在Redis2.8版本后，可以通过修改配置文件redis.conf 的 **hz** 选项来调整这个次数。  

**建议不要将这个值设置超过 100，否则会对CPU造成比较大的压力。** 

通过过期删除策略，对于某些永远使用不到的键，并且多次定期删除也没选定到并删除，那么这些键同样会一直驻留在内存中，又或者在Redis中存入了大量的键，这些操作可能会导致Redis内存不够用，这时候就需要**Redis的内存淘汰策略**了。





