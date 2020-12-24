# Window Server服务

## SQL Server Cheetsheet

### 基本操作

命令行登录

```bash
sqlcmd -S localhost -U SA -P '<YourPassword>'
```

创建库、表，插入

```
CREATE DATABASE TestDB
SELECT Name from sys.Databases
GO # SQL SERVER需要使用GO来执行SQL 语句 
```

```
USE TestDB
CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT)
INSERT INTO Inventory VALUES (1, 'banana', 150); INSERT INTO Inventory VALUES (2, 'orange', 154);
GO
```

### 数据库配置

#### 使用mssql-conf 配置

设置内存限制 memory.memorylimitmb , 修改内存限制为3328MB

```
sudo /opt/mssql/bin/mssql-conf set memory.memorylimitmb 3328
sudo systemctl restart mssql-server
```

#### 使用环境变量配置

**MSSQL_MEMORY_LIMIT_MB**  内存限制

**MSSQL_TCP_PORT**  监听端口

### 数据库同步

业务需求需要将两个数据库进行同步，主要的方法有分为四类：发布订阅、镜像、日志同步、脚本

##### 发布订阅

主服务器通过发布数据的更改（对视图和表进行监控），备份服务器订阅消息，同步到本地，主要发布方式有：

1. 快照发布

   通过打包发送的数据为快照传递给备服务器，带宽需求量大

2. 事务发布

   初始通过快照同步，后续通过事务来同步

3. 合并发布

   在订阅服务器收到已发布数据的初始快照后，发布服务器和订阅服务器可以独立更新已发布数据。更改会定期合并

该方法一个缺点就是要选择发布的对象，主服务服务如果新建表或者视图后，就可能同步不了了（没有在发布对象中注册）

##### 镜像（不推荐，后续被移除）

通过镜像服务器来同步数据，只适用于使用完整恢复模式的数据库。数据库镜像涉及尽快将对主体数据库执行的每项插入、更新和删除操作“重做 ”到镜像数据库中。

##### 日志同步

日志传送以自动将“主服务器”实例上“主数据库”内的**事务日志**备份发送到单独“辅助服务器”实例上的一个或多个“辅助数据库”，故此该方法常用于数据到多个目标数据库

##### 脚本或三方软件

自己实现同步脚本，自定义程度高，不需要主服务器配合，但工作量大；三方软件有bug和安全风险；

### Window Server 2012 密钥

标准版 = NB4WH-BBBYV-3MPPC-9RCMV-46XCB

　　　　MMPXK-NBJDQ-JPM34-WX3FM-G276W