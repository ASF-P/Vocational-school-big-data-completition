# 基础环境的配置

# ssh配置

```shell
ssh-keygen -t rsa #生成密钥
#把~/.ssh下的id_rsa.pub的密钥复制，粘贴在另一台的~/.ssh/authorized_keys下

```

## jdk的配置

==如果说Hadoop是地基，那么jdk就是泥土==

解压

```shell
tar -xvf jdk-XXX.tar
```

配置环境变量

```shell
export JAVA_HOME=${JAVA_HOME}
export PATH=$PATH:$JAVA_HOME/bin
```

分发jdk到其他节点

```shell
scp -r ${JAVA_HOME} root@node2:$PWD   #r表示递归发送用于传输文件夹 
```

更新

```shell
source /etc/profile
```

测试是否安装成功

```shell
java -version
```

## Hadoop的配置

==Hadoop就像是大数据的地基一样，很多东西基于它==

同样先解压

```shell
tar -xvf hadoopX.XX.tar
```

在\${HADOOP\_HOME}/etc/hadoop/ 中修改配置文件

***hadoop-env.sh   (此文件作用是指定启动组件的用户)***

```shell
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root

export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root

export JAVA_HOME=${JAVA_HOME}
```

***core-site.xml  (此文件的作用是主要配置)***

```html
<configuration>
<property>
		<!-- hdfs在哪一个节点的端口 -->
        <name>fs.defaultFS</name>
        <value>hdfs://node1:9000</value>
</property>
<property>
		<!-- 指定缓存文件的位置 -->
        <name>hadoop.tmp.dir</name>
        <value>${HADOOP_HOME}/hadoopDatas/tempDatas</value>
</property>
<property>
		<!-- 指定静态超级用户身份简化登录hive -->
        <name>hadoop.staticuser.user</name>
        <value>root</value>
</property>
</configuration>
```

***hdfs-site.xml (此文件配置了备份、关闭文件检查权限、namenode datanode的存放地方、namenode和secondarynamenode的地址)***

```html
<configuration>
<property>
		<!-- 为了确保某一台机器损坏文件不会丢失，hdfs每一份文件都会被备份，下面是备份数量 -->
        <name>dfs.replication</name>
        <value>2</value>
</property>

<property>
		<!-- 关闭文件检查权限，在测试环境更方便 -->
        <name>dfs.permissions.enabled</name>
        <value>false</value>
</property>

<property>
        <name>dfs.namenode.name.dir</name>
        <value>${HADOOP_HOME}namenodeDatas</value>
</property>

<property>
        <name>dfs.datanode.data.dir</name>
        <value>${HADOOP_HOME}datanodeDatas</value>
</property>

<property>
        <name>dfs.namenode.http-address</name>
        <value>node1:9870</value>
</property>

<property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node1:9868</value>
</property>
</configuration>

```

***mapred-site.xml***

```html
<configuration>
<property>
		<!-- 告诉mapreduce在yarn上执行 -->
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
</property>
<property>
        <!-- 告诉mapreduce 软件的地址 -->
		<name>mapreduce.application.classpath</name>
        <value>/root/software/hadoop-3.2.1/share/hadoop/mapreduce/*:/root/software/hadoop-3.2.1/share/hadoop/mapreduce/lib/*</value>
</property>
<property>
		<!-- 历史服务器的地址 -->
        <name>mapreduce.jobhistory.address</name>
        <value>node1:10020</value>
</property>
<property>
		<!-- 历史服务器的web地址 -->
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>node1:19888</value>
</property>
</configuration>
```

***yarn-site.xml***

```html
<configuration>
<property>
		<!-- 说明白yarn的resourcemanager在哪执行 -->
        <name>yarn.resourcemanager.hostname</name>
        <value>ndoe1</value>
</property>
<property>
		<!-- 不知道 自己看着办..（doge -->
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
</property>
<property>
		<!-- 关闭虚拟磁盘限制 -->
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
</property>
<property>
		<!-- 关闭物理磁盘限制  -->
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
</property>
</configuration>
```

***workers***

```shell
#根据实际写hosts配置的主机名即可
node1
node2
node3
```

接下来可以scp给其他节点

```sql
scp -r ${HADOOP_HOME} root@node2:$PWD
```

配置Hadoop的环境变量

```shell
vim /etc/profile

export HADOOP_HOME=${HADOOP_HOME}
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

必须初始化namenode奥

```shell
hdfs namenode -format
```

原神，\_\_\_\_！

    start-all.sh

## MySQL

==为了下一步hive做铺垫==

老样子第一步解压

```shell
tar -xvf mysql-XXXXX.tar.gz -C mysql #可以用-C 指定解压到哪
```

可能会存在CentOS7自带的mariadb 需要先删除！

```shell
rpm -qa | grep mariadb #-qa 查询 所有 grep 搜索

#如果存在就删除
rpm -e --nodeps mariadbXXXXX-XXXX #-e 删除 --nodeps 不检查依赖
```

安装mysql

```shell
rpm -ivh mysql-community-common-5.7.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-5.7.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-devel-5.7.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-compat-5.7.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-5.7.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-server-5.7.25-1.el7.x86_64.rpm
```

启停MySQL

```shell
systemctl start mysqld #启动
systemctl stop mysqld #关闭
```

先去编辑/etc/my.cnf使我们可以免密登录

```shell
vim /etc/my.cnf

skip-grant-tables  #跳过密码
```

启动MySQL，设置一个密码(123456)

```sql
flush privileges;  #刷新权限

set password for root@localhost = '123456';  #设置密码

exit; #退出
```

再次编辑/etc/my.cnf，注释掉免密登录语句

```shell
vim /etc/my.cnf

#skip-grant-tables
```

再次进入MySQL

```shell
#注意 注意次需要密码
mysql -uroot -p123456
#或者
mysql -u root -p #回车会让你输入密码 当然是不可见的
123456
```

现在需要让hive可以连接MySQL

```sql
#刷新权限
flush privileges;

grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;

#退出
exit;
```

## Hive

==hive和SQL简直就是一家人！语法十分接近。但是这里只是安装配置。详细见另一篇：\[空位]==

老老老样子，解压

```shell
tar -xvf ${HIVE_HOME}
```

配置hive

```shell
cd ${HIVE_HOME}/conf

mv hive-env.sh.template hive-env.sh  #  把template文件重命名

vim hive-env.sh

#  添加或修改原文至以下
HADOOP_HOME=${HADOOP_HOME}hadoop-3.2.1
export HIVE_CONF_DIR=${HIVE_HOME}conf
export HIVE_AUX_JARS_PATH=${HIVE_HOME}lib
```

配置hive-site.xml

```shell
#并不自带 需要自己创建 vim 会自动创建并编辑不存在的文件
vim hive-site.xml
```

配置如下(不需要记忆，根据表格自己填就好，此处只做展示)：

需要解决lib下guava版本低的问题（复制Hadoop的过来）

```html
<configuration>  

<property>

<name>javax.jdo.option.ConnectionURL</name>

<!-- 注意这里 &amp;是需要带上的 -->
<value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>

<description>JDBC connect string for a JDBC metastore</description>

</property>

<property>

<name>javax.jdo.option.ConnectionDriverName</name>

<value>com.mysql.jdbc.Driver</value>

<description>Driver class name for a JDBC metastore</description>

</property>

<property>

<name>javax.jdo.option.ConnectionUserName</name>

<value>root</value>

<description>username to use against metastore database</description>

</property>

<property>

<name>javax.jdo.option.ConnectionPassword</name>

<value>123456</value>

<description>password to use against metastore database</description>

</property>

</configuration>

```

然后配置环境变量

```shell
vim /etc/profile

# 添加
export HIVE_HOME=${HIVE_HOME}
export PATH=$PATH:$HIVE_HOME/bin

# 更新
source /etc/profile
```

在进入\${HIVE\_HOME}/lib做点什么

    rm -rf log4j-slf4j-impl-2.10.0.jar #  因为Hadoop也带了一份 不删除会多重绑定 虽然不影响但是难看
    rm -rf guava-19.0.jar #hive自带的太老了
    cp ${HADOOP_HOME}/share/hadoop/common/lib/guava-27.0-jre.jar $PWD #此时你需要在hive的conf目录 否则把$PWD换成绝对路径

现在我们可以初始化hive了

```shell
schematool -initSchema -dbType mysql -verbose
```

启动！

```shell
hive
```

## Flume

==flume 是 用于传输用的，顾名思义，像"管子"一样让数据实时的从一个地方流向留一个地方==

老老老老样子，解压

```shell
tar -xvf apache-flume-xxxx.tar.gz
```

接下来配置环境变量

```shell
export FLUME_HOME=${FLUME_HOME}
export PATH=$PATH:$FLUME_HOME/bin

source /etc/profile  #更新
```

flume的配置

```shell
#需要在${FLUME_HOME}/conf/目录下重命名flume-env.sh.template
cd ${FLUME_HOME}/conf

mv flume-env.sh.template flume-env.sh

vim flume-env.sh

#编辑或添加JAVA_HOME
export JAVA_HOME=${JAVA_HOME}
```

(对于例题来说是拓展)启动flume，在\${flume\_home}/下新建一个文件夹，名字随意，新建文件名字，随意后缀名是.conf，键入以下内容

```shell
a1.sources = r1
a1.sinks = k1
a1.channels = c1

a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
# bind 设置成localhost 即只能自己监听自己，必要的话设置0.0.0.0 让别人可以连接
a1.sources.r1.port = 44444

a1.sinks.k1.type = logger

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transectionCapacity = 100

a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

安装nc  (如果没有)

```shell
wget https://mirrors.aliyun.com/centos-vault/7.9.2009/os/x86_64/Packages/nmap-ncat-6.40-19.el7.x86_64.rpm

rpm -ivh nmap-ncat-6.40-19.el7.x86_64.rpm

nc
```

跑flume

```shell
flume-ng agent -c /root/software/apache-flume-1.11.0-bin/conf/ -f /root/software/apache-flume-1.11.0-bin/example/example.conf -n a1

#绝对路径根据自己情况(更安全
```

在另一个终端页面输入 "nc node1 44444" 即可连接成功。实际可能不用nc，但是配置的方法大同小异

## Zookeeper

==这个是为字kafka能够在集群上使用。有趣的是kafka自带的没有这个更合适==

```shell
# 解压
tar -xvf apache-zookeeper-3.8.4-bin.tar.gz
# 进入${zookeeper}/conf目录复制一份样品配置文件
cp zoo_sample.cfg zoo.cfg
# 修改一些配置
vim zoo.cfg
```

```shell
# 添加以下代码 server.X 的 X 指的是myid里的编号nodeX换成实际的hosts文件里的配置
server.0 = node1:2888:3888
server.1 = node2:2888:3888
server.2 = node3:2888:3888

# scp 分发给其他节点
scp -r /root/software/apache-zookeeper-3.8.4-bin root@node2:/root/software/
scp -r /root/software/apache-zookeeper-3.8.4-bin root@node3/root/software/

# 这里要留意下dataDir的位置等一下要配置myid 默认是/tmp/zookeeper
# 我们打开到对于dataDir的位置 需要自己创建一个zookeeper文件夹
cd /tmp && mkdir zookeeper
cd zookeeper
# > 把 echo打印的 "X"(这是myid配置的数字，应和zoo.cfg里配置的对应起来)覆盖重定向到myid文件
# 说人话就是创建文件 "myid" 写入一个数字(即 X) 
echo X > myid
```

随后配置以下环境变量

```shell
vim /etc/profile

# 添加以下内容
export ZOOKEEPER_HOME=/root/software/apache-zookeeper-3.8.4-bin
export PATH=$PATH:$ZOOKEEPER_HOME/bin

# shift + zz 退出 或 :wq 退出

#更新资源
source /etc/profile 
```

然后在每一台机器上执行 zkServer.sh start看到started即可，如果不放心可以运行jps，会有一个"QuorumPeerMain"的进程即为成功

## Kafka

解压略

在conf/文件夹里编辑server.properties文件

```
vim server.properties

# 其中 broken.id 就是 zookeeper 配置对于每台机器配置的 myid
# 还需要改  kafka.connect 
# 根据实际更改 zookeeper.connect=node1:2181,node2:2181,node3:2181/kafka

```

分发到其他节点后在后台运行

```shell
nohup kafka-server-start.sh /root/software/kafka_2.13-3.7.2/config/server.properties > /root/software/kafka.log &
# nohup ... & 在后台运行 把日志写在其他地方避免占用shell
```

```shell
#创建一个topic
kafka-topics.sh  --create --topic [name] --bootstrap-server localhost:9092

#展示topic
kafka-topics --list --bootstrap-server localhost:9092

#启动生产者
kafka-console-producer.sh --topic [name] --bootstrap-server localhost:9092

#启动消费者
kafka-console-consumer.sh --topic [name] --bootstrap-server localhost:9092

kafka-topics.sh --create \
  --topic user-events \
  --bootstrap-server node1:9092 \
  --partitions 3 \
  --replication-factor 1
```

Mapreduce作业编写

map阶段

```java
package SortWordCountTest;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class SortMapper extends Mapper<LongWritable, Text, DataSet, Text> {
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, DataSet, Text>.Context context) throws IOException, InterruptedException {
        DataSet dataSet = new DataSet();
        String[] words = value.toString().split(" ");

        // 实例化类 调用set方法
        dataSet.setWord(words[0]);
        dataSet.setNum(Integer.parseInt(words[1]));

        // 传输到下一个流程
        context.write(dataSet, value);
    }
}

```

reduce阶段

```java
package SortWordCountTest;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class SortReducer extends Reducer<DataSet, Text, DataSet, NullWritable> {
    @Override
    protected void reduce(DataSet key, Iterable<Text> values, Reducer<DataSet, Text, DataSet, NullWritable>.Context context) throws IOException, InterruptedException {
        // 这一步只需要留着key就可以
        context.write(key,NullWritable.get());
    }
}
```

自定义数据类型

```java
package SortWordCountTest;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

// 实现 WritableComparable 接口
public class DataSet implements WritableComparable<DataSet> {
    String word;
    int num;

    public String getWord() {
        return word;
    }

    public void setWord(String word) {
        this.word = word;
    }

    public int getNum() {
        return num;
    }

    public void setNum(int num) {
        this.num = num;
    }

    @Override
    public int compareTo(DataSet o) {
        // 当String 为0的时候(两个字符串相等)，比较后面的数字
        int result = word.compareTo(o.word);
        if (result == 0) {
            return this.num - o.num;
        }
        return result;
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException {
        // 序列化 （格式）
        dataOutput.writeUTF(word);
        dataOutput.writeInt(num);
    }

    @Override
    public void readFields(DataInput dataInput) throws IOException {
        // 反序列化
        this.word = dataInput.readUTF();
        this.num = dataInput.readInt();
    }

    @Override
    public String toString() {
        return word + "\t" + num;
    }
}

```

JobMain

```java
package SortWordCountTest;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {
    @Override
    public int run(String[] strings) throws Exception {
        Configuration conf = getConf();
        conf.setInt("ipc.maximum.data.length", 67108864);
        conf.setInt("ipc.maximum.response.length", 67108864);
        FileSystem fs = FileSystem.get(conf);

        Job job = Job.getInstance(conf, "Job Main");
        job.setJarByClass(JobMain.class);


        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path(strings[0]));


        job.setMapperClass(SortMapper.class);
        job.setMapOutputKeyClass(DataSet.class);
        job.setMapOutputValueClass(Text.class);

        job.setReducerClass(SortReducer.class);
        job.setOutputKeyClass(DataSet.class);
        job.setOutputValueClass(NullWritable.class);

        Path path = new Path(strings[1]);

        if (fs.exists(path)) {
            fs.delete(path, true);
        }

        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job, path);

        boolean result = job.waitForCompletion(true);



        return result ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        conf.setInt("ipc.maximum.data.length", 67108864);
        conf.setInt("ipc.maximum.response.length", 67108864);
        ToolRunner.run(conf, new JobMain(), args);
    }
}

```

可能出现分区，以下提供一个分区样例

map

```java
package WordCount;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class MapperTask extends Mapper<LongWritable, Text, Text, LongWritable> {
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, LongWritable>.Context context) throws IOException, InterruptedException {
        String[] data = value.toString().split(" ");
        LongWritable val = new LongWritable(1);
        for (String s : data) {
            context.write(new Text(s), val);
        }
    }
}

```

reduce

```java
package WordCount;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;


public class ReducerTask extends Reducer<Text, LongWritable, Text, LongWritable> {
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Reducer<Text, LongWritable, Text, LongWritable>.Context context) throws IOException, InterruptedException {
        long count = 0;

        for (LongWritable val : values) {
            count += val.get();
        }

        context.write(key, new LongWritable(count));
    }
}

```

partition

```java
package WordCount;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;



public class PartitionerOwn extends Partitioner<Text, LongWritable> {
    @Override
    public int getPartition(Text text, LongWritable longWritable, int i) {
        if (text.toString().length() >= 5) {
            return 0;
        } else {
            return 1;
        }
    }
}

```

JobMain

```java
package WordCount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;


import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {
    @Override
    public int run(String[] strings) throws Exception {
        Configuration conf = getConf();
        FileSystem fs = FileSystem.get(conf);
        conf.setInt("ipc.maximum.data.length", 67108864);
        conf.setInt("ipc.maximum.response.length", 67108864);

        Job job = Job.getInstance(conf);
        job.setJarByClass(JobMain.class);


        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path(strings[0]));

        job.setMapperClass(MapperTask.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        job.setPartitionerClass(PartitionerOwn.class);

        job.setReducerClass(ReducerTask.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);
        job.setNumReduceTasks(2);

        Path path = new Path(strings[1]);
        if (fs.exists(path)) {
            fs.delete(path, true);
        }
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job, path);

        boolean result = job.waitForCompletion(true);



        return result ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        conf.setInt("ipc.maximum.data.length", 67108864);
        conf.setInt("ipc.maximum.response.length", 67108864);
        ToolRunner.run(conf, new JobMain(), args);
    }
}

```

# 时间同步

使用chrony操作

```
vim /etc/chrony.conf
# 编辑配置文件
# 把原来的server注释 添加以下代码
server ntp.aliyun.com

# 在 allow的地方添加自己网段的网络地址, 允许其他的使用 例如
allow 192.168.88.0/24

# 关闭防火墙
systemctl disable firewalld

# 重启 chrony服务
systemctl restart chronyd

# 查看时间同步是否成功
chronyc sources

chronyc tracking

```

其他机器编辑同样的文件，把server注释，添加一个node1为地址的server

# 数据库相关

    # 新建一个用户
    create user 'username'@'host' identified by 'password';

    # 赋予权限
    grant all *.* to  'username'@'host';

# Hive

```sql
# 创建一个外部表 使用external关键字
create table external stu(
	name varchar(5),
	age int
) row format delimited fields terminated by ",";

# 使用其他存储方式
create table external stu(
	name varchar(5),
	age int
) row format delimited fields terminated by ","
stored as [TYPE];

# 有 textfile, squencefile, rcfile, orcfile等

# 需要先新建一个临时表 注意char不能分区用string
# 新建一个分区表
create external table comp(
	name string,
	age int,
	height int
) partitioned by (sex string) row format delimited fields terminated by ",";

# 建一个临时表
create table temnp(
	name string,
	age int,
	sex string,
	height int
)row format delimited fields terminated by ",";

# 动态分区 注意分区要在查询得最后一个
insert overwrite table comp partition(sex) select name, age, height, sex from temp;
```

# Flume

必须要有 sources sinks channels

```
# in this case called 'agent'
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1 f2
a1.sources.r1.filegroups.f1 = /opt/data/flume/.*
a1.sources.r1.filegroups.f2 = /opt/data/flume2/.*

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://node1:9000/flume_toHDFS/%Y%m%d/%H
a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 131000
a1.sinks.k1.hdfs.rollCount = 10
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.useLocalTimeStamp = true

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1

```

# Hbase

首先保存的东西一定要在长期保存的目录下，不然重启会清除。

如果一直出现Hmaster初始化的问题，需要清除hdfs、zookeeper的数据

hdfs dfs -rm /hbase

zkCli.sh

deleteall /hbase

```
# 创建一个表
create "students", "basic_info", "score_info"
students 是表名
basic_info 是列族1
score_info 是列族2

# 插入数据
put "students", "001", "basic_info:name", "Joan"
put "studnets", "001", "basic_info:age", "10"

# 查询数据
get "students", "001", "basic_info"

# 扫描数据 (一行)
scan "students", {LIMIT => 1}

# 查询特定列
scan "students", {COLUMNS => "basic_info:name"}


```

\########高级操作###########

    # 根据行键过滤
    scan "scores", {FILTER => "RowFilter(>=, 'binary:1002')"}
    # scores 是表名
    # FILTER 是过滤的表示
    # RowFilter 表示按照行键来
    # >= 比较方式
    # binary:1002 与。。。比较

    scan "scores", {FILTER => "RowFilter(>=, 'binary:1002') AND RowFilter(<=, 'binary:1004')"}

\###########值过滤器############

    scan "scores", {FILTER => "valueFilter(>=, 'binary:40')"}

\########特定列过滤器############

    scan "scores", {FILTER => "SingleColumnValueFilter('scores', 'math', >, 'binary:80')"}
    # 查找 scores 列族, math 列， 大于80的 信息

\########行键前缀包含##############

    scan "scores", {FILTER => "PrefixFilter('100')"}
    # 行键包含 "100" 的信息

\########只显示某列信息#################

    scan "scores", {FILTER => "XXXX", COLUMNS => ["XX", "XX"]}

    scan "school:students", {FILTER => "SingleColumnValueFilter('scores', 'math', >, 'binary:60')", COLUMNS => ['info:name']}

