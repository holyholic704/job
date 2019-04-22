Redis其他的功能
	慢查询
	pipeline
	发布订阅
	Bitmap
	HyperLogLog
	GEO

慢查询
	生命周期
		慢查询发生在第三阶段
		客户端超时不一定慢查询，慢查询是客户端超时的一个因素
	两个配置
		slowlog-max-len：先进先出的队列、固定长度、保存在内存中
		slowlog-log-slower-than：慢查询阈值（微秒）、=0记录所有命令、<0不记录任何命令
		配置方法
			默认值
			config get slowlog-max-len = 128
			config get slowlog-log-slower-than = 10000
			修改配置文件重启（不建议）
			动态配置
			config set slowlog-max-len 1000
			config set slowlog-log-slower-than 1000
	三个命令
		slowlog get[n]：获取慢查询队列
		slowlog len：获取慢查询队列长度
		slowlog reset：清空慢查询队列
	运维经验
		slowlog-max-len阈值不要设置过大，默认10ms，通常1ms
		slowlog-log-slower-than不要设置过小。通常设置1000左右
		理解命令生命周期
		定期持久化慢查询
		
pipeline
	注意每次pipeline携带数据量
	pipeline每次只能作用在一个redis节点上
	M操作与pipeline的区别

发布订阅
	角色
		发布者、频道、订阅者
	模型
	API
		publish：publish channel message
		subscrible：subscribe [channel] 订阅频道，一个或多个
		unsubscrible：subscribe [channel] 取消订阅频道，一个或多个
		不常用
		psubscribe [pattern]：订阅模式
		punsubscribe [pattern]：退订指定的模式
		pubsub channels：列出至少有一个订阅者的频道
		pubsub numsub [channel]：列出频道的订阅者数
		pubsub numpat：列出被订阅模式的数量
	发布订阅与消息队列的区别和使用场景

Bitmap
	位图
	本质是字符串
	setbit key offset value：给位图指定索引设置值
	getbit  key offset：
	bitcount key [start end]：获取位图指定范围位值为1的个数，单位为字节，不指定范围则为全部
	bitop op destkey key [key……]：做多个Bitmap的and、or、not、xor操作并将结果保存在destkey中
	bitpos key targetBat [start] [end]：计算位图指定范围第一个偏移量对应的值等于targetBit的位置
	注意setbit时的偏移量，可能有较大耗时
	
HyperLogLog
	新的数据结构？
		用极小的空间完成独立数量统计
		本质还是字符串
	三个命令
		pfadd key element [element……]：向hyperloglog添加元素
		pfcount key [key……]：计算hyperloglog的独立总数
		pfmerge destkey sourcekey [sourcekey……]：合并多个hyperloglog
	内存消耗
		小
	使用经验
		是否能容忍错误：错误率0.81%
		是否需要单条数据

GEO
	什么是GEO
		地理信息定位：存储经纬度、计算两地距离、范围计算等
	5个城市经纬度
	命令
		geoadd key longitude latitude member [longitude latitude member]
			增加地理位置信息
		geopos key member [member……]：获取地理位置信息
		geodist key m1 m2 [unit]：获取两个地理位置的距离，unit为单位，m、km、mi、ft
		georadius：看图
	说明
		3.2版本提供
		type geoKey = zset
		没有删除api