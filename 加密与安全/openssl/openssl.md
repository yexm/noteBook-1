# OpenSSL

## 1. 网络通讯基础

![](/Linux/openssl/assets/网络模型.png)

同一主机间的进程通信： IPC，消息队列Message queue, shm, semerphor

不同主机间的通信： socket（IP:PORT），使用port标志进程地址，进程向内核注册使用某个端口，独占使用。

## 2. OpenSSL

**安全目标：**

* 保密性：confidentiality
* 完整性：integrality
* 可用性：availability

**攻击类型：**

* 威胁保密性：窃听、通信量分析；
* 威胁完整性：更改、伪装、重放、否认
* 威胁可用性：拒绝服务

**解决方案：**

技术（加密和解密）、服务（用于抵御攻击的服务，也是为了上述安全目标而特地设计的安全服务）

**加密和解密：**

传统加密方法：替代加密算法，置换加密算法

现代加密方法：现代块加密方法

**服务：**

认证机制

访问控制机制

**密钥算法和协议：**

* 对称加密：加密解密使用同一密钥。分割原始数据为固定大小块，逐个进行加密，密钥分发困难；

  ```
   DES\(Data Encryption Standard\)\(64位密钥\)【非安全】——&gt;3DES：Triple DES;

   AES\(Advanced Encryption Standard\)\(128,192,256,384位密钥\)

   Blowfish

   Twofish

   IDEA

   RC6

   CAST5
  ```

* 非对称加密：密钥分为公钥与私钥，私钥自己留存，公钥可公开；用公钥加密数据只能使用与之配对的私钥解密，反之亦然；私钥加密公钥解密，实现数字签名；密钥交换，使用对方公钥加密对称密钥。RSA,DSA\(不可加解密\),ELGamal

* 单向加密：只能加密不能解密，提取数据指纹（特征码hash），定长输出，雪崩效应；只要用于完整性验证；常见算法：MD5（128位），sha1（160位），sha224，sha256，sha384，sha512

* 密钥交换：IKE（Internet Key Exchange），常见实现：公钥加密，DH算法

![点对点加密通信流程](/Linux/openssl/assets/点对点加密流程.png)

存在的问题，提供的公钥是否可信，可进行中间人攻击，因此需要引入CA解决此问题。

## 3. PKI

**基础组件：**

* 签证机构：CA
* 注册机构：RA
* 证书吊销列表：CRL
* 证书存取库

**X.509数字证书结构定义，以及认证协议标准**

* 版本号
* 序列号
* 签名算法ID
* 发行者名称
* 有效期限
* 主体名称
* 主体公钥
* 发行者的唯一标识
* 主体的唯一标识
* 扩展
* 发行者的签名

SSL：Secure Sockets Layer

TLS：Transport Layer Security，事实标准；分层设计：1.最底层：基础算法原语；2.算法实现；3.组合算法实现；4.组件拼装密码学协议软件

ssl访问流程

![](/Linux/openssl/assets/Ssl_handshake_with_two_way_authentication_with_certificates.png)

## 4. Linux系统安全实现：OpenSSL、GPG

OpenSSL由三部分组成：

* libencrypt
* libssl
* openssl



