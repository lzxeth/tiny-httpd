TinyHttpd - 对原版加了一些修改和bug修复
===================================

实现一个webServer要做的事：

1、建立一条TCP通道(startup,accept_request)

2、能够解析HTTP协议包，包括消息头解析和消息体解析(get_line)

3、能够响应符合HTTP规范的包(bad_request,headers,not_found,unimplemented)

4、处理静态请求(serve_file,cat)

5、处理动态请求(execute_cgi,cannot_execute)


修改
------------
1、添加函数readBufferBeforeSend，这个函数通过判断请求方法是GET或POST来进行相应的head和body的读取。原版中，当返回数据时，要判断请求方法是GET就先读出全部head并丢弃后返回数据，是POST就从http   head中读出CONTENT-LENGTH,然后按这个长度读取body后再返回数据，我给整合成一个方法，并且用这个方法修复了一个bug。

2、修改main函数中u_short port = 8008，对于测试来讲，并不希望web的端口通过系统分配。

3、在startup函数中加入
```
int reuse = 1;
if (setsockopt(httpd, SOL_SOCKET, SO_REUSEADDR, &reuse, sizeof(reuse)) < 0) {
    error_die("setsockopet");
}
```
在测试中，经常需要修改源码，改完后重新编译再启动会报错：
```
bind: Address already in use
```
但是httpd是已经关闭了的，往往过一会再启动就没事了。
因为TCP协议在四次挥手中主动关闭方会进入TIME_WAIT状态，并且会等待2MSL的时间，而服务端的重启和崩溃都会进入这个状态。这里通过重用TCP连接解决。



修复bug
-------------------------
1、原版中，在接收请求(accept_request)后，通过get_line解析请求行，发现不是GET或POST方法后，就调用unimplemented方法，通知客户端方法没有实现，但是在返回前没有读取tcp通道中的stream数据，这样客户端（可以用postman测试）就得不到响应，通过在调用unimplemented方法前添加readBufferBeforeSend方法，并在其后调用close关闭连接描述符，



License
-------



