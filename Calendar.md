## Java中Calendar对日期的操作

### Calendar的调用

Calendar cal = Calendar.getInstance();

### 常用方法

//加减时间值 
* add(int field,int amount) 

//取出指定字段的值
* get(int field) 

//返回Calendar 可指定地区
* getInstance()     

//以毫秒返回时间
* getTimeInMills() 

//加减时间值，不进位
* roll(int field,boolean up)   

//设定制定字段的值
* set(int field,int value) 

//设定完整的时间
* set(year,month,day,hour,minute) 

//以毫秒指定时间
* setTimeInMillis(long millis)

### 关键字

//每月的几号
* DATE / DAY_OF_MONTH

//小时
* HOUR/HOUR_OF_DAY

//毫秒
* MILLSECOND

//分钟
* MINUTE

//月份
* MONTH

//年份
* YEAR

//时区位移
* ZONE_OFFSET
