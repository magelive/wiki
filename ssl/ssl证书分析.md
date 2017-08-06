# ssl证书分析
SSL(Secure socket Layer 安全套接层协议)指使用公钥和私钥技术组合的安全网络通讯协议。
SSL协议指定了一种在应用程序协议(如Http、Telenet、NMTP和FTP等)和TCP/IP协议之间提供数据安全性分层的机制，它为TCP/IP连接提供数据加密、服务器认证、消息完整性以及可选的客户机认证，主要用于提高应用程序之间数据的安全性，对传送的数据进行加密和隐藏，确保数据在传送中不被改变,即确保数据的完整性。

SSL 以对称密码技术和公开密码技术相结合，可以实现如下三个通信目标：
1. 秘密性: SSL客户机和服务器之间传送的数据都经过了加密处理,网络中的非法窃听者所获取的信息都将是无意义的密文信息	
2. 完整性: SSL利用密码算法和散列(HASH)函数,通过对传输信息特征值的提取来保证信息的完整性,确保要传输的信息全部到达目的地,可以避免服务器和客户机之间的信息受到破坏。
3. 认证性:利用证书技术和可信的第三方认证，可以让客户机和服务器相互识别对方的身份。为了验证证书持有者是其合法用户(而不是冒名用户)，SSL要求证书持有者在握手时相互交换数字证书，通过验证来保证对方身份的合法性。

OpenSSL  简单地说,OpenSSL是SSL的一个实现,SSL只是一种规范.理论上来说,SSL这种规范是安全的,目前的技术水平很难破解,但SSL的实现就可能有些漏洞,如著名的"心脏出血".OpenSSL还提供了一大堆强大的工具软件,强大到90%我们都用不到.

## 证书标准
X.509 - 这是一种证书标准,主要定义了证书中应该包含哪些内容.其详情可以参考RFC5280,SSL使用的就是这种证书标准.

## 编码格式
同样的X.509证书,可能有不同的编码格式,目前有以下两种编码格式.

### PEM 
Privacy Enhanced Mail,打开看文本格式,以"-----BEGIN..."开头, "-----END..."结尾,内容是BASE64编码.
查看PEM格式证书的信息:openssl x509 -in certificate.pem -text -noout
Apache和*NIX服务器偏向于使用这种编码格式.

### DER 
Distinguished Encoding Rules,打开看是二进制格式,不可读.
查看DER格式证书的信息:openssl x509 -in certificate.der -inform der -text -noout
Java和Windows服务器偏向于使用这种编码格式.



## prive key
## public key

## 参考
[SSL、TLS协议格式入门学习](http://www.tuicool.com/articles/rQjEzy3)
[那些证书相关的玩意儿(SSL,X.509,PEM,DER,CRT,CER,KEY,CSR,P12等)](http://www.cnblogs.com/guogangj/p/4118605.html)