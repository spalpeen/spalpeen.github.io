## 通信协议-RESP

* Redis的通信协议采用RESP(Redis序列化协议)
* 传输协议有5中最小单元类型,单元结束都会加上回车换行符号\r\n
* 下面将具体介绍这5中类型
1. 单行字符串

	* 单行字符串以 + 开头
	```
	+hello world\r\n
	```

2. 多行字符串

	* 多行字符串 以 $ 符号开头后跟字符串长度
	
	```
	$11\r\nhello world\r\n
	```

	* NULL用多行字符串表示不过长度要写成-1
	
	```
	$-1\r\n
	```
	* 空串 用多行字符串表示长度填 0
	* 注意这里有两个\r\n,两个\r\n之间隔的是空串
	
	```
	$0\r\n\r\n
	```

3. 整数

	* 整数值 以 : 符号开头后跟整数的字符串形式

	
	```
	$11\r\nhello world\r\n
	```

4. 错误消息

	* 错误消息 以 - 符号开头
	
	```
	-WRONGTYPE Operation against a key holding the wrong kind of value\r\n
	```
5. 数组

	* 数组 以 * 号开头后跟数组的长度
	
	```
	*3\r\n:1\r\n:2\r\n:3\r\n
	```

##### 客户端->服务器

* 客户端向服务器发送的指令只有一种格式---多行字符串数组
* 举个例子:客户端发送指令 set name kongming 实际上会序列化成下面的字符串
	
```
*3\r\n$3\r\nset\r\n$4\r\nname\r\n$8\r\nkongming\r\n
```

##### 服务器->客户端

* 服务端回复客户端可以返回多种数据格式
	
* 单行字符串
	
```
127.0.0.1:6379> set name kongming
OK //这里的OK实际上是+OK\r\n
```

* 错误
	
```
127.0.0.1:6379> incr name
(error) ERR value is not an integer or out of range //实际为-ERR value...\r\n
```
	
* 整数
	
```
127.0.0.1:6379> incr age
(integer) 1   //这里的(integer) 1 实际上是 :1\r\n
```
	
	* 多行字符串

	```
	127.0.0.1:6379> get name
	"kongming" //这里的"kongming" 实际上是 $8\r\nkongming\r\n
	```

	* 数组

	```
	127.0.0.1:6379> hset kongming name kongming
	(integer) 1
	127.0.0.1:6379> hset kongming age 18
	(integer) 1
	127.0.0.1:6379> hset kongming sex man
	(integer) 1
	127.0.0.1:6379> hgetall kongming
	1) "name"		//key
	2) "kongming"	//value
	3) "age"
	4) "18"
	5) "sex"
	6) "man"
	//实际返回数据为 *6\r\n$4\r\nname\r\n$8\r\nkongming\r\n$3\r\nage\r\n$2\r\n18\r\n$3\r\nsex\r\n$3\r\nman\r\n
	```
