---
title: 使用C++实现数字证书中心系统
date: 2017-12-16 11:28:42
tags: 
- C++
- OpenSSL
- certificate
---

## 概要
**OpenSSL_CA**使用C++语言编写，开发框架为QT(mingw)，目前使用的版本为**5.7.0**，~~理论上支持跨平台~~。 程序使用OpenSSL提供的API 目前已完成具备基本操作的1.0版本，正在重写2.0版本，包括以下工作：
*1.* 修复已知bug，重写函数   
*2.* 重新设计UI，分离服务器及客户端
*3.* 统一代码风格，合并重复的代码块
*4.* 添加部分新功能，调整逻辑

**开源地址**： [gitosc](http://git.oschina.net/rx_z/openssl_ca) | [github](https://github.com/zengrx/openssl_ca)

<!-- more -->

## 模块介绍
证书中心系统分为客户端和服务器端两部分，首先使用者在客户端生成**公私钥对**，生成**用户私钥**[filename].prikey，接下来填写相应的证书信息后生成**证书请求文件**[filename].csr，点击文件传输模块的发送按键即可将证书请求发送至证书中心。证书中心处于监听状态（目前接收文件结束后会自动关闭listen），接收到客户端请求后就会将证书请求文件存储在reqfiles文件夹内。相应的，中心验证、签发、撤销证书等操作也是在对应文件夹中完成，core文件夹储存所有操作相关的核心文件，如根证书、中心私钥、签发记录和撤销链等。
### 客户端：
用户使用客户端填写证书请求内容，并可设置相应的密钥长度、加密算法等，从而在某个指定目录上生成一个**证书请求文件**。通过文件传输模块将证书请求文件发送至服务器。
![证书中心客户端](http://upload-images.jianshu.io/upload_images/2424151-5d1d592e3ac7c9cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 服务器端：
服务器端能够接收用户的请求文件，使用自己的私钥签发证书，能够验证证书，能够生成撤销链并撤销证书或恢复被撤销的证书。在“其他”tab中目前实现了treeview用户附件文件浏览功能，方便使用者直接打开阅读用户上传的相关pdf、word或图片文件。
![证书中心](http://upload-images.jianshu.io/upload_images/2424151-b019b72bec53442c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 目录树：

          some path openssl_ca
                        ├─ca_client
                        │    ├─core
                        │    │    └─xxx.prikey
                        │    ├─executable
                        │    │    ├─CAClient.exe
                        │    │    └─some.dll
                        │    └─reqfile
                        │         └─xxx.csr
                        └─ca_server
                             ├─core
                             │    ├─Crl.crl
                             │    ├─rootca.crt
                             │    ├─rootca.prikey
                             │    ├─signlist.json
                             │    └─signSerial.txt
                             ├─executable
                             │    ├─CAServer.exe
                             │    └─some.dll
                             ├─reqfiles
                             │    └─xxx.csr
                             ├─reqfin
                             │    └─xxx.csr
                             └─signedfiles
                                  └─xxx.crt

## 文件说明：
### 客户端：
![生成私钥及证书请求文件流程](http://upload-images.jianshu.io/upload_images/2424151-51a1b144b4bcee27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

core文件夹中的xxx.prikey是用户在本地生成的私钥文件，由CAClient工程的**genkeypair.cpp**源文件实现。私钥并不是单独的生成，而是由RSA_generate_key函数生成**rsa公私钥对**，使用rsa类型变量接收该公私钥对之后，即可使用pem提供的函数PEM_write_bio_RSAPrivateKey生成私钥文件，私钥文件可以选择是否使用口令加密，如果使用了口令加密，则需要设置相应的参数，如加密口令、口令长度、加密方式等。
reqfile中存储了用户生成的证书请求文件，在该工程的**certreq.cpp**源文件中实现。在生成请求文件的函数中要求传入之前生成的rsa公私钥对，因为在生成csr请求文件时需要使用私钥进行签名，并且要将匹配的公钥写入，并放置摘要值。
文件传输模块不详细描述

**生成私钥过程主要函数：**
```cpp
//new一个EVP类型公钥对象
pkey = EVP_PKEY_new();
//生成公私钥对
rsapair = RSA_generate_key(bits, e, NULL, NULL);
//BIO类型对象，写入用户私钥
bp = BIO_new_file(/*[char类型文件名称]*/, "w");
//生成无口令保护私钥(PEM)
PEM_write_bio_RSAPrivateKey(bp, rsapair, NULL,NULL ,0, NULL, NULL);
//生成使用口令保护的私钥，参数3为私钥加密算法，参数4为口令
PEM_write_bio_RSAPrivateKey(bp, rsapair, des, [pwd], strlen([pwd]), NULL, NULL);
```
**生成证书请求过程主要函数：**
```cpp
//x509类型对象 存储请求文件
X509_REQ *req;
/*函数中需要使用之前生成的rsapair密钥对*/
//设置版本号
X509_REQ_set_version(req, version);
//开始读取用户输入内容并写入请求entry
//写入用户信息基本就用这三行类推
name = X509_NAME_new();
entry = X509_NAME_ENTRY_create_by_txt(&entry, "commonName",
                                          V_ASN1_UTF8STRING,
                                          (unsigned char *)bytes, len);
X509_NAME_add_entry(name, entry, 0, -1);
//subject name
X509_REQ_set_subject_name(req, name);
//EVP类型数据 存储用户公钥
pkey = EVP_PKEY_new();
//采用bit->rsa->pkey模式
EVP_PKEY_assign_RSA(pkey, rsapair);
//将公钥放入证书请求中    
X509_REQ_set_pubkey(req, pkey);
//设置摘要三步
md = EVP_sha1();
X509_REQ_digest(req, md, (unsigned char *)mdout, (unsigned int*)&mdlen);
X509_REQ_sign(req, pkey, md);
//使用这个函数就可以生成PEM格式的请求了 b为BIO类型数据 同bp
PEM_write_bio_X509_REQ(b, req);
```
### 服务器端：
core文件夹中存储了Crl.crl[证书撤销链]、rootca.crt[证书中心根证书]、rootca.prikey[证书中心私钥]以及标记签名信息的json和txt文件。signlist.json内容为该中心签发的所有证书记录，包含证书序列号、签发日期及证书目前状态（是否被撤销）。serialNumber.txt文件保存了中心目前签发到的序列数目+1，可以作为下一个签发证书的序列号，有空整合进json文件中。
reqfiles文件夹中存储中心接收到但还没有签发的[xxx.csr]证书请求文件。相应的，当中心成功签发证书后，该证书请求文件就会被移动到reqfin文件夹中，而签发产生的[xxx.crt]用户证书文件就存在signedfiles文件夹中。
在CAServer工程中，中心签发实现在rootsign.cpp中、撤销证书实现在rvkcert.cpp中、证书验证实现在rootverify.cpp中、json操作在jsonoper.cpp、文件接收在filerecv.cpp中实现。

####根证书签名
根证书签名tab中包含选择文件、选择签发天数、根证书签名按钮、CA中心签发记录及快捷撤销按钮。按照代码实现顺序，根证书签名是服务器中首次使用到中心公私钥对，所以用于**加载证书中心根证书及密钥函数**loadCert与loadKey的实现就放在该源文件内，实际上在主函数中封装了名为loadRootCA的函数，该函数调用了上述两个子函数，以减少代码冗余。
**根证书签名过程主要函数**
```cpp
//在X509 * MainWindow::loadCert()函数中
//定义X509类型变量，存储将要载入的根证书
X509 * x509 = NULL; 
//定义BIO类型变量，存储直接读取的根证书
BIO * in = NULL;
//读取证书存入in中
in = BIO_new_file([rootca.crt],"r"); //读取根证书
//将BIO类型变量读取到X509对象中，至此已获取证书（包含公钥）内容
x509 = PEM_read_bio_X509(in, NULL, NULL, NULL);
    
//在EVP_PKEY * MainWindow::loadKey()函数中
//EVP_PKEY类型数据，存储私钥
EVP_PKEY *pkey = NULL;
//同理
BIO * in = NULL;
in = BIO_new_file([rootca.prikey], "r");
//将BIO类型变量读取到X509对象中，获得私钥内容
pkey = PEM_read_bio_PrivateKey(in, NULL, 0, NULL);
    
//bool MainWindow::createCertFromRequestFile()作为最基本实现签名函数
//拿到之前获取的根证书及私钥值
X509 * rootCert = xxx; 
EVP_PKEY * rootKey = xxx;
//用户证书、公钥及请求文件对象，用法同理根CA
X509 * userCert = NULL;
EVP_PKEY * userKey = NULL;
X509_REQ *req = NULL;
BIO *in;
in = BIO_new_file([requestFile], "r");
//从请问文件中获取信息
req = PEM_read_bio_X509_REQ(in, NULL, NULL, NULL);
//从请求文件中获取公钥
userKey = X509_REQ_get_pubkey(req);
//用于生成证书的x509对象 
userCert = X509_new(); 
//设置版本
X509_set_version(userCert, 2);
//设置用户证书序列号
ASN1_INTEGER_set(X509_get_serialNumber(userCert),  [serialNumber]);
//设置证书有效期起始时间
X509_gmtime_adj(X509_get_notBefore(userCert), 0);
//设置证书有效期结束时间，差值days即为有效日
X509_gmtime_adj(X509_get_notAfter(userCert), (long)60 * 60 * 24 * [days]);
//将公钥载入至用户证书
X509_set_pubkey(userCert, userKey); 
//设置证书公钥信息
X509_set_subject_name(userCert, req->req_info->subject);
//设置签发者信息
X509_set_issuer_name(userCert, X509_get_issuer_name(rootCert));
//使用根CA私钥签名
X509_sign(userCert, rootKey, EVP_sha1()); 
//数据类型对象同理
BIO * bcert = NULL, *bkey = NULL;
//按格式签发用户证书并生成私钥
//签发DER类型证书
i2d_X509_bio(bcert, userCert);
//签发PEM类型证书<--用这个
PEM_write_bio_X509(bcert, userCert);
```
#### 根证书验证
根证书验证tab中主要完成根证书验证，根据接收到的证书文件，依次检查是否是本CA中心签发证书、是否被撤销以及是否处于有效期内，当**三项检查全部返回真值**时才能说明该待验证证书通过中心验证，否则都认定为无效证书。所以在代码中，每一次检测的按钮点击事件都调用rootCaVerify函数，通过该函数进一步调用checkByRootCert、checkByCrl、checkByTime三个子函数进行验证。
**根证书验证过程主要函数**
```cpp
//根证书检查，四行
//传值
X509 *x509 = [usercert]; 
X509 *root = [rootcert];
//获取公钥
EVP_PKEY * pcert = X509_get_pubkey(root); 
//根CA使用私钥签名，带入公钥检查即可
X509_verify(x509,pcert); 
  
//撤销链序列号检查
//为用户证书及撤销链对象传值
X509 *x509 = [usercert]
X509_CRL *crl = [certop.crl];
//获取证书吊销链内容
STACK_OF(X509_REVOKED) *revoked = crl->crl->revoked;
//撤销链对象
X509_REVOKED *rc;
//获取待验证书序列号
ASN1_INTEGER *serial = X509_get_serialNumber(x509);
//获取撤销链长度
int num = sk_X509_REVOKED_num(revoked);
//循环判断撤销链中是否有值与待验证书序列号相等
for(int i=0; i<num; i++)
{
    rc = sk_X509_REVOKED_value(revoked,i);
    //cmp函数返回0则if为真，表明证书已被撤销
    if(ASN1_INTEGER_cmp(serial,rc->serialNumber)==0)
}
    
//证书有效期检查
//获取当前系统时间
time_t ct = QDateTime::currentDateTime().toTime_t();
//两行获取证书时限前后时间
asn1_string_st *before = X509_get_notBefore(x509); 
asn1_string_st *after = X509_get_notAfter(x509);
//数据类型转换
ASN1_UTCTIME *be = ASN1_STRING_dup(before), *af = ASN1_STRING_dup(after)
//满足此签发起始时间大于当前时间或结束时间小于当前时间则表明证书不在有效期限
//体现在windows证书文件中内容为此证书不在有效期内或未被使用
ASN1_UTCTIME_cmp_time_t(be,ct)>=0||ASN1_UTCTIME_cmp_time_t(af,ct)<=0
```
#### 根证书撤销
撤销（吊销）功能类似于将CA签发的某证书序列号添加入黑名单，在以后检查证书合法性的时候只需**读出序列号**并循环判断撤销链中是否存在。所以在证书撤销功能模块中需要实现的内容基本有如下几点：
*1.* 生成根证书撤销链
*2.* 初始化撤销链
*3.* 通过序列号撤销证书
*4.* 显示撤销链内容
*5.* 恢复被撤销的证书
*6.* 其他，如防止重复撤销证书，防止提前撤销未发布序列号等
只有实现了撤销链后，证书验证部分中“是否被撤销”功能才能正常使用。
##### 生成撤销链
目前有一个bug。在根证书撤销tab中有一个生成撤销链的按钮事件，点击可以在目录中重新生成一个撤销链文件。但目前新生成撤销链的内容为空，这样则与json中储存的签发信息发生冲突，导致快捷撤销/恢复功能发生异常。有空解决
**根证书撤销过程主要函数**
```cpp
//生成根证书撤销链
//new一个crl
X509_CRL = x509_CRL_new();
//设置版本
X509_CRL_set_version(crl,3);
//设置颁发者
issuer = X509_NAME_dup(x509->cert_info->issuer);
X509_CRL_set_issuer_name(crl,issuer);
//设置上次发布时间
lastUpdate = ASN1_TIME_new();
ASN1_TIME_set(lastUpdate,t);
X509_CRL_set_lastUpdate(crl,lastUpdate);
//设置下次发布时间
nextUpdate = ASN1_TIME_new();
//设置下次发布时间，默认1000个单位
ASN1_TIME_set(nextUpdate,t+1000);
X509_CRL_set_nextUpdate(crl,nextUpdate);
//签名
X509_CRL_sign(crl,pkey,EVP_md5());
//生成CRL
PEM_write_bio_X509_CRL(bp,crl);
 
//初始化撤销列表
//关键在于得到撤销列表的长度及revoked数据结构
//定义，传值
STACK_OF(X509_REVOKED) *revoked;
revoked = crl->revoked;
//拿到长度用于循环比对
num = sk_X509_REVOKED_num(revoked);
//定义rc
X509_REVOKED *rc
//在一个num长度的循环中使用rc接收revoked对应i的值
rc = sk_X509_REVOKED_value(revoked, i);
//rc->revocationDate存储了证书被撤销的时间
//rc->serialNumber存储了被撤销函数的序列号
   
//验证是否撤销等操作
//这些操作套路非常一致，读出证书序列号与CRL比较
//待验证证书序列号
ASN1_INTEGER *serial
//循环cmp，结果为0说明两值相等即证书被撤销
ASN1_INTEGER_cmp(serial,rc->serialNumber)
    
//恢复被撤销的证书
//之前的操作都差不多
//使用delete函数将对应证书的序列号删除即可
sk_X509_REVOKED_delete(crl->revoked, [index])
```

## 算是总结
国庆前一周回学校验课设，验收过程中通过与老师的交流，结合自己之前的一些想法，总结一些有待改善的功能和可以添加的内容。

### 数据库的建立：
目前的CA项目只是作为从证书请求文件的生成到证书被签发等一系列过程的实现，并不是一个可以实际使用的系统。当初为了方便，并且数据量不大，选择了使用JSON来存储签发数据。如需完善，对CA使用者及其所属公司机构要有比较详尽的资料存储，或证书请求文件、用户公私钥hash等信息记录，还是应该建立与数据库的连接。

### 根证书验证：
目前根证书验证的默认文件夹路径是根证书签发公钥目录，讲道理当初是为了测试方便才这么写的⁄(⁄ ⁄•⁄ω⁄•⁄ ⁄)⁄，实际使用过程中，大多应该由通信某方用户向CA中心提交对方公钥，进行使用前的确认。所以创建一个专门接收用户提交公钥的文件夹还是很有必要的，虽说根证书验证是服务器模块中唯一一个没有被写死路径的文件浏览对象。可以在客户端文件传输功能中添加与文件类型对应的字符串标记，例如[.csr]-AaA，[.crt]-BbB等；或者在服务器接收到文件后根据后缀名判断并移动文件。不过这么一想用FTP真是不要太方便╮（╯＿╰）╭

### 公钥MD5记录：
由于密钥对是在客户端生成的，很有可能在各自生成伪随机数，再产生密钥对时出现某些奇葩情况。所以服务器在签名生成证书文件之后可以记录公钥文件的指纹，记录到对应的数据库列中。

### 其他服务器端的保护操作：
类似对服务器文件接收的保护，上传文件的限制，上传信道的安全加固，shell的查杀等，这个方面的扩展就很多了。