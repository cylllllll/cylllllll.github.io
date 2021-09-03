---
title: Kylin 安装配置
categories: Big Data
date: 2018-11-03 22:00:12
---
# Kylin 安装配置
1. 解压缩到/usr/local目录
`tar -zxvf Kyligence.tar.gz -C/usr/local/`
2. 配置环境变量KYLIN_HOME     在/etc/profile中添加：           `export KYLIN_HOME=/usr/local/Kyligence`
3. 在HDFS上创建/kylin用户目录，并赋予全权限
    ```
    su hdfs 
    hdfs dfs -mkdir /kylin
    hdfs dfs -chown root /kylin
    hdfs dfs -mkdir /user/root
    hdfs dfs -chown root /user/root
    ```
* root可以替换为你的用户
4. 如果机器为配置SPARK_HOME环境，就指向KYLIN的spark目录 在/etc/profile中添加： `export SPARK_HOME=$KYLIN_HOME/spark`
5. 启动前检查
`$KYLIN_HOME/bin/check_env.sh`
6. 启动KYLIN
   check-env.sh通过，修改`$KYLIN_HOME/tomcat/conf/server.xml`中`<Connector port="7070" 为 Connector port="15022"` 端口号最好在15020-15039之间，注意端口使用占用,最后启动KYLIN。
    `./kylin.sh start`


# 报错信息
* The executor's instances: 4 configured in kylin.properties shouldn't beyond maximum: 3 in theory        
解决：修改'$KYLIN_HOME/conf/kylin.properties'中 kap.storage.columnar.spark-conf.spark.executor.instances=3
*  Couldn't find hive configuration directory. Please set HIVE_CONF to the path which contains hive-site.xml
解决：添加环境变量

    ```
    export HIVE_CONF=/usr/hdp/2.6.0.3-8/hive/conf
    export HCAT_HOME=/usr/hdp/2.6.0.3-8/hive-hcatalog
    export HIVE_LIB=/usr/hdp/2.6.0.3-8/hive/lib
    ```

* Couldn't find hive.metastore.uris in hive-site.xml. hive.metastore.uris specifies Thrift URI of hive metastore . Please export HIVE_METASTORE_URI with hive.metastore.uris before starting kylin , for example: export HIVE_METASTORE_URI=thrift://sandbox.hortonworks.com:9083
在hive.site.xml添加

    ```
    </property>
    <property>
    <name>hive.metastore.uris</name>
      <value>thrift:你的hostname:9083</value>
    </property>
    ```
* 如图
![屏幕快照 2018-11-01 下午17.20.33 下午-w897](https://lh3.googleusercontent.com/-aM_R-143j7Y/W91HODVoEbI/AAAAAAAAAE8/vkuVOZAWK3AntXHZ69Hie8Wh7UnG6zVgACHMYCw/I/%25255BUNSET%25255D)

    用当前用户执行：`hive` 检查当前用户是否具有权限。若无权限可以尝试在HDFS配置Custom core-site中增加：
![屏幕快照 2018-11-03 下午17.26.55 下午](https://lh3.googleusercontent.com/-zx8Gloq0vpk/W91rG02SJdI/AAAAAAAAAFY/_bFmxAQzT_cpIoVcGACpcHDFAsFaiTK5ACHMYCw/I/%255BUNSET%255D)




