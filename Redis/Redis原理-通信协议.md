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
