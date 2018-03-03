---
title: android OTA 升级包校验分析
date: 2018-03-03 21:06:19
tags:
---

做一个安卓系统OTA校验的记录
### 0x00  singal.emit()
去年七八月份的时候看同事出过一个插卡自动升级脚本，核心功能非常简单了，把外置卡的update.zip拷贝到内置卡，写升级命令到command后reboot recovery
今年过完年回来客户报了OTA的问题，给到的截图显示升级包校验失败，因为recovery里基本是G家源码没有改动过，所以基本可以判断是升级包不完整。保险起见嘛还是过一遍源码
### 0x01  源码
手上是一套4.4，OTA相关的内容在 path_to_source/bootable/recovery 路径下。关于安卓recovery和main
 system的逻辑就不多说，结尾贴两篇大佬分析的文章看看基本ojbk。这里主要还是拿升级包对着verifier.cpp分析一波

<!-- more -->

#### 从install开始
+ 先看调用verify_file()方法的install.cpp
```cpp
static int
really_install_package(const char *path, int* wipe_cache)
{
    ui->SetBackground(RecoveryUI::INSTALLING_UPDATE);
    ui->Print("Finding update package...\n");
    // Give verification half the progress bar...
    ui->SetProgressType(RecoveryUI::DETERMINATE);
    ui->ShowProgress(VERIFICATION_PROGRESS_FRACTION, VERIFICATION_PROGRESS_TIME);
    LOGI("Update location: %s\n", path);

    //-----------------1-找文件-----------------//
    if (ensure_path_mounted(path) != 0) {  
        LOGE("Can't mount %s\n", path);
        return INSTALL_CORRUPT;
    }

    ui->Print("Opening update package...\n");

    int numKeys;
    Certificate* loadedKeys = load_keys(PUBLIC_KEYS_FILE, &numKeys);
    if (loadedKeys == NULL) {
        LOGE("Failed to load keys\n");
        return INSTALL_CORRUPT;
    }
    LOGI("%d key(s) loaded from %s\n", numKeys, PUBLIC_KEYS_FILE);

    ui->Print("Verifying update package...\n");

    int err;
    //-----------------2-校验包-----------------//
    err = verify_file(path, loadedKeys, numKeys);
    free(loadedKeys);
    LOGI("verify_file returned %d\n", err);
    if (err != VERIFY_SUCCESS) {
        LOGE("signature verification failed\n");
        return INSTALL_CORRUPT;
    }

    /* Try to open the package.
     */
    ZipArchive zip;
    //-----------------3-解包-----------------//
    err = mzOpenZipArchive(path, &zip);
    if (err != 0) {
        LOGE("Can't open %s\n(%s)\n", path, err != -1 ? strerror(err) : "bad");
        return INSTALL_CORRUPT;
    }

    /* Verify and install the contents of the package.
     */
    ui->Print("Installing update...\n");
    //-----------------4-安装-----------------//
    return try_update_binary(path, &zip, wipe_cache);
}
```
在recovery/install.cpp的really_install_package函数中可以看到，先ensure_path_mounted()找到包，再load_keys()加载密钥，之后调用verify_file()做验证。所以验证的时候并没有解压OTA文件
#### 进入verifer源文件
+ 读一下写在verifier.cpp最前面的注释：
> // Look for an RSA signature embedded in the .ZIP file comment given
// the path to the zip.  Verify it matches one of the given public
// keys.
//
// Return VERIFY_SUCCESS, VERIFY_FAILURE (if any error is encountered
// or no key matches the signature).

一旦触发了解析中列出的错误情况或没有任何RSA公钥匹配，小机器人就(扑po)倒(街gai)了：
![要主系统亲亲才能起来_(:⁍」∠)_](http://upload-images.jianshu.io/upload_images/2424151-d9d0b216c2c6b755.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 校验压缩文件脚注
+ 读注释
> // An archive with a whole-file signature will end in six bytes:
    //
    //   (2-byte signature start) $ff $ff (2-byte comment size)
    //
    // (As far as the ZIP format is concerned, these are part of the
    // archive comment.)  We start by reading this footer, this tells
    // us how far back from the end we have to start reading to find
    // the whole comment.

一个整包签名的文件以6个字节结尾。通过读取6字节脚注就可以定位并找到所有的（签名）注释
就是这个 -> (2-byte signature start) $ff $ff (2-byte comment size)
看起来是这样的：
![ota_footer](http://upload-images.jianshu.io/upload_images/2424151-af0c914e261d5477.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**所以可以得到：signature start是06B8(十进制值1720)，comment size是06CA(十进制值1738)**
这里的几个判断
```cpp
// 移不动指针
if (fseek(f, -FOOTER_SIZE, SEEK_END) != 0) {
        LOGE("failed to seek in %s (%s)\n", path, strerror(errno));
        fclose(f);
        return VERIFY_FAILURE;
}
```
```cpp
// 读不到footer
if (fread(footer, 1, FOOTER_SIZE, f) != FOOTER_SIZE) {
    LOGE("failed to read footer from %s (%s)\n", path, strerror(errno));
    fclose(f);
    return VERIFY_FAILURE;
}
```
```cpp
// footer中间两byte不是FFFF
if (footer[2] != 0xff || footer[3] != 0xff) {
    LOGE("footer is wrong\n");
    fclose(f);
    return VERIFY_FAILURE;
}
```
```cpp
// 签名开始位置减去footer大小（就是6）放不下签名
// RSANUMBYTES大小应该是256
if (signature_start - FOOTER_SIZE < RSANUMBYTES) {
    // "signature" block isn't big enough to contain an RSA block.
    LOGE("signature is too short\n");
    fclose(f);
    return VERIFY_FAILURE;
}
```
第四个条件结合文件来分析：
![signature block size](http://upload-images.jianshu.io/upload_images/2424151-3c554ec7e937bc76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从signature start到文件结尾的整个区域大小1720，所以放signature comment部分大小为1714

##### 校验EOCD
EOCD即end of central directory，跟在目录列出的最后一个文件名之后，并以一个魔术字 50 4B 05 06 开头，长度为22字节
![EOCD](http://upload-images.jianshu.io/upload_images/2424151-c514824109a3e0c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

又是几个错误情况判断
```cpp
// eocd_size = comment_size + EOCD_HEADER_SIZE
// 不想算就winhex连跳两次⊙ω⊙
// comment_size大小就是06CAH，EOCD头为22字节
// 移不动指针
if (fseek(f, -eocd_size, SEEK_END) != 0) {
    LOGE("failed to seek in %s (%s)\n", path, strerror(errno));
    fclose(f);
    return VERIFY_FAILURE;
}
```
```cpp
// malloc不到read不到
unsigned char* eocd = (unsigned char*)malloc(eocd_size);
if (eocd == NULL) {
    LOGE("malloc for EOCD record failed\n");
    fclose(f);
    return VERIFY_FAILURE;
}
if (fread(eocd, 1, eocd_size, f) != eocd_size) {
    LOGE("failed to read eocd from %s (%s)\n", path, strerror(errno));
    fclose(f);
    return VERIFY_FAILURE;
}
```
```cpp
// 找不到魔术字或魔术字出现多次
if (eocd[0] != 0x50 || eocd[1] != 0x4b ||
    eocd[2] != 0x05 || eocd[3] != 0x06) {
    LOGE("signature length doesn't match EOCD marker\n");
    fclose(f);
    return VERIFY_FAILURE;
}

size_t i;
for (i = 4; i < eocd_size-3; ++i) {
    if (eocd[i  ] == 0x50 && eocd[i+1] == 0x4b &&
        eocd[i+2] == 0x05 && eocd[i+3] == 0x06) {
        // if the sequence $50 $4b $05 $06 appears anywhere after
        // the real one, minzip will find the later (wrong) one,
        // which could be exploitable.  Fail verification if
        // this sequence occurs anywhere after the real one.
        LOGE("EOCD marker occurs after start of EOCD\n");
        fclose(f);
        return VERIFY_FAILURE;
    }
}
```
##### 校验签名
这里熟悉openssl的话没什么好看了，核心就是RSA_verify()
```cpp
// The 6 bytes is the "(signature_start) $ff $ff (comment_size)" that
// the signing tool appends after the signature itself.
if (RSA_verify(pKeys[i].public_key, eocd + eocd_size - 6 - RSANUMBYTES,
               RSANUMBYTES, hash, pKeys[i].hash_len)) {
    LOGI("whole-file signature verified against key %d\n", i);
    free(eocd);
    return VERIFY_SUCCESS;
} else {
    LOGI("failed to verify against key %d\n", i);
}
```

### 0x02 画个图
哇 libreoffice 好难用。。回家再画

### 0x03 Reference
强推三篇文章
[Android签名与校验过程详解](http://blog.csdn.net/gulinxieying/article/details/78677487)
[Android Recovery OTA升级（二）—— Recovery源码解析](http://blog.csdn.net/wzy_1988/article/details/46862247)
[Android recovery文集](http://blog.csdn.net/csdn66_2016/article/category/6762933)

### 0x04 完结撒花
这篇文章算是源码OTA校验部分分析的一点记录，要是能结合signtools的角度去看升级包是怎么签名怎么加comment啊怎么写EOCD等等就更好了(ง •_•)ง