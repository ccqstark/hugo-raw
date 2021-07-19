---
title: "[Elastic]使用logstash同步MySQL数据"
date: 2021-01-28T21:26:00+08:00
draft: true
image: logstash-mysql.jpg
slug: "logstash_mysql"
tags:
    - ElasticSearch
categories:
    - 环境搭建
--- 


## 安装logstash

在实际项目中使用es进行搜索，我们就要把mysql数据库中的数据同步到es索引库中。进行这项过程的工具很多，比如go-mysql-elasticsearch，canal等等，当然也可以使用ELK组合中的`logsatsh` 来完成。这里同样用docker来部署logstash容器。

### 拉取镜像

```bash
docker pull logstash:7.10.1
```

### 启动容器

启动后进入容器内，修改jvm启动的内存设置，地址为`/usr/share/logstash/config/jvm.options`

```bash
# 修改jvm内存分配
vi jvm.options

# 修改下面的参数，单位可以为g和m
-Xms256m
-Xmx256m
```

修改后重启容器即可

### 下载插件与依赖包

```bash
docker exec -it logstash bash
```

安装`logstash-input-jdbc`插件

```bash
bin/logstash-plugin install logstash-input-jdbc
```

如果出现以下ERROR，说明logstash里本身已经包含有这个插件了，就无需安装。7.10.1的版本是已经自带了。

```bash
ERROR: Installation aborted, plugin 'logstash-input-jdbc' is already provided by 'logstash-integration-jdbc'
```

下载mysql-connector-java，也就是jdbc驱动

MySQL官方下载地址：[https://downloads.mysql.com/archives/c-j/](https://downloads.mysql.com/archives/c-j/)

下载对应版本后本地解压，上传到服务器，然后用`docker cp`命令复制到logstash容器中

只需要其中的jar包即可

```bash
# 把文件复制到容器内
docker cp [jar包路径] logstash:[容器内路径]
```

在`/usr/share/logstash`目录下新建`mysql/`目录，把jar包复制到这里

### 同步配置文件

在刚刚的mysql目录下新建`jdbc.conf` 文件，来配置同步操作

- **单表同步**

```bash
input {
    jdbc {
			# jar包的绝对路径
			jdbc_driver_library => "/usr/share/logstash/mysql/mysql-connector-java-5.1.48.jar"
    	jdbc_driver_class => "com.mysql.jdbc.Driver"
   	  # 数据库连接信息
			jdbc_connection_string => "jdbc:mysql://[ip]:3306/[库名]?characterEncoding=UTF-8&autoReconnect=true"
    	jdbc_user => "[mysql用户]"
    	jdbc_password => "[密码]"
			# cron的定时执行语法一样，默认每分钟同步一次
    	schedule => "* * * * *"
			# 执行的sql语句语法，这里是通过将主键大于最后一次同步所记录的值来实现增量同步的
    	statement => "SELECT * FROM activity WHERE activity_id > :sql_last_value order by activity_id asc"
			use_column_value => true
			# 用来作为增量同步的判断字段，最好为表的主键
			tracking_column => "activity_id"
			# 是否记录上次执行结果，true表示会将上次执行结果的tracking_column字段的值保存到last_run_metadata_path指定的文件中；
			record_last_run => true
			# record_last_run上次数据存放位置
			last_run_metadata_path => "/usr/share/logstash/mysql/last_id.txt"
			# 是否清除last_run_metadata_path的记录，需要增量同步时此字段必须为false
			clean_run => false
    }
}

output{
	elasticsearch{
		hosts => ["[ip]:9200"]
		index => "[索引名]"
		# 数据唯一索引（建议使用数据库KeyID）
		document_id => "%{activity_id}"
	}
  stdout { 
		codec => rubydebug 
	}
}
```

- **多表同步**

多表配置和单表配置的区别在于input模块的jdbc模块有几个type，output模块就需对应有几个type

```bash
input {
	stdin {}
	jdbc {
		 # 多表同步时，表类型区分，建议命名为“库名_表名”，每个jdbc模块需对应一个type；
		type => "TestDB_DetailTab"
		
		 # 其他配置此处省略，参考单表配置
		 # ...
		 # ...
		 # record_last_run上次数据存放位置；
		last_run_metadata_path => "mysql\last_id.txt"
		 # 是否清除last_run_metadata_path的记录，需要增量同步时此字段必须为false；
		clean_run => false
		 #
		 # 同步频率(分 时 天 月 年)，默认每分钟同步一次；
		schedule => "* * * * *"
	}
	jdbc {
		 # 多表同步时，表类型区分，建议命名为“库名_表名”，每个jdbc模块需对应一个type；
		type => "TestDB_Tab2"
		# 多表同步时，last_run_metadata_path配置的路径应不一致，避免有影响；
		 # 其他配置此处省略
		 # ...
		 # ...
	}
}

filter {
	json {
		source => "message"
		remove_field => ["message"]
	}
}

output {
	# output模块的type需和jdbc模块的type一致
	if [type] == "TestDB_DetailTab" {
		elasticsearch {
			 # host => "192.168.1.1"
			 # port => "9200"
			 # 配置ES集群地址
			hosts => ["192.168.1.1:9200", "192.168.1.2:9200", "192.168.1.3:9200"]
			 # 索引名字，必须小写
			index => "detailtab1"
			 # 数据唯一索引（建议使用数据库KeyID）
			document_id => "%{KeyId}"
		}
	}
	if [type] == "TestDB_Tab2" {
		elasticsearch {
			# host => "192.168.1.1"
			# port => "9200"
			# 配置ES集群地址
			hosts => ["192.168.1.1:9200", "192.168.1.2:9200", "192.168.1.3:9200"]
			# 索引名字，必须小写
			index => "detailtab2"
			# 数据唯一索引（建议使用数据库KeyID）
			document_id => "%{KeyId}"
		}
	}
	stdout {
		codec => json_lines
	}
}
```

为了统一，把数据也放这个目录下，在mysql目录下再新建目录`data`

```bash
mkdir data
```

### 启动logstash同步

cd回到`/usr/share/logstash`目录，启动同步

```bash
./bin/logstash -f mysql/jdbc.conf --path.data=/usr/share/logstash/mysql/data/
```

注意，要保证给elasticsearch的分配的内存足够大才行，测试用的256m是不够的，会导致es容器退出，至少给个1g

`—-path.data`参数用来设置同步数据存放的位置

启动之后控制台会打印出大概以下信息

```bash
Using bundled JDK: /usr/share/logstash/jdk
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.jruby.ext.openssl.SecurityHelper (file:/tmp/jruby-423/jruby7667758569951782495jopenssl.jar) to field java.security.MessageDigest.provider
WARNING: Please consider reporting this to the maintainers of org.jruby.ext.openssl.SecurityHelper
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
```

开始同步会打印出从数据库读取的，插入到elasticsearch的信息

实际使用中用`nohup` 来保持后台运行

### 缺点

logstash的缺点很明显，就是只能同步mysql中新增的数据，对于更改的、删除的就无能为力了。这其实也很好理解，ELK其实本来就是用来收集与分析日志的，而同步增加的数据已经足够了。

我觉得es与mysql最好的同步工具其实是阿里的开源的`canal`

[alibaba/canal](https://github.com/alibaba/canal)