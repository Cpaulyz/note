# VLAN间路由实验

## 1 小组成员

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |

## 2 实验目的

设计拓扑，验证VLAN间路由，使用两种方式（三层交换机和单臂路由）实现vlan间通信并试分析其性能差异。

### 实验设备

PC：四台；

Router：一台；

交换机：三台；

三层交换机：一台；

交叉线；

直连线；

## 3 单臂路由

### 3.1 实验拓扑

![1590311151432](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150116715-901827584.png)

### 3.2 实验步骤

1. 如图连接拓扑，配置四台PC的地址和默认网关

2. 在S1、S2上规划VLAN10、VLAN20，并设置中继端口，本征VLAN为99

    ```
    以S1上的配置为例
    // 配置VLAN
    S1(config)#vlan 10
    S1(config-vlan)#vlan 20
    S1(config-vlan)#end
    // 配置端口
    S1(config)#in F0/11
    S1(config-if)#switchport mode access
    S1(config-if)#switchport access vlan 10
    S1(config)#in f0/12
    S1(config-if)#switchport mode access
    S1(config-if)#switchport access vlan 20
    S1(config-if)#exit
    S1(config)#in f0/1
    S1(config-if)#switchport mode trunk 
    S1(config-if)#switchport trunk native vlan 99
    // 查看vlan
    S1#show vlan brief
    ```

    ![1590311356701](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150116452-1608465419.png)

3. 配置路由器子端口

    ```
    Router(config)#in f0/0.10
    Router(config-subif)#encapsulation dot1Q 10
    Router(config-subif)#ip address 192.168.10.1 255.255.255.0
    Router(config-subif)#in f0/0.20
    Router(config-subif)#encapsulation dot1Q 20
    Router(config-subif)#ip address 192.168.20.1 255.255.255.0
    Router(config-subif)#in f0/0
    Router(config-if)#no shut
    Router(config-if)#no shutdown 
    ```

4. 配置S3

    ```
    S3(config)#in f0/1
    S3(config-if)#switchport mode trunk 
    S3(config-if)#switchport trunk native vlan 99
    S3(config-if)#in f0/2
    S3(config-if)#switchport mode trunk 
    S3(config-if)#switchport trunk native vlan 99
    S3(config-if)#in f0/3
    S3(config-if)#switchport mode trunk 
    S3(config-if)#switchport trunk native vlan 99
    ```

5. 四台PC互相ping，发现都能ping成功，说明实现了单臂路由的VLAN间通信

    ![1590311485597](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150116214-1797785721.png)

## 4 三层交换机

### 4.1 实验拓扑

![1590331949307](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150115992-127362981.png)

### 4.2 实验步骤

1. 将把S3和路由器换成三层路由器，如上图

2. 打开三层交换机的路由功能

      ```
      Switch(config)#ip routing
      ```

3. 将F0/1、F0/2口改为trunk
    ```
    Switch(config)#interface FastEthernet0/1
    Switch(config-if)#switchport mode trunk
    Switch(config-if)#switchport trunk native vlan 99
    Switch(config)#interface FastEthernet0/2
Switch(config-if)#switchport mode trunk
    Switch(config-if)#switchport trunk native vlan 99
    ```
    
4. 配置VLAN

    ```
    Switch(config)#in vlan 10
    Switch(config-if)#ip address 192.168.10.1 255.255.255.0
    Switch(config-if)#no shut
    Switch(config-if)#no shutdown 
    Switch(config-if)#exit
    Switch(config)#in vlan 20
    Switch(config-if)#ip address 192.168.20.1 255.255.255.0
    Switch(config-if)#no shutdown 
    Switch(config-if)#exit
    ```

5. 四台PC互相ping，发现都能ping成功，说明实现了三层交换机的VLAN间通信

    ![1590332003489](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150115417-1431756528.png)

## 5 对比性能差异

分别192.168.10.2 ping 192.168.20.3 -n 100

1.  单臂路由

	![1590312882989](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150115192-2076107011.png)

2. 三层交换机

	![1590332148206](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150114881-1472562069.png)

3.  说明单臂路由的性能较三层交换机较差。

### 差异原因

将拓扑修改为如下，发现这时候性能几乎没有差距

![1590332231245](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150114646-789405758.png)

![1590332250127](https://img2020.cnblogs.com/blog/1958143/202006/1958143-20200621150114387-2061209416.png)

推测这种性能上的差异可能是因为三层交换机具有路由器+交换机的功能，可以少用一个交换机所导致的。