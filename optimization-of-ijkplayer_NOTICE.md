#		Optimization of Ijkplayer注意事项



### 配置文件

使用`ln -s module-default.sh module.sh`支持较多格式(完全编译，几乎所有格式都支持)，文件较大。使用该配置才会保证miracast投屏时不会出现偶尔无法起播的问题。

**（git仓库上已经是module-default.sh的配置，无需再修改）**



### NDK版本

使用**android-ndk-r21e**版本进行编译



### OPENSSL编译问题

在将OpenSSL升级至版本1.1.1o时需要注意的问题：

1、编译FFmpeg前，要把当前目录下的每个`ffmpeg-平台架构`的文件夹里面的configure改下，不然会报错找不到openssl。

```
use_pkg_config openssl openssl openssl/ssl.h OPENSSL_init_ssl ||
```

改成

```
use_pkg_config openssl openssl openssl/ssl.h OPENSSL_init_ssl ||
check_lib openssl openssl/ssl.h OPENSSL_init_ssl -lssl -lcrypto ||
```



另外，还需在该文件中删除以下代码(前提是你系统有libxml2库,debian/ubuntu可能叫做`libxml2-dev`,archlinux默认带，叫做`libxml2`)

```
require_pkg_config libxml2 libxml-2.0 libxml2/libxml/xmlversion.h xmlCheckVersion
```

参考链接：[hydrogenium2020-offical/ijkplayer: Fork from https://github.com/bilibili/ijkplayer and update openssl,android ndk support](https://github.com/hydrogenium2020-offical/ijkplayer/tree/master)



### FFMPEG编译问题

##### 编译./compile-ffmpeg.sh 时报错：

1、libavcodec/hevc_mvs.c:207:15: error: 'x0000000' undeclared (first use in this function)

解决办法：将libavcodec/hevc_mvs.c文件的变量B0改成b0，xB0改成xb0，yB0改成yb0

2、libavcodec/opus_pvq.c: In function 'quant_band_template':
libavcodec/opus_pvq.c:498:9: error: expected identifier or '(' before numeric constant
     int B0 = blocks;
解决办法：将libavcodec/opus_pvq.c文件的变量B0改成b0

参考链接：[Linux下编译ffmpeg-4.1，arm32, arm64, x86_ffmpeg arm linux 编译-CSDN博客](https://blog.csdn.net/qq_34732729/article/details/107761816)



##### 在编译armv7-a时，汇编文件报错: 

libswscale/arm/rgb2yuv_neon_common.S:251:11: error: unexpected token in argument list
CO_BV .dn d2.s16[2]
          ^
<instantiation>:4:5: error: invalid instruction
    vmul y16x16_l, n16x16_l, CO_RY
    ^
在编译ffmpeg4.3.1时没有这个问题，但是编译ffmpeg3.3.9时报错，简单粗暴的操作就是在ffmpeg configure选项中添加--disable-asm禁用汇编，但也可以选择只禁用neon，即在armv7-a的ffmpeg configure选项中加入--disable-neon

 **(已将该修复提交到git仓库，无需再修改)**    

参考链接：https://blog.csdn.net/mvp_Dawn/article/details/109125631

### 日志等级设置

若需要简化日志打印内容可在FFmpeg的<libavutil/log.c>文件中进行等级设置，修改static int av_log_level参数值。不同打印等级如下：

```c
/**

 \* Print no output.

 */

#define AV_LOG_QUIET   -8



/**

 \* Something went really wrong and we will crash now.

 */

#define AV_LOG_PANIC   0



/**

 \* Something went wrong and recovery is not possible.

 \* For example, no header was found for a format which depends

 \* on headers or an illegal combination of parameters is used.

 */

#define AV_LOG_FATAL   8



/**

 \* Something went wrong and cannot losslessly be recovered.

 \* However, not all future data is affected.

 */

#define AV_LOG_ERROR   16



/**

 \* Something somehow does not look correct. This may or may not

 \* lead to problems. An example would be the use of '-vstrict -2'.

 */

#define AV_LOG_WARNING  24



/**

 \* Standard information.

 */

#define AV_LOG_INFO   32



/**

 \* Detailed information.

 */

#define AV_LOG_VERBOSE  40



/**

 \* Stuff which is only useful for libav* developers.

 */

#define AV_LOG_DEBUG   48



/**

 \* Extremely verbose debugging, useful for libav* development.

 */

#define AV_LOG_TRACE   56
```




