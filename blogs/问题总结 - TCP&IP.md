
## TCP粘包
TCP是流式传送的也就是连接建立后可以一直不停的发送,并没有明确的边界定义。而用UDP发送的时候是可以按照一个一个数据包去发送的一个数据包就是一个明确的边界。因为TCP是流式传送，所以会开辟一个缓冲区，发送端往其中写入数据，每过一段时间就发送出去，因此有可能后续发送的数据（属于另一个包）和之前发送的数据同时存在缓冲区中并一起发送，早晨粘包。接收端也有缓存，因此也存在粘包。
处理粘包的唯一方法就是制定应用层的数据通讯协议，通过协议来规范现有接收的数据是否满足消息数据的需要。在应用中处理粘包的基础方法主要有两种分别是以4节字描述消息大小或以结束符，实际上也有两者相结合的如HTTP,redis的通讯协议等。

## TCP的拥塞控制
`慢开始`：在主机刚刚开始发送报文段时可先将拥塞窗口 cwnd 设置为一个最大报文段 MSS 的数值。在每收到一个对新的报文段的确认后，将拥塞窗口增加至多一个 MSS 的数值。用这样的方法逐步增大发送端的拥塞窗口 cwnd，可以使分组注入到网络的速率更加合理。 

`拥塞避免`：当拥塞窗口值大于慢开始门限时，停止使用慢开始算法而改用拥塞避免算法。拥塞避免算法使发送端的拥塞窗口每经过一个往返时延RTT就增加一个MSS的大小。

`快重传`算法规定，发送端只要一连收到三个重复的 ACK 即可断定有分组丢失了，就应立即重传丢失的报文段而不必继续等待为该报文段设置的重传计时器的超时。

`快恢复`算法：
(1) 当发送端收到连续三个重复的 ACK 时，就重新设置慢开始门限 ssthresh。
(2) 与慢开始不同之处是拥塞窗口 cwnd 不是设置为 1，而是设置为 ssthresh + 3 *MSS。 
(3) 若收到的重复的 ACK 为 n 个（n > 3），则将 cwnd 设置为 ssthresh + n * MSS。
(4) 若发送窗口值还容许发送报文段，就按拥塞避免算法继续发送报文段。
(5) 若收到了确认新的报文段的 ACK，就将 cwnd 缩小到 ssthresh。

`“乘法减小“`是指不论在慢开始阶段还是拥塞避免阶段，只要出现一次超时（即出现一次网络拥塞），就把慢开始门限值 ssthresh 设置为当前的拥塞窗口值乘以 0.5。当网络频繁出现拥塞时，ssthresh 值就下降得很快，以大大减少注入到网络中的分组数。

`“加法增大”`是指执行拥塞避免算法后，当收到对所有报文段的确认就将拥塞窗口 cwnd增加一个 MSS 大小，使拥塞窗口缓慢增大，以防止网络过早出现拥塞。 

## HTTP缓存控制
HTTP所控制的缓存主要基于浏览器缓存，以及缓存代理服务器来实现。主要涉及以下`6个HTTP Header`：
`Expires`、`Cache-Control Header`、`Last-Modified`、`If-Modified-Since`、`ETag`、`If-None-Match`。

Expires/Cache-Control Header是控制浏览器`是否直接从浏览器缓存取数据还是重新发请求到服务器取数据`。只是Cache-Control比Expires可以控制的多一些，而且`Cache-Control会重写Expires的规则`。“Cache-control”常见的取值有private、no-cache、max-age、must-revalidate等。如果指定cache-control的值为private、no-cache、must-revalidate，那么打开新窗口访问时都会重新访问服务器。而如果指定了max-age值，那么在此值内的时间里就不会重新访问服务器，例如：Cache-control: max-age=5表示当访问此网页后的5秒内再次访问不会去服务器

Last-Modified/If-Modified-Since和ETag/If-None-Match是浏览器`发送请求到服务器后`判断文件是否已经修改过，如果没有修改过就只发送一个304回给浏览器，告诉浏览器直接从自己本地的缓存取数据；如果修改过那就整个数据重新发给浏览器。

Expires和Cache-Control max-age的区别与联系：
（1）Expires在HTTP/1.0中已经定义，Cache-Control:max-age在HTTP/1.1中才有定义。
（2）Expires指定一个绝对的过期时间(GMT格式)，这么做会导致至少2个问题：1）客户端和服务器时间不同步导致Expires的配置出现问题。2）很容易在配置后忘记具体的过期时间，导致过期来临出现浪涌现象；max-age 指定的是从文档被访问后的存活时间，这个时间是个相对值，相对的是文档第一次被请求时服务器记录的Request_time(请求时间)
（3）Expires指定的时间可以是相对文件的最后访问时间(Atime)或者修改时间(MTime)，而max-age相对对的是文档的请求时间(Atime)
（4）在Apache中，max-age是根据Expires的时间来计算出来的max-age = expires- request_time:(mod_expires.c)

Last-Modified/If-Modified-Since和ETag/If-None-Match工作方式
第一种，浏览器把缓存文件的最后修改时间通过If-Modified-Since来告诉Web服务器（初次请求时不带这个头）。（其实浏览器缓存里存储的不只是网页文件，还有服务器发过来的该文件的最后服务器修改时间）。服务器会把这个时间与服务器上实际文件的最后修改时间进行比较。如果时间一致，那么返回HTTP状态码304（不返回文件内容），客户端接到之后，就直接把本地缓存文件显示到浏览器中。如果时间不一致，就返回HTTP状态码200和新的文件内容，客户端接到之后，会丢弃旧文件，把新文件缓存起来，并显示到浏览器中（当文件发生改变，或者第一次访问时，服务器返回的HTTP头标签中有Last-Modified，告诉客户端页面的最后修改时间）。
第二种，浏览器把缓存文件的ETag，通过If-None-Match，来告诉Web服务器。思路与第一种类似。

一个例子：
Request Headers
Host localhost
User-Agent Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.8.1.16) Gecko/20080702 Firefox/2.0.0.16
Accept text/xml,application/xml,application/xhtml+xml,text/html;q=0.9,text/plain;q=0.8,image/png,*/*;q=0.5
...
`If-Modified-Since` Tue, 19 Aug 2008 06:49:35GMT
`If-None-Match` 7936caeeaf6aee6ff8834b381618b513
`Cache-Control` max-age=0

Response Headers
Date Tue, 19 Aug 2008 06:50:19 GMT
...
`Expires` Tue, 19 Aug 2008 07:00:19 GMT
`Last-Modified` Tue, 19 Aug 2008 06:49:35GMT
`Etag` 7936caeeaf6aee6ff8834b381618b513

对应以上两组缓存控制Header，按F5刷新浏览器和在地址栏里输入网址然后回车。这两个行为是不一样的。按F5刷新浏览器，浏览器会去Web服务器验证缓存。如果是在地址栏输入网址然后回车，浏览器会直接使用有效的缓存，而不会发http request去服务器验证缓存，这种情况叫做`缓存命中`。
Cache-Control: public 指可以`公有缓存`，可以是数千名用户共享的。
Cache-Control: private 指只支持`私有缓存`，私有缓存是单个用户专用的。
此外，针对不同的Cache-Control值，对浏览器执行不同的操作，其缓存访问行为也不一样，这些操作包括：打开新窗口、在地址栏回车、按后退按钮、按刷新按钮。


## TCP发送缓存与接收缓存
发送缓存用来暂时存放：
 1.发送应用程序传送给发送方 TCP `准备发送的`数据；
 2.TCP 已发送出但`尚未收到确认的`数据。

接收缓存用来暂时存放：
 1.`按序到达的、但尚未被接收应用程序读取的`数据；
 2.`不按序到达的`数据。

`注意`：
1.A 的发送窗口并不总是和 B 的接收窗口一样大（因为有一定的时间滞后）。
2.TCP 标准没有规定对不按序到达的数据应如何处理。通常是先临时存放在接收窗口中，等到字节流中所缺少的字节收到后，再按序交付上层的应用进程。
3.TCP 要求接收方必须有累积确认的功能，这样可以减小传输开销。


## TCP 最主要的特点
1.TCP 是`面向连接`的运输层协议。
2.每一条 TCP 连接`只能有两个端点`(endpoint)，每一条 TCP 连接只能是点对点的（一对一）。 
3.TCP 提供`可靠交付`的服务。
4.TCP 提供`全双工`通信。
5.面向字节流。

`注意`
1.TCP 连接是一条`虚连接`而不是一条真正的物理连接。
2.TCP 对应用进程一次把多长的报文发送到TCP 的缓存中是不关心的。
3.TCP `根据对方给出的窗口值和当前网络拥塞的程度来决定一个报文段应包含多少个字节`（`UDP 发送的报文长度是应用进程给出的`）。
4.TCP 可把太长的数据块划分短一些再传送。TCP 也可等待积累有足够多的字节后再构成报文段发送出去。

网络通信中，write返回成功后，是否确保数据发送成功或是被对端服务收到？
不是，只是表明待发送数据已被写入系统缓存；

## HTTP头Access-Control-Allow-Origin的作用
在某域名下使用Ajax向另一个域名下的页面请求数据，会遇到跨域问题。另一个域名必须在response中添加 Access-Control-Allow-Origin 的header，才能让前者成功拿到数据。
只有当目标页面的response中，包含了 Access-Control-Allow-Origin 这个header，并且它的值里有我们自己的域名时，浏览器才允许我们拿到它页面的数据进行下一步处理。
如果它的值设为 * ，则表示谁都可以用。

## 套接字的类型
①流式套接字（SOCK_STREAM）：提供面向连接、可靠的数据传输服务，数据无差错、无重复的发送，且按发送顺序接收。流式套接字实际上是基于TCP协议实现的；
②数据报式套接字（SOCK_DGRAM）：提供无连接服务。数据包以独立包形式发送，不提供无错保证，数据可能丢失或重复，并且接收顺序混乱。数据报式套接字实际上是基于UDP协议实现的。
③原始套接字（SOCK_RAW）


## TCP滑动窗口
TCP 把连接作为最基本的抽象。每一条 TCP 连接有两个端点。TCP 连接的端点不是主机，不是主机的IP 地址，不是应用进程，也不是运输层的协议端口。TCP 连接的端点叫做`套接字(socket)`或插口。端口号拼接到(contatenated with) IP 地址即构成了套接字。每一条 TCP 连接唯一地被通信两端的两个端点（即两个套接字）所确定。

以字节为单位的滑动窗口,根据 B 给出的窗口值
1.A 构造出自己的发送窗口
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_3.png)
2.A 发送了 11 个字节的数据
  P3 – P1 = A 的`发送窗口`（又称为通知窗口）
  P2 – P1 = 已发送但尚未收到确认的字节数
  P3 – P2 = 允许发送但尚未发送的字节数（又称为`可用窗口`） 
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_4.png)
3.A 收到新的确认号，发送窗口向前滑动
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_5.png)
4.A 的发送窗口内的序号都已用完，但还没有再收到确认，必须停止发送。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_6.png)

## TCP端口
端口用一个 16 位端口号进行标志。`端口号只具有本地意义`，即端口号只是为了标志本计算机应用层中的各进程。
三类端口:
1.`熟知端口`，数值一般为 0~1023。
2.`登记端口号`，数值为1024~49151，为没有熟知端口号的应用程序使用的。使用这个范围的端口号必须在 IANA 登记，以防止重复。
3.`客户端口号或短暂端口号`，数值为49152~65535，留给客户进程选择暂时使用。当服务器进程收到客户进程的报文时，就知道了客户进程所使用的动态端口号。通信结束后，这个端口号可供其他客户进程以后使用。

## 在TCP的拥塞控制中，什么是慢开始、拥塞避免、快重传和快恢复算法？这里每一种算法各起什么作用？“乘法减少”和“加法增大”各用在什么情况下？
答：慢开始：在主机刚刚开始发送报文段时可先将拥塞窗口 cwnd 设置为一个最大报文段 MSS 的数值。在每收到一个对新的报文段的确认后，将拥塞窗口增加至多一个 MSS 的数值。用这样的方法逐步增大发送端的拥塞窗口 cwnd，可以使分组注入到网络的速率更加合理。 
拥塞避免：当拥塞窗口值大于慢开始门限时，停止使用慢开始算法而改用拥塞避免算法。拥塞避免算法使发送端的拥塞窗口每经过一个往返时延RTT就增加一个MSS的大小。
快重传算法规定，发送端只要一连收到三个重复的 ACK 即可断定有分组丢失了，就应立即重传丢失的报文段而不必继续等待为该报文段设置的重传计时器的超时。
快恢复算法：(1) 当发送端收到连续三个重复的 ACK 时，就重新设置慢开始门限 ssthresh。
(2) 与慢开始不同之处是拥塞窗口 cwnd 不是设置为 1，而是设置为 ssthresh + 3 *MSS。 
(3) 若收到的重复的 ACK 为 n 个（n > 3），则将 cwnd 设置为 ssthresh + n * MSS。
(4) 若发送窗口值还容许发送报文段，就按拥塞避免算法继续发送报文段。
(5) 若收到了确认新的报文段的 ACK，就将 cwnd 缩小到 ssthresh。


## Tcp连接的建立和断开的过程
TCP用三次握手建立连接：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_1.png)
1.A的TCP向B发出连接请求报文段，其首部中的`同步位SYN = 1`，并选择`序号seq = x`，表明传送数据时的第一个数据字节的序号是 x。
2.B的TCP收到连接请求报文段后，如同意，则发回确认。B 在确认报文段中应使`SYN = 1`，使`ACK = 1`，其确认号`ack = x+1`，自己选择的序号`seq = y`。
3.A收到此报文段后向B给出确认，其`ACK = 1`，确认号`ack = y+1`。A 的 TCP 通知上层应用进程，连接已经建立。B 的 TCP 收到主机 A 的确认后，也通知其上层应用进程：TCP 连接已经建立。 

TCP连接的断开：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_2.png)
1.数据传输结束后，通信的双方都可释放连接。现在 A 的应用进程先向其 TCP 发出连接释放报文段，并停止再发送数据，主动关闭 TCP 连接。A 把连接释放报文段首部的`FIN = 1`，其序号`seq = u`，等待 B 的确认。
2.B发出确认，确认号`ack = u+1`，而这个报文段自己的序号`seq = v`。TCP 服务器进程通知高层应用进程。从 A 到 B 这个方向的连接就释放了，TCP 连接处于`半关闭状态`。B 若发送数据，A 仍要接收。
3.若B已经没有要向 A 发送的数据，其应用进程就通知TCP释放连接。
4.A收到连接释放报文段后，必须发出确认。在确认报文段中`ACK = 1`，确认号`ack = w+1`，自己的序号`seq = u + 1`。


## TCP可靠通信的实现原理
1.TCP 连接的每一端都必须设有两个窗口——一个`发送窗口`和一个`接收窗口`。
2.TCP 的可靠传输机制用字节的序号进行控制。`TCP所有的确认都是基于序号`而不是基于报文段。
3.TCP 两端的四个窗口经常处于动态变化之中。
4.TCP连接的往返时间 RTT 也不是固定不变的。需要使用特定的算法估算较为合理的重传时间。


## TCP的Nagle算法
事实上，Nagle算法所谓的“提高网络利用率”只是它的一个副作用，`Nagle算法的主旨在于“避免发送‘大量’的小包”`。Nagle算法并没有阻止发送小包，它只是阻止了发送大量的小包！
TCP/IP协议中，无论发送多少数据，总是要在数据前面加上协议头，同时，对方接收到数据，也需要发送ACK表示确认。为了尽可能的利用网络带宽，TCP总是希望尽可能的发送足够大的数据。Nagle算法就是为了尽可能发送大块数据，避免网络中充斥着许多小数据块。
`Nagle算法的基本定义是任意时刻，最多只能有一个未被确认的小段`。 所谓“小段”，指的是小于MSS尺寸的数据块，所谓“未被确认”，是指一个数据块发送出去后，没有收到对方发送的ACK确认该数据已收到。Nagle的算法通常会在TCP程序里添加两行代码，在未确认数据发送的时候让发送器把数据送到缓存里。任何数据随后继续直到得到明显的数据确认或者直到攒到了一定数量的数据了再发包。
默认情况下，发送数据采用Nagle 算法。这样虽然提高了网络吞吐量，但是实时性却降低了，在一些交互性很强的应用程序来说是不允许的，使用TCP_NODELAY选项可以禁止Nagle 算法。


## tcp/ip 同时打开,同时关闭
两个应用程序同时执行主动打开的情况是可能的，虽然发生的可能性较低。每一端都发送一个SYN,并传递给对方，且每一端都使用对端所知的端口作为本地端口。例如：
主机a中一应用程序使用7777作为本地端口，并连接到主机b 8888端口做主动打开。
主机b中一应用程序使用8888作为本地端口，并连接到主机a 7777端口做主动打开。
`tcp协议在遇到这种情况时，只会打开一条连接`。
这个连接的建立过程需要4次数据交换，而一个典型的连接建立只需要3次交换（即3次握手）
但多数伯克利版的tcp/ip实现并不支持同时打开。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_10.png)

如果应用程序同时发送FIN，则在发送后会首先进入FIN_WAIT_1状态。在收到对端的FIN后，回复一个ACK，会进入CLOSING状态。在收到对端的ACK后，进入TIME_WAIT状态。这种情况称为同时关闭。
同时关闭也需要有4次报文交换，与典型的关闭相同。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_11.png)


## SOCKET编程流程
基于TCP（面向连接）的socket编程流程：
	服务器端：
	①创建套接字；
	②将套接字绑定到一个本地地址和端口上；
	③将套接字设为监听模式，准备接收客户请求；
	④等待客户请求到来；当请求到来后接受连接请求，返回一个新的对应于此次连接的套接字；
	⑤用返回的套接字和客户端进行通信；
	⑥返回，等待另一客户请求；
	⑦关闭套接字；
	客户端：
	①创建套接字；
	②向服务器发出连接请求；
	③和服务器端进行通信；
	④关闭套接字；
基于UDP（面向无连接）的socket编程流程：
	服务器端：
	①创建套接字；
	②将套接字绑定到一个本地地址和端口上；
	③等待接收数据
	④关闭套接字；
	客户端：
	①创建套接字；
	②向服务器发送数据；
	③关闭套接字；


## HTTP Referer头的安全问题
Referer是HTTP协议中的一个请求报头，`用于告知服务器用户的来源页面`。比如说从Google搜索结果中点击进入了某个页面，那么该次HTTP请求中的Referer就是Google搜索结果页面的地址。如果某篇博客中引用了其他地方的一张图片，那么对该图片的HTTP请求中的Referer就是你那篇博客的地址。
一般Referer主要用于统计，像CNZZ、百度统计等可以通过Referer统计访问流量的来源和搜索的关键词（包含在URL中）等等，方便站长们有针性对的进行推广和SEO。

Referer另一个用处就是`防盗链`。可以用referrer-killer（一个js库）来实现反反盗链。

Referer是由浏览器自动加上的，`以下情况是不带Referer的`
（1）直接输入网址或通过浏览器书签访问
（2）使用JavaScript的Location.href或者是Location.replace()
（3）HTTPS等加密协议

Referer的安全问题：以新浪微博曾经的一个漏洞（新浪微博`gsid劫持`）为例
gsid是一些网站移动版的认证方式，移动互联网之前较老的手机浏览器不支持cookie，为了能够识别用户身份（实现类似cookie的作用），就在用户的请求中加入了一个类似“sessionid”的字符串，通过GET方式传递，带有这个id的请求，就代表你的帐号发起的操作。后来又因用户多次认证体验不好，gsid的失效期是很长甚至永久有效的（即使改了密码也无用哦，这个问题在很多成熟的web产品上仍在发生）。也就是说，一旦攻击者获取到了这个gsid，就等同于长期拥有了你的身份权限，对你的帐号做任意操作。
gsid这个非常重要的参数竟然就在URL里，只要攻击者在微博上给你发一个链接（指向攻击者的服务器），你通过手机点击进入之后，手机当前页面的URL就通过Referer主动送到了攻击者的服务器上，攻击者自然就可以轻松拿到你的gsid进而控制你的账号。


## OAuth 2.0授权方式
OAuth是一个关于授权（authorization）的开放网络标准，目前的版本是2.0版。
OAuth的作用就是让"客户端"（第三方应用）安全可控地获取"用户"的授权，与"服务商提供商"（平台，比如微信）进行互动。
OAuth在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。`"客户端"不能直接登录"服务提供商"，只能登录授权层`，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。"客户端"登录授权层以后，`"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料`。

OAuth 2.0的运行流程如下图：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_7.png)	

客户端的授权模式（步骤B）
OAuth 2.0定义了四种授权方式：
（1）`授权码模式`（authorization code） 适用于有server端的应用授权
是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。
（A）用户访问客户端，后者将前者导向认证服务器。
（B）用户选择是否给予客户端授权。
（C）假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。
（D）客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
（E）认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。
即：用一个URI去申请，获得用户授权后得到一个对应该URI的授权码。之后就可以用该URI+对应的授权码来获取一个令牌，之后就可以使用该令牌来通过授权层。

（2）`隐式授权`（implicit）	适用于通过客户端访问的应用授权
不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤，因此得名。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。
（A）客户端将用户导向认证服务器。
（B）用户决定是否给于客户端授权。
（C）假设用户给予授权，认证服务器将用户导向客户端指定的"重定向URI"，并在URI的Hash部分包含了访问令牌。
（D）浏览器向资源服务器发出请求，其中不包括上一步收到的Hash值。
（E）资源服务器返回一个网页(typically an HTML document with an embedded script)，其中包含的代码可以获取Hash值中的令牌。
（F）浏览器执行上一步获得的脚本，提取出令牌。
（G）浏览器将令牌发给客户端（客户端就可以凭借此令牌来获取数据）。
实例：

其中短暂停留的那个页面的url为：
https://www.zhihu.com/oauth/callback/login/qqconn?code=680726D150FF0B9DF2EBBE2EFEEEC0D4&state=7f13b99dc94e506e69ecb9ec83296eec
页面效果：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_8.png)
页面代码：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/net_9.png)


（3）`密码模式`（resource owner password credentials）
用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。这通常用在用户对客户端高度信任的情况下。

（4）`客户端模式`（client credentials）
指客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证。严格地说，客户端模式并不属于OAuth框架所要解决的问题。在这种模式中，用户直接向客户端注册，客户端以自己的名义要求"服务提供商"提供服务，其实不存在授权问题。


# 网络编程基本模型
所有的网络应用都是基于相同的基本编程模型，有着相似的整体逻辑结构，并且依赖相同的编程接口。每个网络应用都是基于客户端-服务器模型的。一个应用是由一个服务器进程和一个或多个客户端进程组成。
	客户端-服务器模型中的基本操作是事务，一个客户端-服务器事务由四步组成：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/basic_1.png)
注意：客户端和服务器是进程，而不是机器或者主机。
客户端和服务器端通过在“连接”上发送和接收字节流来通信。套接字是“连接”的端点。套接字=地址：端口。
	当客户端发起一个连接请求时，客户端套接字地址中的端口是由内核自动分配的，称为临时端口。然而服务器套接字地址中的端口通常是某个知名的端口，是和服务对应的。在Unix机器上，文件etc/services包含一张这台机器提供的服务以及它们的知名端口号的综合列表。
	套接字接口是一组用来结合Unix I/O函数创建网络应用的函数。大多数现代系统上都实现它，包括所有Unix变种、Windows、Macintosh系统。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/basic_2.png)
套接字地址存放在类型为sockaddr_in的16字节结构中。对于因特网应用，sin_family成员是AF_INTE，sin_port成员是一个16位的端口号，而sin_addr成员就是一个32位的IP地址。IP地址和端口号总是以网络字节顺序（大端法）存放的。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/basic_3.png)
客户端和服务器使用socket函数来创建一个套接字描述符：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/basic_4.png)
如：clientfd = Socket( AF_INET, SOCK_STREAM, 0);
Socket返回的clientfd描述符仅是部分打开的，并且不能用于读写。如何完成打开套接字的工作，取决于我们是客户端还是服务器。
客户端通过调用connect函数来建立和服务器的连接：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/basic_5.png)
connect函数试图与套接字地址为serv_addr的服务器建立一个因特网连接，其中addrlen是sizeof(sockaddr_in)。connect函数会阻塞，一直到连接成功建立或是发生错误，如果成功，sockfd描述符现在就准备好读写了，并且得到的连接是由套接字对：
(x:y, serv_addr.sin_addr:serv_addr.sin_port)刻画的。x，y分别表示客户端的IP地址和端口。
bind、listen、accept三个函数用来和客户端建立连接：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/basic_6.png)
bind函数告诉内核将my_addr中的服务器套接字地址和套接字描述符sockfd联系起来。
默认情况下内核会认为socket函数创建的描述符对应于主动套接字，它存在于一个连接的客户端。服务器调用listen函数告诉内核，描述符是被服务器而不是客户端使用的。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/basic_7.png)
listen函数将sockfd从一个主动套接字转化为一个监听套接字，该套接字可以接受来自客户端的连接请求。backlog参数暗示了内核在开始拒绝连接请求前应该放入队列中等待的未完成连接请求的数量，其确切含义要求对TCP/IP协议的理解。通常会被设置为一个较大的值，比如1024.
服务器通过调用accept函数来等待来自客户端的连接请求：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/basic_8.png)
accept函数等待来自客户端的连接请求，默认会阻塞进程，直到有一个客户连接建立后返回，它返回的是一个新的可用套接字，即“连接套接字”。参数中的listenfd是“监听套接字”，addr是一个结果参数，它用来接受一个返回值，指定客户端的地址。addrlen也是结果参数。
如果accept成功返回，则服务器与客户端已经正确建立连接了，此时服务器通过accept返回的套接字来完成与客户的通信。
“监听套接字”是作为客户端连接请求的一个端点，它只被创建一次，并存在于服务器的整个生命周期。“连接套接字”是客户端和服务器之间已经建立起来的连接的一个端点，服务器每次接受请求时，都会创建一次，只存在于服务器为一个客户端服务的过程。


