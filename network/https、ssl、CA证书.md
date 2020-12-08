

0. 客户端向服务端发送https请求（服务端443端口）， 发送的信息为：随机值random1和支持的加密算法列表。
1. 服务端接收到信息之后返回客户端握手信息，包扩随机值random2和协商号的加密算法。
2. 随机服务端发送第二个相响应包：数字证书。
3. 证书校验：

​       CA私钥ca_private_key加密证书cert摘要hashA=签名sig;客户端请求服务端，服务端先返回cert和sig，使用同样的摘要算 法得到证书cert的摘要hashB, 使用CA的公钥ca_public_key(内置在系统或者浏览器中的信赖证书里)对sig进行解密得hashDecode。如果hashB和hashDecode相等，那么证书就是信任的CA机构颁发的证书。

4. 客户端证书通过后，生成random3,使用证书中的公钥加密random3（记为preMasterKey）以Client Key Exchange报文发送到服务端。此后发送Change cipher Spec报文表示此后数据传输进行加密传输.
5. 服务端收到会话密钥后使用私钥解密preMasterKey得到random3，此后发送Change cipher Spec报文表示此后数据传输进行加密传输。 此事客户端和服务端都知道了random1,random2,random3的值了，客户端和服务端使用同样的加密算法(对称加密传输)对random1,random2,random3进行加密得到会话密钥MasterKey。后面的数据传数据都用这个密码进行加密解密，由于random3是通过非对称加密传输的，所以能保证安全。

