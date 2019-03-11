---
title: 宝塔面板PHP7安装Oracle扩展
date: 2019-03-11 11:05:45
tags: 宝塔,PHP,Oracle
---

## 前言
------------
自从认识了宝塔面板这个好东西之后，我觉得我可以把我用了4年之久的Kloxo-MR面板换掉了。具体是如何的面板以及其如何的好用，我在此就不再“推销”了。可以进入[宝塔官网](http://www.bt.cn)看一看。
## 需求
--------------
1. 最近做一个项目，要求PHP的后端连接Oracle数据库，原想使用宝塔的“安装扩展”的功能来简单添加，可是并没有发现有该选项的存在。所以只能求助于自己编译和安装扩展了
2. 虽然本教程是在安装有bt面板下进行操作的，但是这个安装方法其实是通用的，只是路径可能有所不同。
## 安装
----------------
### 环境
* CentOS 7.4 x86_64
* PHP 7.0.19
* Apache24
* InstantClient 12.2
### Oracle InstantClient 安装
1. [下载地址](http://www.oracle.com/technetwork/database/database-technologies/instant-client/downloads/index.html) 需要注意下载basic与SDK(Devel)两个包，多下的SDK是用来编译oci以及pdo_oci的。我选择的是rpm的包，能免几行代码。
>如果直接wget可能会失败，因为下载会先让你登录，登陆之后，让浏览器直接下载，下载时，复制其URL后再wget即可
2. 安装两个rpm，其实zip的话也可以，我使用的rpm，devel被放在了/usr/include/oracle/,basic被放在了/usr/lib/oracle/
3. 接下来配置环境,把下面的代码放在/etc/profile内**如果版本不同一定注意将路径修改一下**
```
export ORACLE_HOME=/usr/lib/oracle/12.2/client64
export PATH=$PATH:$ORACLE_HOME/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/oracle/12.2/client64/lib
export C_INCLUDE_PATH=$C_INCLUDE_PATH:/usr/include/oracle/12.2/client64
export NLS_LANG="SIMPLIFIED CHINESE_CHINA.ZHS16GBK"
```
> 让其立即生效的命令为 source /etc/profile
### 安装OCI8
1. 两种方式，一个是pecl install oci8,另一个是下载phpsrc进行编译安装
> PHPsrc 下载地址： http://be.php.net/distributions/php-x.y.z.tar.gz
> 比如我的PHP版本为7.0.19，下载地址为http://be.php.net/distributions/php-7.0.19.tar.gz
对于OCI8，先用pecl安装，进入php的bin目录，比如我的目录是*/www/server/php/70/bin*，输入以下的命令即可
```
./pecl install oci8
```
一般来说，直接回车就可以安装，安装成功后会有以下的提示：
```
Build process completed successfully
Installing '/www/server/php/70/lib/php/extensions/no-debug-non-zts-20151012/oci8.so'
install ok: channel://pecl.php.net/oci8-2.1.8
Extension oci8 enabled in php.ini
```
如果提示oci.h找不到，则说明C_INCLUDE_PATH没有配置成功，检查一下，可以echo $C_INCLUDE_PATH看看要include的目录是否正确。
### 安装PDO_OCI
我最初想用pecl 一键解决pdo_oci的问题，但是失败了，返回以下的提示：
```
[root@localhost bin]# ./pecl install pdo_oci
WARNING: "pear/PDO_OCI" is deprecated in favor of "channel://http://www.php.net/pdo_oci/ext/pdo_oci"
pear/PDO_OCI requires PHP (version >= 5.0.3, version <= 6.0.0), installed version is 7.0.19
No valid packages found
install failed
```
> 我没有用低版本的PHP去亲自跑一遍试试,所以结果不可知，自己可以去尝试一下。

我最终还是下载了个php的源码包，解压后，进里面的ext，可以发现pdo_oci文件夹在里面躺着,于是乎，立马cd进去。

cd进去后，使用phpize命令来获取一个.configure，如果没有配PATH或者链接的话，需要加上绝对路径去
运行命令，比如：
```
/www/server/php/70/bin/phpize
```
具体的路径参考自己的配置。
有了.configure了，我们就运行一下它：
如果出现了以下的错误：
```
configure: error: Cannot find php-config. Please use --with-php-config=PATH
```
那么就老老实实地加上path，这个php-config与pecl、phpize都在一个目录下，很好找。
configureOK之后就是make & make install ，直接安装就好了
###php-fpm配置
 如果oci与pdo_oci都配置好了，phpinfo 会显示相关的模块信息。但是这样是不够的，如果你用php-fpm的话，还要在**php-fpm.conf**的www下面加上下面的几行，对于宝塔环境，php-fpm.conf 在这里：/www/server/php/70/etc
```
env[LD_LIBRARY_PATH] = /usr/lib/oracle/12.2/client64/lib
env[ORACLE_HOME] = /usr/lib/oracle/12.2/client64/lib
```
**注意，目录要和实际环境匹配**
> 注意这个ORACLE_HOME ,和刚才在profile配的是不同的！
## 最终检查
* php.ini 里面有没有加载这两个so
必须有这两个：
```
extension="oci8.so"
extension="pdo_oci.so"
```
*  通过phpinfo();确定是否有这两个模块
*  自己连接一下数据库试试。（如果出现了Check the character set is valid and that PHP has access to Oracle libraries and NLS 之类的错误，一般是env没有配好，参照上面的php-fpm配置）
