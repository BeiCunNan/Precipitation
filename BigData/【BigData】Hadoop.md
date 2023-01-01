# 【BigData】Hadoop

<font color=#ff9f43>Importance</font>

<font color=#FD7272>Importance</font>

<font color=#00d2d3>Importance</font>



## 一 、Overview

###  2.1 Big Data

四个特点：4V

编程

## 二 HDFS

### 2.1 Overview

#### 2.1.1 WWH

随着数据量的增多，需要将数据存储到不同的OS中，需要使用到<font color=#ff9f43>分布式文件管理系统</font>，HDFS(Hadoop Distributed File System)就是一种分布式文件管理系统，用来<font color=#ff9f43>存储、管理</font>文件。适合一次性读入，多次读出的场景



#### 2.1.2 Advantages

- 高容错性：数据自动保存多个副本，丢掉一个副本后可自动恢复
- 适合处理大数据：能够处理PB级和百万规模以上的文件数量
- 成本低：构建在廉价机器上



#### 2.1.3 Disadvantage

- 不适合低时延数据访问
- 无法高效的对大量小文件进行存储：无论大文件还是小文件都需要NameNode来存储目录和块信息，对于小文件来说资源其实是很浪费的
- 不支持并发写入、文件随机修改：仅支持数据append追加



#### 2.1.4 Compose

**NameNode(NN)**：存储文件的<font color=#ff9f43>元数据</font>，如文件名、文件目录结构、文件属性（生成时间、副本数、文件权限）以及每个文件的<font color=#ff9f43>块列表</font>和块所在的DataNode等

**DataNode(DN)**：在本地文件系统存储文件块数据以及块数据的校验和，执行数据块的读/写操作

**Secondary NameNode(2NN)**：小秘书，每隔一段时间对NameNode元数据进行备份（不是热备份），若NN挂掉了，需要一段时候来替换NN

NN下达指令，DN进行实际操作

![image-20221226221619173](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221226221619173.png)

#### 2.1.5 File block

Hadoop2.X和Hadoop3.X都是128Mb，Hadoop1.X是64Mb

为什么块的大小既不能设置太小也不能设置太大呢？

- 若设置太小，增加了寻址时间
- 若设置太大，计算机传输速度和处理速度过慢
- <font color=#ff9f43>大小取决于你计算机本身的传输速率</font>



### 2.2 Shell

#### 2.2.1 Basic grammer

hdfs fs -[operation]

hdfs hds -[operation]



#### 2.2.2 Common commands

| Create File |                                                           |
| ----------- | --------------------------------------------------------- |
|             | [root@hadoop102 hadoop-3.1.3]# `hadoop fs -mkdir /sanguo` |



| Upload File                                                  |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 方式一： -move本地<font color=#ff9f43>剪切</font>粘贴到HDFS  | [root@hadoop104 hadoop-3.1.3]# `hadoop fs -moveFromLocal ./shuguo.txt /sanguo` |
| 方式二：本地<font color=#ff9f43>拷贝</font>到HDFS            | [root@hadoop104 hadoop-3.1.3]# `hadoop fs -copyFromLocal weiguo.txt /sanguo` |
| 方式三：等同于<font color=#ff9f43>拷贝</font>，<font color=#ff9f43>生产环境</font>更习惯用这个，因为短小精悍 | [root@hadoop104 hadoop-3.1.3]# `hadoop fs -put ./wuguo.txt /sanguo` |
| 方式四：<font color=#ff9f43>追加</font>                      | [root@hadoop104 hadoop-3.1.3]# `hadoop fs -appendToFile liubei.txt /sanguo/shuguo.txt` |



| Download File                                                |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 方式一：从HDFS拷贝到本地                                     | [root@hadoop104 hadoop-3.1.3]# `hadoop fs -copyToLocal /sanguo/shuguo.txt ./` |
| 方式二：等同于<font color=#ff9f43>拷贝</font>，<font color=#ff9f43>生产环境</font>更习惯用这个，短小精悍，可以<font color=#ff9f43>修改文件名字</font> | [root@hadoop104 hadoop-3.1.3]# `hadoop fs -get /sanguo/shuguo.txt ./shuguo2.txt` |



[root@hadoop104 hadoop-3.1.3]# hadoop fs -[operation]

Hadoop与Linux的指令差不多，区别尤其注意每个operation之前都要加 <font color=#ff9f43>-</font>

| Other Operation       |                                                              |
| :-------------------- | ------------------------------------------------------------ |
| -ls                   | 显示相关信息                                                 |
| -cat                  | 显示文件内容                                                 |
| -chgrp、-chmod、chown | 修改文件权限                                                 |
| -cp                   | 拷贝文件                                                     |
| -mv                   | 移动文件                                                     |
| -tail                 | 显示一个文件的末尾1kb的数据                                  |
| -rm                   | 删除文件或文件夹                                             |
| -rm -r                | 递归删除目录和目录中的内容                                   |
| -du                   | 统计文件夹的大小![image-20221231111230197](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231111230197.png)这里有两列数据，左边一列是表示本文件的大小，右边一列因为集群搭建了三台设备，因此有三个副本 |
| -setrep               | 设置HDFS中文件的副本数量这里                                                                                                                                          我设置了shuguo.txt的副本数量为10，在HDFS中可以看到是10，但是因为我们实际上只有三台主机（即3个DataNode），因此实际上只有3个副本。因此这里的10只是NameNode记录的数据，实际得看DataNode |

此外，在实际操作中，可以在<font color=#ff9f43>hadoop/bin</font>目录下使用`./hdfs dfs -put 文件A 目录B` 实现将文件A上传到hadoop的B目录下



### 2.3 API

若在XShell中操作，其实实际上是在Hadoop103<font color=#ff9f43>内部</font>操作，而Windows是客户端，是Hadoop的<font color=#ff9f43>外部</font>操作，因此Hadoop需要提供一些<font color=#ff9f43>API</font>给外部

<img src="https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231111635184.png" alt="image-20221231111635184" style="zoom:80%;" />

#### 2.3.1 Configuration in IDEAL

配置xml文件

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>3.1.3</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
    
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.36</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

编写log4j.proprties日志文件

```properties
log4j.rootLogger=INFO, stdout 
log4j.appender.stdout=org.apache.log4j.ConsoleAppender 
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout 
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n 
log4j.appender.logfile=org.apache.log4j.FileAppender 
log4j.appender.logfile.File=target/spring.log 
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout 
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

创建com.bcn.hdfs文件夹，随后就可以在文件夹中创建各种各样的文件了



#### 2.3.2 Routine（套路）

1. 获取资源（客户端对象）
2. 执行操作
3. 关闭资源

valpeopleDF=spark.sparkContext.textFile("hdfs://hadoop102:8020/input/people.txt").map(_.split(",")).map(attributes=>Person(attributes(0),attributes(1).trim().toInt)).toDF()



#### 2.3.3 Example -- File operation

**File Create Download Delete Rename Move Detail**

```java
package com.bcn.hdfs;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

/**
 * @author 大白菜
 * @date Created in 2022/7/22 18:50
 */

public class HdfsClient {
    private FileSystem fs;

    @Before
    public void init() throws URISyntaxException, IOException, InterruptedException {
        // 连接集群地址
        URI uri = new URI("hdfs://hadoop102:8020");
        // 创建一个配置文件
        Configuration configuration=new Configuration();
        // 用户
        String user="root";
        // 获取客户端对象
        fs = FileSystem.get(uri,configuration,user);
    }

    @After
    public void close() throws IOException {
        fs.close();
    }

    @Test
    public void testmkdir() throws IOException {
        // 创建一个文件夹
        fs.mkdirs(new Path("/xiyouji/huaguoshan1"));
    }
}

// 上传文件
@Test
public void testPut() therows IOException {
    // 参数一：是否删除原数据
    // 参数二：是否允许覆盖
    // 参数三：原数据路径
    // 参数四：目的数据路径
    fs.copyFromLocalFile(false,false,new Path("E:\\Hadoop and Spark\\TestData\\sunhukong.txt"),new Path("hdfs://hadoop102/xiyouji/hauguoshan1"));
}

//文件下载
@Test
public void testGet() throws IOException {
    // 参数一：是否删除原文件,参数二：原文件路径HDFS,参数三：目的地址Win,参数四:本地是否开启校验
    fs.copyToLocalFile(false,new Path("hdfs://hadoop102/xiyouji/huaguoshan1"),new Path("E:\\Hadoop and Spark"),false);
}

// 删除文件
@Test
public void testDel() throws IOException {
    // 参数一：要删除的路径,参数二：是否递归删除\
    // 删除文件
    fs.delete(new Path("hdfs://hadoop102/xiyouji/hauguoshan"),false);
    //删除空目录
    fs.delete(new Path("hdfs://hadoop102/xiyouji"),false);
    //删除非空目录
    fs.delete(new Path("hdfs://hadoop102/xiyouji"),true);
}

// 文件改名和移动（是同时发生的）
@Test
public void testmv() throws IOException {
    //参数一：原文件地址，参数二：目标文件地址
    fs.rename(new Path("hdfs://hadoop102/xiyouji/hauguoshan"),new Path("hdfs://hadoop102/xiyouji/huaguoshan"));
}

// 文件详细信息查看
@Test
public void testFileDetails() throws IOException {
   // 获取所有文件信息
    RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"),true);

    // 遍历文件
    while(listFiles.hasNext()) {
        LocatedFileStatus fileStatus = listFiles.next();
        System.out.println("========"+fileStatus.getPath()+"========");
        System.out.println(fileStatus.getOwner());
        System.out.println(fileStatus.getGroup());
        System.out.println(fileStatus.getLen());
        System.out.println(fileStatus.getModificationTime());
        System.out.println(fileStatus.getReplication());
        System.out.println(fileStatus.getBlockSize());
        System.out.println(fileStatus.getPath().getName());
    }
}

```



## 🔥2.4 Read and Write

#### 2.4.1 Write data

总体流程

- 建立连接
- 建立传输通道
- 正式传输数据

**建立连接**：类似计网中的四次握手

<font color=#ff9f43>Distributed FileStyle</font>：分布式文件系统，后续与NameNode建立连接的谈判代表人

<font color=#ff9f43>FSDataOutStream</font>：数据流对象，作为数据具体的传输介质

<img src="https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231145928966.png" alt="image-20221231145928966" style="zoom: 33%;" />



**建立传输通道**：就近原则，离哪个DataNode近，就先和哪个DataNode建立通道连接

![image-20221231150030260](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231150030260.png)

**正式传输数据**

<font color=#ff9f43>最小</font>传输单位是<font color=#ff9f43>chunk</font>，一个一个chunk组成一个<font color=#ff9f43>基本</font>传输单位<font color=#ff9f43>Packet</font>

传输的方式是并行+串行：一个个Packet传输到DataNode中，传输完毕马上到下一个DataNode，及和计网中的TCP的流水线式传输方式一样

![image-20221231150229078](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231150229078.png)



#### 2.4.2 Network topology

网络拓扑--节点距离计算

机架：物理上就是机房里可以放很多服务器的铁皮架子

![image-20221231150530848](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231150530848.png)

在HDFS写数据的过程中，上传数据的主机会选择离自己最近的DataNode上传数据，那么这个最近距离怎么计算呢？

**节点距离**：两个结点到达<font color=#ff9f43>最近</font>的<font color=#ff9f43>共同祖先</font>的距离总和

1. 同一结点上的进程，自己和自己的祖先当然也是自己，距离自然是0
2. 不同数据中心的节点：跳机架为1，跳集群为1，跳互利网（橙色的内容）为1，总共为3，两个节点自然就是6

![image-20221231150645853](C:\Users\大白菜\AppData\Roaming\Typora\typora-user-images\image-20221231150645853.png)

#### 2.4.3 Rack aware（机架感知）

作用：副本存储节点选择

![image-20221231150651770](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231150651770.png)

三个节点存储的策略如下

1. 第一个节点存储在本地，保证一个数据<font color=#ff9f43>最快</font>
2. 第二个存储在另一个机架，保证数据<font color=#ff9f43>可靠性</font>
3. 第三个存储在第二个节点的机架上，保证后续数据<font color=#ff9f43>传输效率</font>

![image-20221231150737399](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231150737399.png)

#### 2.4.4 Read data

读取是按照元数据中存储的内容，一个一个读取

![image-20221231150939566](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221231150939566.png)



### 2.5 NN and 2NN

#### 2.5.1 Working machanism（工作机制）

NameNode 中的元数据存储在内存中还是磁盘中呢？

1. 如果存储在磁盘中，需要进行随机访问，以及响应客户端请求，效率非常低，因此第一个排除
2. 如果存储在内存中，一旦断电整个集群就挂了，因此需要备份元数据--FsImage
3. 如果每次更新元数据时也更新FsImage那效率非常低，因此需要一个追加数据文件--Edits
4. 每次元数据更新，将更新的数据存储到Edits中，因此<font color=#ff9f43>元数据=FsImage+Edits</font> 

![image-20230101160158012](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20230101160158012.png)

#### 2.5.2 FsImage and Edits

NameNode被格式化后会在以下目录产生以下内容 

![image-20230101160226740](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20230101160226740.png)

1. Fsimage：元数据的<font color=#ff9f43>永久检查点</font> ，包含HDFS文件系统中所有目录和文件inode的序列化信息
2. Edits：存储HDFS文件系统中所有更新操作的路径
3. seen_txid:保存的是最后一个edits_的数字
4. 每次NameNode启动的时候都会将Fsimage文件读入内存，加载Edits里面的更新操作，对二者进行合并，保证元数据信息都是最新的



### 2.6 DataNode

#### 2.6.1 Work machanism

1. 启动后，DataNode主动向NameNode汇报工作，汇报自己的数据块信息
2. DataNode扫描所有块信息的时间：6h
3. DataNode汇报将所有块信息汇报给NN时间：6h
4. DN告诉NN我还活着时间：3s
5. NN判定DN不可用，没有接收到信息：10:30s

![image-20230101160331657](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20230101160331657.png)

#### 2.6.2 Data integrity

保证数据完整性策略

1. 当DataNode读取Block的时候，它会计算CheckSum
2. CheckSum如果与创建时的Block值不一样，说明Block已经挂了
3. Client读取其他DataNode上的Block

常见校验算法：crc（32）、md5（128）、shal（160）

比较low的有奇偶校验法：计算传输的数据中有多少个1，如果1的个数为偶数，则校验位为0

![image-20230101160453308](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20230101160453308.png)



## 三 MAPREDUCE

### 3.1 Overview

#### 3.1.1 WWH

Mapreduce是一个<font color=#ff9f43>分布式</font>运算程序的<font color=#ff9f43>编程框架</font>，核心功能是将用户写的<font color=#ff9f43>业务逻辑代码</font>和自身<font color=#ff9f43>默认代码</font>整合成一个<font color=#ff9f43>完整的分布式运算程序</font>，并发运行在一个Hadoop集群上

#### 3.1.2 Advantage

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |



#### 3.1.3 Disadvantage



#### 3.1.4 Compose

#### 3.1.5 Example--WordCount

#### 3.1.6 Routine

### 3.2 Serialization

![image-20230101160957286](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20230101160957286.png)

### 3.3 Core framework principles

InputFormat

OutFormat

Shuffle

Join

ETL

Conclusion

### 3.4 Compress(压缩)

What

Traits

How



### 3.5 Common problems and solutions



## 四 YARN

