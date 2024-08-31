# Oracle


## Net Configuration Assistant和Net Manager的区别

在oracle的配置工具中，Net Configuration Assistant（网络配置助手）和Net Manager（网络管理器）都可以配置监听和TNS，但这两个工具又各有不同:

1.   
<span style="color: rgb(255, 0, 0);">
<b>
Net Manager只是修改参数，并不对现有服务进行更新（启动或重启）; <br>
Net Configuration Assistant,向导式操作，配置简单，会更新现有的服务
</b>
</span>
2.  
<span style="color: rgb(255, 0, 0);"><b>
用Net Manager新增名为test1的监听并设置参数，保存网络设置后，在服务中是看不到名为test1的监听服务的。<br>
而Net Configuration Assistant配置一个test3的监听文件，这时可以看到test3的监听服务了，并且它是启动的。
</b></span>

`
可通过“lsnrctl status”命令查看监听启动情况，“lsnrctl start”(lsnrctl stop)可开启或关闭监听,
命令“netca”可启动Net Configuration
`


## 配置监听与服务
![](../../document/images/public/oracle/2017-06-11_234539.png)
```
当建立了Oracle数据库后，必须合理地配置监听程序和网络服务名之后，客户应用才能访问该数据库。其中，监听程序是在服务器端配置，网络服务名是在客户端配置。
1、服务器端的监听程序用于接收客户端的连接请求。
在建立了Oracle数据库之后，为了使得客户应用可以访问特定数据库，必须要在监听程序中追加该数据库。 一个监听程序可以监听多个Oracle数据库，多个监听程序也可以监听同一个数据库。但是监听程序只能用于同一台服务器上的Oracle数据库。
安装了Oracle时，会自动建立默认的监听程序LISTENER.一般只需要将需要使用的数据库追加到这个监听程序上就可以了。 监听程序使用的默认端口为1521 保存了监听程序配置之后，必须要重新启动监听程序才能生效。windows中可以在“服务”中重新启动。
2。客户端需要配置网络服务名，应用程序使用网络服务名才能访问Oracle数据库 ,一般使用数据库名作为服务名 网络协议要与监听程序的一致 。
```

### 监听配置
**方式一：通过 Net Manager**  
打开Net Manager  ----> 展开监听程序 ------->  对 '**数据库服务'** 和 '**监听位置'** 进行设置。
![](../../document/images/public/oracle/Image.png)
![](../../document/images/public/oracle/Image1.png)
`保存配置后，重启监听程序`

**方式一：方式二：通过 Net Configuration Assistant**
打开 Net Configuration Assistant --------> 选择 '监听程序配置'
![](../../document/images/public/oracle/Image2.png)
根据需要选择 添加、删除、或者 重新配置，下一步
![](../../document/images/public/oracle/Image3.png)
输入监听程序名称：
![](../../document/images/public/oracle/Image4.png)
选择网络协议，默认为TCP/IP：
![](../../document/images/public/oracle/Image5.png)
配置端口号，默认1521：
![](../../document/images/public/oracle/Image6.png)
是否配置另一个监听程序，默认 '否'，下一步完成。
![](../../document/images/public/oracle/Image7.png)
`该方式不必重启监听程序`

### 网络服务名配置
**方式一：通过 Net Manager**
打开Net Manager  ----> 选中 ‘服务命名’
![](../../document/images/public/oracle/Image8.png)
![](../../document/images/public/oracle/Image9.png)

**方式二：通过 Net Configuration Assistant**
打开 Net Configuration Assistant --------> 选择 '本地网络服务名配置'
![](../../document/images/public/oracle/Image10.png)
根据需要选择 添加、删除、或者 重新配置，下一步
![](../../document/images/public/oracle/Image11.png)
![](../../document/images/public/oracle/Image12.png)