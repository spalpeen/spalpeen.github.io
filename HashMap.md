## HashMap 实现原理
* HashMap 是用于存储Key-Value键值对的集合，而每一个键值对也叫做Entry 这些Entry分散在一个数组中，这个数组就是HashMap的主干 默认初始长度是16 并且每次自动扩展收手动初始化长度时 长度必须是2的幂

* HashMapp数组中每一个元素的初始值都是null
* 对于HashMap最常用的两个方法就是get 和put


#### put
 当调用hashMap.put("hello","world") 插入一个Key为hello 的元素这个时候需要利用哈希函数来确定这个Entry的位置
 
 index = Hash("hello")
 
 假设index=2 那么这个 结果将放在主干数组2的位置 但是HashMap的长度是有限的 当插入的Entry越来越多时 HashMap也会有index冲突的时候 这个时候就用链表来解决

 HashMap数组的每一个元素不止是一个Entry对象，还是一个链表的头节点，每一个Entry对象通过next指针指向它的下一个Entry节点，当新来的Entry映射到冲突的数组位置只要插入到链表的头部就可以了（头插法 之所以采用头插法是认为后插入的Entry被查找的可能性更大）

#### get
当使用get根据Key来查找Value的时候 首先会把输入的Key做一次Hash映射
 
 index=Hash("hello")
 
 这时就需要根据对应Entry的头节点一直向下查找

#### HashMap的容量是有限的 当多次插入元素时 HashMap达到一定的饱和度，Key映射位置发生冲突的几率会逐渐提高 这个时候HashMap需要扩展它的长度 也就是进行Resize 而发生redize的因素有两个
 * 1.Capacity
    HashMap的当前长度
 * 2.LoadFactor
    HashMap负载因子 默认值为0.75f

 * 衡量HashMap是否进行resize的条件公式为：HashMap.size > = Capacity * LoadFactor

#### 扩容的步骤：
 1.扩容
 创建一个新的Entry数组，长度为原数组的2倍

 2.Rehash
 遍历原数组 把所有的Entry重新hash到新的数组，

 
* hashmap在高并发情况下容易出现链表环
 
 ## ConurrentHashMap

 * ConcurrentHashMap集合中有2的n次方个Segment 共同保存在一个名为Segments的数组当中
 
   Segment本身就相当于一个HashMap对象，同HashMap一样，Segment包含一个HashEntry数组，数组中每一个HashEntry即时一个键值对也是一个链表的头节点
 
 * 可以成ConcurrentHashMap是一个二级的哈希表 在一个总的哈希表下面有若干个子哈希表
 
 * ConcurrentHashMap的优势就是常采用锁分段技术 每一个Segment就好比一个自治区 读写操作高度自治

 #### put方法
 1.为输入的key做哈希运算，得到hash值
 
 2.通过hash值定位到对应的Segment
 
 3.获取可重入锁
 
 4再次通过hash值定位到Segment当中数组的具体位置
 
 5.插入或覆盖HashEntry对象
 
 6.释放锁
 
