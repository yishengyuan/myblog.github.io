## 课程地址：
https://www.bilibili.com/video/BV1Vt411X7JF?share_source=weixin&vd_source=d98ea4bfa03d1add090b065c10d92495&spm_id_from=333.788.player.switch&p=10

白皮书 和黄皮书分别指什么？ 


区块链发展到了中期早期

![[Pasted image 20251118085828.png]]

![[Pasted image 20251118085837.png]]

crypto-currency 
crptographic hash function 
collision resistance  hash碰撞
256 hash值 

H(m)  文件修改后hash值会被修改 

MD5 目前已经不安全了  可以人为hash碰撞

digital commitment 
sealed envelope 


SHA-256. 
Secure Hash Algorithm

public key  private key  公钥 和私钥 

256位 超级计算机产生公私钥对无法

asymmetric encryption algorithm 非对称加密 

比特币中主要用到了密码学中两个功能：1.哈希 2.签名。密码学中的哈希函数（cryptographtic hash function） 一、哈希函数 哈希函数主要有三个特性：1、碰撞特性（collision resistance）；2、隐秘性（Hiding）；3、谜题友好（puzzle friendly）。 1、collision resistance collision resistance 即为输入两个输入值X，Y，经过哈希函数之后得到H(X)=H(Y)，即为哈希碰撞。利用哈希碰撞，可以检测对信息的篡改。假设输入x1，哈希值为H(x1)，此时很难找到一个x2，使得H(x1)=H(x2)。 2、Hiding hiding 意思是哈希函数的计算过程是单向的，不可逆的。但前提要满足输入控件足够大，且分布均匀。通常我们在实际操作中会使用添加随机数的方法。假设给定一个输入值X，可以算出哈希值H(x)，但是不能从H(x)推出X。 3、puzzle friendly 通常我们限定输出的哈希值在一定范围内，即H(block header + nonce) < target （block header是区块链的链头），这个确定链头范围的过程即为挖矿。 二、签名 签名就相当于每个人的开户行账号。公钥签名，即开户账号，验证签名用私钥，即为账号密码。以此来确保比特币的安全传输。




