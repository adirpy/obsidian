
## Linux下用Oracle Instant Client 11.0.2 的问题

在CentOS 7上进行环境搭建，出现了工作空间无法打开的原因。首先检查了工作空间，Oracle的TNSNAME，License，都没有问题。
然后检查超图SMO相关依赖：
`ldd libWrapj.so`
发现缺少了本地包，在网上下载之后，发现还是无法打开。然后新建本地工作空间放上去尝试打开，发现可以打开本地工作空间，但无法打开Oracle工作空间。说明不是SMO的问题，咨询超图得知，还需要检查Oracle相关的依赖。
`ldd libSuEngineOracle.so`
发现果然是缺少了libclntsh.so.10.1这个lib。
在Oracle的Instant Client的目录下，可以发现libclntsh.so.11.1，可见这只是一个文件版本的问题，直接
`cp libclntsh.so.11.1 libclntsh.so.10.1`
然后再次打开oracle工作空间，已经可以。

## Suse下依赖问题

```bash
libSuEngineOracle.sdx
libSuEngineUDB.sdx
libSuEngineImagePlugin.sdx
```

只要这3个SDX

## Supermap 性能优化

*   全部线和面建索引（R-树索引目前来看，是性能最优的）

*   对点数据集创建SMX, SMY的联合索引: CREATE INDEX SM\_IDX\_FXBMA ON FTTXBMA\_POLEP\_2 (SMX, SMY)

*   对精度要求不高的线和面进行重采样

*   可编辑线数据，也可以创建R树索引+给smkey创建数据库索引

*   License Server配置

*   前提 开通linux客户机和许可服务器之间的1947端口（TCP协议），可通过如下方法验证端口是否开通：

    *   在linux客户机上运行 ping 许可服务器IP
    *   在linux客户机上运行 telnet 许可服务器IP 1947

*   步骤

    *   停止linux客户机许可服务：进入etc/init.d 目录，运行 sh aksusbd stop
    *   修改linux客户机许可配置到指定许可服务器：把 /etc/hasplm下的haspim.ini文件打开，修改\[REMOTE]下的serveraddr为许可服务器ip（如没有该参数，增加该参数）。【注】如果在 /etc/hasplm下没有haspim.ini文件，可将附件文件修改参数后，拷入到这个目录下；
    *   重新启动linux客户机许可服务：进入etc/init.d 目录，运行 sh aksusbd start

## Supermap 工作空间6R和7C混用之后的处理方法

```sql
alter table smdatasourceinfo drop (smprojectinfo)alter table smdatasourceinfo add (smprojectinfo blob)
alter table smregister drop (smprojectinfo)alter table smregister add (smprojectinfo blob)
alter table smimgregister drop (smprojectinfo)alter table smimgregister add (smprojectinfo blob)
```

置空后投影需要重新设置

## WGS1984 坐标系下，经纬度换算成长度

*   经度1度=85.39km1分=1.42km1秒=23.6m
*   纬度1度=111km1分=1.85km1秒=30.9m

## SuperMap 安装环境注意事项

*   点图层
    *   SMX, SMY的联合字段索引
    *   RES\_ID的字段索引
    *   如果有文本标签，可以考虑把文本标签字段也建索引
    *   如果有其余字段相关的专题图（比如单值），对应字段建索引
*   线/面图层
    *   SmSdriW, SmSdriN, SmSdriE, SmSdriS 建联合字段索引
    *   RES\_ID的字段索引
    *   如果有文本标签，可以考虑把文本标签字段也建索引
    *   考虑建R-Tree索引
    *   如果有其余字段相关的专题图（比如单值），对应字段建索引
*   地图记得打开Alpha通道
*   集群每个进程 预留2线程，6g左右内存

## 超图遇到问题汇总

*   2015-02-27日 泰国True现场服务器，发现服务器license一直配置有问题，报错中间首先替换了com.supermap.license.jar，但是替换完之后，只是配置license的时候没有那些错误，但是还是一样的没有正确的license写入。后来将lic文件放入/opt/SuperMap/License目录之后，发现可以了，估计是lic文件的读取权限可能有问题。

*   遇到大数据量数据无法导入supermap oracle数据源时在服务器\$ORACLE\_HOME/network/admin下，增加一个sqlnet.ora，内容是SQLNET.SEND\_TIMEOUT=600

*   copy版 iDesktop，下拉列表没有tns原因注册表HKEY\_LOCAL\_MACHINE\SOFTWARE\Oracle是否有ORACLE\_HOME变量

*   license处理方式将试用许可（\*.lic7c）放置到指定目录，如果没有该目录，手动创建，并保证使用的账户对该目录有读写权限。

    *   Windows放置位置：操作系统所在盘符: `\Program Files\Common Files\SuperMap\License`
    *   Linux放置位置：/opt/SuperMap/License/opt 目录普通账号通常没有读写权限，需要授权对该目录的读写权限。

*   service的管理界面报错，错误日志：

    ```bash
    2021-2-5 08:36:23 - WARN - Delegate RememberMeManager instance of type [org.apache.shiro.web.mgt.CookieRememberMeManager] threw an exception during getRememberedPrincipals().
    org.apache.shiro.crypto.CryptoException: Unable to execute 'doFinal' with cipher instance [javax.crypto.Cipher@4a6b8a05].
     at org.apache.shiro.crypto.JcaCipherService.crypt(JcaCipherService.java:462)
     at org.apache.shiro.crypto.JcaCipherService.crypt(JcaCipherService.java:445)
     at org.apache.shiro.crypto.JcaCipherService.decrypt(JcaCipherService.java:390)
     at org.apache.shiro.crypto.JcaCipherService.decrypt(JcaCipherService.java:382)
     at org.apache.shiro.mgt.AbstractRememberMeManager.decrypt(AbstractRememberMeManager.java:482)
    ```

    在WEB-INF/shiro.ini文件添加这个

    ```bash
    invalidRequest.blockNonAscii = false
    ```

    ## PostGIS下修改超图默认的主键和空间字段

    1.  修改SMRegister表

        ```SQL
        select smdatasetname ,smidcol , smgeocol  from smregister s ;
        ```

    2.

