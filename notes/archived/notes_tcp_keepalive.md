为什么要keepalive
检测通讯双方的“非活动状态"，为什么
	中间路由器崩溃，线路断线，只要两端主机不重启，则连接依然存在
并非TCP规范的一部分，有争议，争论点：不应该有TCP本是提供，而应该由应用程序来完成
KeepAlive功能主要为服务端应用程序提供，RLogin,Telnet服务其默认使用这个选项

Telnet客户端连接服务器
Telnet直接断点，导致在服务端留下一个半开连接，客户已经消失，（如果客户端再也不登录，则服务器永远不知道；
如果客户端重启后再使用相同的4元组发送消息给服务端，则服务端返回一个复位（这倒是我们希望的结果））
KeepAlive就是用来在服务端检测这种半开连接，及时释放这种半开放的连接
单位秒，默认2小时，如果服务端检测出客户机2小时内还没有发出任何一个报文，则向客户机发送一个“KeepAlive探查”
客户机处于以下4种状态：
1. 客户机正常，从服务器可达，回应服务器正常：服务器设置在最后一次交换数据后的未来2（tcp_keepalive_time）小时再发探查
2. 客户机已经崩溃并且关闭或正在重新启动，此时服务器收不到对探查的响应，并在75秒（tcp_keepalive_intvl）后超时服务器总共发送10（tcp_keepalive_probes）个探查，每个间隔75秒。如果服务器没有收到一个响应，它就认为客户主机已经关闭并终止连接
3. 客户主机崩溃后已经重启（注意测试客户机必须仍然使用上次连接的端口），此时客户机响应的是一个复位，服务器接收到复位响应后终止这个连接
4. 客户机正常，但是从服务器不可达（中间路由或线路问题，这与2系统）

以上的客户机崩溃和关闭均指的是不正常的关闭（比如断电），因为正常的客户端关闭或退出，TCP会在连接上发出FIN，收到FIN的服务器向应用程报告连接关闭，这是正常的关闭。

以上2,3,4应用层会收到TCP向上层发出的差错报告，以使得应用程序得知连接发生了哪些具体错误。
2：比如“连接超时”之类的错误
3：连接已被对方复位之类错误
4：同2

对于（4），扒掉网线就可以模拟
keepalive探查报文是一个ack，其值是:下一个要发送的报文ISN-1
tcpdump不鞥呢答应SYN,FIN,RST标志的空数据的序号

以下这段需要理解一下：
···
如果能够观察到第6和第1 0行的保活探查中的所有字段，我们就会发现序号字段比下一个
将要发送的序号字段小 1（在本例中，当下一个为 1 4时，它就是1 3） 。但是因为报文段中没有
数据，t c p d u m p不能打印出序号字段（它仅能够打印出设置了 S Y N、F I N或R S T标志的空数
据的序号） 。正是接收到这个不正确的序号，才导致服务器的 T C P对保活探查进行响应。这个
响应告诉客户，服务器下一个期望的序号是 1 4
···

