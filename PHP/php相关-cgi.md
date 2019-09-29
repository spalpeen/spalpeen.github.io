## cgi，fast-cgi，php-cgi，php-fpm区别

#### cgi

* CGI的英文是（COMMON GATEWAY INTERFACE）公共网关接口

* 作用：帮助服务器与语言进行通信 普遍用于nginx与php通信，nginx 与 php 语言不通所以需要一个协议来进行双方沟通，而cgi就是这个沟通的协议

#### fast-cgi

* cgi协议在每次连接请求时会开启一个进程进行处理，处理完毕后关闭，当多个连接产生时就会产生多个进程消耗资源和内存

* fast-cgi解决了这个问题，当每次处理完请求后并不会直接关闭进程，使这个进程可以处理多个连接

#### php-cgi

* php-cgi是php提供给web server的cgi接口协议程序

* 每次接到web server请求都会开启一个php-cgi进程进行处理

###$ php-fpm

* php-fpm是php提供给web server的fast-cgi接口协议程序

* 它不会想php-cgi一样每次连接都开启一个新进程，处理完关闭进程

* 它允许一个进程处理多个连接

* php-fpm会开启多个php-cgi程序，每次接到web server请求，php-fpm会将连接信息分配给下面其中的一个子程序也就是php-cgi处理，处理完也不会马上关闭，而是等待下次分配

* php-fpm 常驻内存

* php-fpm 可以让修改的php配置平滑过渡
