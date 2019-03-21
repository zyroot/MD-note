# GIT心得体会

# 一、安装git.exe

--------下一步，下一步

# 二、克隆项目

从gitlab克隆项目时，使用SSH协议还是HTTP协议**

- SSH协议？请参考：[SSH协议百度百科](https://baike.baidu.com/item/ssh/10407?fr=aladdin)
- HTTP协议？请参考：[HHTP协议](https://baike.baidu.com/item/http/243074?fr=aladdin&fromid=1276942&fromtitle=HTTP%E5%8D%8F%E8%AE%AE)
- 当从gitlab仓库中克隆项目时，推荐使用SSH协议，如果不适用SSH协议，在克隆过程中会发生错误。

# 三、为gitlab账号添加SSH key

1、登录个人gitlab账号，在账号的profiles中点击设置，出现SSH keys的页面。



2、打开tortoiseGit中的PuttyKey Gen生成私钥、公钥。将生成的公钥，拷贝到设置页面中，并add Key。

```java
由于本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以必须要让github仓库认证你SSH key，在此之前，必须要生成SSH key。

 
第1步：创建SSH Key。在windows下查看[c盘->用户->自己的用户名->.ssh]下是否有id_rsa、id_rsa.pub文件，如果没有需要手动生成。
打开git bash，在控制台中输入以下命令。

1
$ ssh-keygen -t rsa -C "youremail@example.com"
密钥类型可以用 -t 选项指定。如果没有指定则默认生成用于SSH-2的RSA密钥。这里使用的是rsa。

同时在密钥中有一个注释字段，用-C来指定所指定的注释，可以方便用户标识这个密钥，指出密钥的用途或其他有用的信息。所以在这里输入自己的邮箱或者其他都行。

输入完毕后程序同时要求输入一个密语字符串(passphrase)，空表示没有密语。接着会让输入2次口令(password)，空表示没有口令。3次回车即可完成当前步骤，此时[c盘>用户>自己的用户名>.ssh]目录下已经生成好了。


第2步：登录github。打开setting->SSH keys，点击右上角 New SSH key，把生成好的公钥id_rsa.pub放进 key输入框中，再为当前的key起一个title来区分每个key。
```

3、完成SSH key的配置

# 四、从gitlab上克隆项目到本地

从远端克隆项目到本地直接拉取即可。