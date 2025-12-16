# Windows Hadoop 环境配置完整指南

> 按顺序执行以下步骤，每一步都必须完成才能进行下一步。

---

## 第一步：安装 Java JDK 1.8

> ⚠️ **重要：必须使用 Java 8！** Java 17+ 版本会导致 `getSubject is not supported` 错误，Hadoop 不兼容。

### 1.1 下载
- 访问：https://www.oracle.com/java/technologies/downloads/#java8-windows
- 下载 **Windows x64 Installer** (jdk-8uXXX-windows-x64.exe)

### 1.2 安装
- 双击安装，安装路径建议：`C:\Java\jdk1.8.0_xxx`
- **注意：路径不要有中文和空格！**
- 如果安装到 `C:\Program Files\Java\jdk1.8.0_xxx`，后续配置需要使用短路径格式

### 1.3 配置环境变量
1. 右键"此电脑" → 属性 → 高级系统设置 → 环境变量
2. 在"系统变量"中点击"新建"：
   - 变量名：`JAVA_HOME`
   - 变量值：`C:\Java\jdk1.8.0_xxx`（你的实际安装路径）
3. 找到系统变量中的 `Path`，点击"编辑" → "新建"，添加：
   ```
   %JAVA_HOME%\bin
   ```

### 1.4 验证安装
打开 **新的** CMD 或 PowerShell，输入：
```cmd
java -version
```
应显示类似：`java version "1.8.0_xxx"`

---

## 第二步：安装 Hadoop 3.3.1

> ⚠️ **重要：使用 Hadoop 3.3.1 而不是 3.4.0！** Hadoop 3.4.0 在 Windows 上有 `posix:permissions not supported` 的 bug，无法运行 `hadoop jar` 命令。

### 2.1 下载 Hadoop
- 访问：https://archive.apache.org/dist/hadoop/common/hadoop-3.3.1/
- 下载：`hadoop-3.3.1.tar.gz`

### 2.2 解压
- 使用 7-Zip 或 WinRAR 解压到 `C:\hadoop-3.3.1`
- **最终目录结构应该是：** `C:\hadoop-3.3.1\bin`、`C:\hadoop-3.3.1\etc` 等

### 2.3 下载 Windows 补丁文件（重要！）
Hadoop 在 Windows 上需要额外的 winutils.exe 和 hadoop.dll：
1. 下载整个仓库：https://github.com/kontext-tech/winutils/archive/refs/heads/master.zip
2. 解压后找到 `winutils-master\hadoop-3.3.1\bin` 目录
3. 将该目录下的**所有文件**（winutils.exe、hadoop.dll、hdfs.dll 等）复制到 `C:\hadoop-3.3.1\bin\` 目录下，覆盖已有文件

### 2.4 配置环境变量
在"系统变量"中添加：

| 变量名 | 变量值 |
|--------|--------|
| `HADOOP_HOME` | `C:\hadoop-3.3.1` |

编辑 `Path` 变量，添加：
```
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

### 2.5 验证安装
打开 **新的** CMD，输入：
```cmd
hadoop version
```
应显示 Hadoop 版本信息。

---

## 第三步：创建必要的目录

打开 CMD（管理员权限），执行：
```cmd
mkdir C:\hadoop\data\namenode
mkdir C:\hadoop\data\datanode
mkdir C:\hadoop\tmp
```

---

## 第四步：修改 Hadoop 配置文件

配置文件都在 `C:\hadoop-3.3.1\etc\hadoop\` 目录下。

### 4.1 修改 hadoop-env.cmd
打开 `C:\hadoop-3.3.1\etc\hadoop\hadoop-env.cmd`，找到 `set JAVA_HOME=` 这一行，修改为：
```cmd
set JAVA_HOME=C:\Java\jdk1.8.0_xxx
```
（替换为你的实际 Java 路径，**不要用 %JAVA_HOME%**）

> ⚠️ **如果 Java 安装在 Program Files 目录（路径有空格）**，使用短路径格式：
> ```cmd
> set JAVA_HOME=C:\PROGRA~1\Java\jdk1.8.0_202
> ```

### 4.2 修改 core-site.xml
打开 `C:\hadoop-3.3.1\etc\hadoop\core-site.xml`，将整个内容替换为：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:///C:/hadoop/tmp</value>
    </property>
</configuration>
```
> ⚠️ **注意：Windows 路径必须加 `file:///` 前缀**，否则会报 `No class configured for C` 错误。

### 4.3 修改 hdfs-site.xml
打开 `C:\hadoop-3.3.1\etc\hadoop\hdfs-site.xml`，将整个内容替换为：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///C:/hadoop/data/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///C:/hadoop/data/datanode</value>
    </property>
</configuration>
```

### 4.4 修改 mapred-site.xml
打开 `C:\hadoop-3.3.1\etc\hadoop\mapred-site.xml`，将整个内容替换为：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>%HADOOP_HOME%/share/hadoop/mapreduce/*,%HADOOP_HOME%/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

### 4.5 修改 yarn-site.xml
打开 `C:\hadoop-3.3.1\etc\hadoop\yarn-site.xml`，将整个内容替换为：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

---

## 第五步：格式化 HDFS（仅首次执行）

打开 CMD，执行：
```cmd
hdfs namenode -format
```

看到 `Storage directory ... has been successfully formatted` 表示成功。

⚠️ **警告：这个命令只能执行一次！重复执行会清除所有数据！**

---

## 第六步：启动 Hadoop 服务

### 6.1 启动 HDFS
```cmd
start-dfs.cmd
```
会弹出两个新窗口（NameNode 和 DataNode），**不要关闭它们**。

### 6.2 启动 YARN
```cmd
start-yarn.cmd
```
会弹出两个新窗口（ResourceManager 和 NodeManager），**不要关闭它们**。

### 6.3 手动启动 NodeManager（如果缺失）
如果 `start-yarn.cmd` 没有启动 NodeManager，手动执行：
```cmd
start cmd /k "yarn nodemanager"
```

### 6.4 验证服务
```cmd
jps
```
应该看到以下进程：
- NameNode
- DataNode
- ResourceManager
- NodeManager

> ⚠️ **如果 `jps` 命令找不到**，使用完整路径：
> ```cmd
> "%JAVA_HOME%\bin\jps"
> ```

### 6.5 访问 Web 界面
- HDFS 管理界面：http://localhost:9870
- YARN 管理界面：http://localhost:8088

---

## 第七步：运行本项目

### 7.1 编译项目
在项目目录 `C:\Users\39025\Desktop\project2` 下打开 CMD：
```cmd
mvn clean package
```

### 7.2 上传数据到 HDFS
```cmd
hdfs dfs -mkdir -p /input
hdfs dfs -put data_fix /input/
```
> ⚠️ **注意：Windows CMD 不支持 `*` 通配符**，直接上传整个目录即可。

### 7.3 运行 MapReduce 作业
```cmd
hadoop jar target/factor-calculation-1.0.0.jar com.quantum.FactorCalculationDriver /input/data_fix /output/factors /output/final
```

### 7.4 查看结果
```cmd
hdfs dfs -cat /output/final/part-r-00000
```

### 7.5 下载结果到本地
```cmd
hdfs dfs -get /output/final/part-r-00000 result.csv
```

---

## 常见问题

### Q1: `hadoop` 命令找不到
**解决：** 检查环境变量 `HADOOP_HOME` 和 `Path` 是否正确配置，然后**重新打开** CMD。

### Q2: 启动 HDFS 时报错 "JAVA_HOME is not set"
**解决：** 检查 `hadoop-env.cmd` 中的 JAVA_HOME 路径是否正确，使用完整路径而不是变量。

### Q3: 格式化 NameNode 失败
**解决：** 
1. 删除 `C:\hadoop\data` 和 `C:\hadoop\tmp` 目录
2. 重新创建目录
3. 再次执行 `hdfs namenode -format`

### Q4: DataNode 启动失败
**解决：**
1. 停止所有服务：`stop-dfs.cmd` 和 `stop-yarn.cmd`
2. 删除 `C:\hadoop\data\datanode` 下的所有文件
3. 重新启动：`start-dfs.cmd`

### Q5: 运行 MapReduce 时报 "Output directory already exists"
**解决：** 删除已存在的输出目录：
```cmd
hdfs dfs -rm -r /output/factors
hdfs dfs -rm -r /output/final
```

---

## 停止服务
```cmd
stop-yarn.cmd
stop-dfs.cmd
```

---

## 快速检查清单

- [ ] Java JDK 1.8 已安装，`java -version` 正常显示
- [ ] Hadoop 3.3.1 已解压到 `C:\hadoop-3.3.1`
- [ ] winutils.exe 和 hadoop.dll 已放入 bin 目录
- [ ] 环境变量 JAVA_HOME 和 HADOOP_HOME 已配置
- [ ] 四个配置文件已修改（core-site.xml, hdfs-site.xml, mapred-site.xml, yarn-site.xml）
- [ ] hadoop-env.cmd 中的 JAVA_HOME 已设置
- [ ] HDFS 已格式化
- [ ] `jps` 显示 4 个进程
- [ ] http://localhost:9870 可以访问
