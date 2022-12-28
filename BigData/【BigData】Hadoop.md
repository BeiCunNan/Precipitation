# 【BigData】Hadoop



## 一 、Overview

###  2.1 Big Data

四个特点：4V

编程

## 二 HDFS

### 2.1 Overview

#### 2.1.1 Why

随着数据量的增多，需要将数据存储到不同的OS中，需要使用到**分布式文件管理系统**，HDFS(Hadoop Distributed File System)就是一种分布式文件管理系统，用来**存储、管理**文件。适合一次性读入，多次读出的场景



#### 2.1.2 Advantages

- 高容错性：数据自动保存多个副本，丢掉一个副本后可自动恢复
- 适合处理大数据：能够处理PB级和百万规模以上的文件数量
- 成本低：构建在廉价机器上



#### 2.1.3 Disadvantage

- 不适合低时延数据访问
- 无法高效的对大量小文件进行存储：无论大文件还是小文件都需要NameNode来存储目录和块信息，对于小文件来说资源其实是很浪费的
- 不支持并发写入、文件随机修改：仅支持数据append追加



#### 2.1.4 Compose

**NameNode(NN)**：存储文件的**元数据**，如文件名、文件目录结构、文件属性（生成时间、副本数、文件权限）以及每个文件的**块列表**和块所在的DataNode等

**DataNode(DN)**：在本地文件系统存储文件块数据以及块数据的校验和，执行数据块的读/写操作

**Secondary NameNode(2NN)**：小秘书，每隔一段时间对NameNode元数据进行备份（不是热备份），若NN挂掉了，需要一段时候来替换NN

NN下达指令，DN进行实际操作

![image-20221226221619173](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221226221619173.png)

#### 2.1.5 File block

Hadoop2.X和Hadoop3.X都是128Mb，Hadoop1.X是64Mb

为什么块的大小既不能设置太小也不能设置太大呢？

- 若设置太小，增加了寻址时间
- 若设置太大，计算机传输速度和处理速度过慢
- **大小取决于你计算机本身的传输速率**

<font color=#C1CDCD >但是</font>

### 2.2 Shell





### 2.3 API



## 三 MAPREDUCE





## 四 YARN

