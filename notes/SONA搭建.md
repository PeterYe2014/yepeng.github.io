# SONA搭建指南

## 搭建准备

### 需要的软件和版本

1. OpenStack Ocata/Pike 服务（Nova 、Neutron ）
2. OVS 2.6 or later
3. networking-onos

### Openstack 基础

主要服务：

Keystrone （认证）、Ceilometer（监控计费）、Horizon（UI）、Nova（计算）、Neutron（网络SDN）、Swift（对象存储）、Cinder（块存储）、Glance（镜像）、Heat（编排）

搭建方式：

1）DevStack （网络容易挂，系统一定16.04）选中

2）手动至少安装四个服务

3） Kolla-Ansible 容器化部署简单（配置很难做到一致）

4）Vagrant 

#### 搭建步骤

1. 安装VirtualBox

2. 导入控制器、compute、onos节点

3. 配置网络，修改桥接网络地址和host only网络地址

   ```
   主机网络地址范围为: 192.168.15.0/24
   控制器：192.168.15.137
   计算节点：192.168.15.138/139
   onos: 192.168.15.135
   ```

4. 启动openstack控制节点和计算节点

5. 登录onos节点，输入openstack-nodes命令，检测SONA环境搭建成功

#### 遇到的问题

Q:devstack  设置了HOST_IP，还是找不到，

A：virtualbox默认的nat子网是10.0.0.0/22，devstack不能找到这个网段下的ip地址，需要重新指定nat网段

Q: vagrant ssh登录不上

A：网络问题以及私钥不对，不同网卡的网段不要冲突

Q： 网络创建失败

A：内存和磁盘空间分配不够

Q: ONOS 2.4 启动失败

A: 安装java11

Q: 搭建compute节点，无法房访问keystone地址

A: 代理的原因，设置no_proxy环境变量，内网地址不使用代理

Q: compute节点报错:Didn't find service registered by hostname after 120 seconds

A：在local.conf中加入 enable_service placement-api

Q:  compute节点报错不能通过认证

A：local_conf密码设置错误，设置controller的密码

#### 节点网络设置

controller neo@192.168.15.137 (neo)

onos vagrant@192.168.15.135 (vagrant)

compute1 vagrant@192.168.15.138 (vagrant)

compute2 vagrant@192.168.15.139 (vagrant)

openstack控制页面：http://192.168.15.137/dashboard/project/instances/ (admin/abc123)

![image-20201116022736337](C:\Users\35114\AppData\Roaming\Typora\typora-user-images\image-20201116022736337.png)

![image-20201116022720744](C:\Users\35114\AppData\Roaming\Typora\typora-user-images\image-20201116022720744.png

onos控制页面：http://192.168.15.135:8181/onos/ui/index.html (karaf/karaf)

![image-20201116022704533](C:\Users\35114\AppData\Roaming\Typora\typora-user-images\image-20201116022704533.png)

进入onos cli，onos节点上：

```
ssh -p 8101 sdn@localhost (密码：sdn)
 
 onos> openstack-nodes
 .....
```

![image-20201116023151707](C:\Users\35114\AppData\Roaming\Typora\typora-user-images\image-20201116023151707.png)

![image-20201120172616769](C:\Users\35114\AppData\Roaming\Typora\typora-user-images\image-20201120172616769.png)

#### Openstack常见命令

```
# 查看计算服务节点
openstack compute service  list
# 查看主机
openstack host list
# 查看节点资源
openstack hypervisor show
# ip netns
# ip netns exec NETNS_NAME ssh USER@SERVER
# ip netns exec qdhcp-6021a3b4-8587-4f9c-8064-0103885dfba2 \
  ssh cirros@10.0.0.2
  
user:cirros
pass:cubswin:)

export JAVA_OPTS="-Dds.lock.timeout.milliseconds=15000"
```

#### OpenvSwitch命令

```
# 获取datapaht_id,配置integrationBridge
sudo ovs-vsctl get bridge br-int datapath_id
```

