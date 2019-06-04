## RabbitMQ安装说明 ##

### RabbitMQ环境搭建 ###

RabbitMQ与Erlang安装时，有版本对应关系，具体请看[**官网说明**](http://www.rabbitmq.com/which-erlang.html)。当前搭建环境的软件版本为：**Erlang 20.3**、**RabbitMQ 3.7.6**

**RabbitMQ** 是由 **Erlang** 开发的消息队列。所以首先要安装**Erlang**

一、安装**Erlang**

1、下载**Erlang**安装包，[**点击下载**](http://erlang.org/download/otp_win64_20.3.exe)


2、安装**Erlang**

（1）、双击下载好的安装包，点击next：

![](https://i.imgur.com/YpwmkuO.png)

（2）、修改安装路径，点击next：

![](https://i.imgur.com/sEIwW7C.png)

（3）、点击install，进行安装：

![](https://i.imgur.com/o61JPgb.png)


3、配置环境变量

（1）、右击计算机 --> 选择属性 --> 点击高级系统设置后如下图所示：

![](https://i.imgur.com/3g7Jmxb.png)

（2）、点击环境变量：

![](https://i.imgur.com/mkLaXlp.png)

（3）、在系统环境变量，点击新建：

![](https://i.imgur.com/ZnnsK2f.png)

（4）、输入变量信息，点击确定：

![](https://i.imgur.com/betq3RX.png)

（5）、在**Path**中进行变量配置：

![](https://i.imgur.com/KVbpDkh.png)

（6）、测试**Erlang**是否安装成功：

进入cmd，输入erl -v，查看是否能看到Erlang版本

![](https://i.imgur.com/0gZS5r0.png)

二、安装**RabbitMQ**

1、下载**RabbitMQ**安装包，[**点击下载**](https://github.com/rabbitmq/rabbitmq-server/releases)，选择3.7.6版本的exe文件

![](https://i.imgur.com/eDggpDr.png)

2、安装**RabbitMQ**

（1）、双击下载好的安装文件，点击next：

![](https://i.imgur.com/nxxqP5T.png)

（2）、修改安装路径，点击install：

![](https://i.imgur.com/yAWCICJ.png)

（3）、点击next:

![](https://i.imgur.com/x6qYbhr.png)

（4）、点击finish，完成安装：

![](https://i.imgur.com/mlkjClo.png)

三、开启消息队列

1、点击系统开始按钮-->RabbitMQ Server-->点击RabbitMQ Service - start

![](https://i.imgur.com/sCYaahk.png)

四、访问web控制台

1、在浏览器输入，[http://localhost:15672/#/](http://localhost:15672/#/)，默认登录名和密码都为**guest**

![](https://i.imgur.com/YRzU6gz.png)