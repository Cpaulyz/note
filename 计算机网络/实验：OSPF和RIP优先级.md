

# 1 小组成员

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |

# 2 实验目的

设计拓扑，对比RIP及OS PF优先级差异并作验证。

# 3 实验步骤

## 3.1 实验设备

PC：两台；

Router：四台；

交叉线；

直连线；

## 3.2 实验拓扑

![1589810389272](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174544361-2146890490.png)

## 3.3 IP地址规划

| 设备名称 | 端口                                        | 地址                                           |
| -------- | ------------------------------------------- | ---------------------------------------------- |
| PC0      | FastEthernet                                | 192.168.10.1                                   |
| PC1      | FastEthernet                                | 192.168.60.2                                   |
| 路由器1  | FastEthernet0/0<br>Serial2/0<br/>Serial3/0  | 192.168.10.2<br>192.168.20.1<br/>192.168.40.1  |
| 路由器2  | Serial2/0<br/>Serial3/0                     | 192.168.20.2<br>192.168.30.1                   |
| 路由器3  | Serial2/0<br/>Serial3/0                     | 192.168.50.1<br/>172.168.40.12                 |
| 路由器4  | FastEthernet0/0<br/>Serial2/0<br/>Serial3/0 | 192.168.60.1<br/>192.168.50.2<br/>192.168.30.2 |

# 4 实验步骤

1. 如图连接拓扑

2. 分别为PC0、PC1配置IP地址

3. 为R1、R2、R3、R4打开端口并分配IP地址

	```
	以R1为例：
	R1(config)#interface FastEthernet0/0
	R1(config-if)#ip address 192.168.10.2 255.255.255.0
	R1(config)#interface Serial2/0
	R1(config-if)#no shutdown
	R1(config-if)#clock rate 64000
	R1(config-if)#ip address 192.168.20.1 255.255.255.0
	R1(config)#interface Serial3/0
	R1(config-if)#no shutdown
	R1(config-if)#clock rate 64000
	R1(config-if)#ip address 192.168.40.1 255.255.255.0
	R1(config-if)#end
	```

4. 为PC配置默认网关，以PC0为例

	![1589205051835](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174544047-1074033762.png)

5. R1、R2、R3打开并配置rip，以R1为例

	```
	R1(config)#router rip
	R1(config-router)#network 192.168.10.0
	R1(config-router)#network 192.168.20.0
	```

6. 在R1上查看路由表，并PC0 ping PC1

	![1589809740510](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174543825-2009791293.png)

	报文走的链路如下

	![1589809772327](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174541726-527829895.png)

7. 为R1、R3、R4配置OSPF

	```
	R1(config)#route ospf 1
	R1(config-router)#network 192.168.10.0 0.0.0.255 area 0
	R1(config-router)#network 192.168.40.0 0.0.0.255 area 0
	
	R3(config)#route ospf 1
	R3(config-router)#network 192.168.40.0 0.0.0.255 area 0
	R3(config-router)#network 192.168.50.0 0.0.0.255 area 0
	
	R4(config)#route ospf 1
	R4(config-router)#network 192.168.50.0 0.0.0.255 area 0
	R4(config-router)#network 192.168.60.0 0.0.0.255 area 0
	```

8. 在R1上查看路由表，并PC0 ping PC1

	![1589809887645](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174543243-1309955169.png)

	![1589809912259](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174542772-1063866172.png)

	发现到192.168.60.0网段的路由发生了改变。

	判断OSPF的优先级高于RIP

9. 在R1上启动debug模式

	```
	debug ip rip
	```

	发现可以收到rip报文，有关于192.168.60.0网段的路由信息，但路由表仍以OSPF为准，说明OSPF优先级高于RIP

	![1589810269426](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174542479-117996745.png)

10. 在R1上停用ospf协议

	```
	R1(config)#no route ospf 1
	```

	发现

	![1589810144383](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174542196-1562390050.png)

	路由表发生改变，再次PC0 ping PC1，走的是RIP的链路

	![1589809772327](https://img2020.cnblogs.com/blog/1958143/202005/1958143-20200524174541726-527829895.png)

# 5 实验结论

OSPF的优先级高于RIP，在同时配置RIP和OSPF的情况，路由选择以OSPF为准。