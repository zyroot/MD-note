## 一、开始环境配置：

```properties
Editor--> Colors&Font --> Font 设置字体大小

Editor--> Color&Font --> Console Font 控制台字体大小

Editor--> File Encodings   文件编码格式

Keymap                   快捷键设置--导jar包 File--import Setting

Editor--> General--> Auto Import 自动导包设置  All

Appearance&Behavior --> Appearance  Theme 主题设置
Editor--> Color&Font --> Language Defaults --> Comments --> Block comment(块注释设置/***/) 
--> Line comment(注释设置//)喜欢的颜色类型：3875D6

场景
Editor-->Colors&Font-->General-->Text-->Default text-->Background 	点击设置护眼颜色CD7EDCC
默认情况下，每次打开Intellij IDEA，都会连带着打开上次打开的项目。如果不希望它每次打开时都连带的打开上次的项目，可通过“系统设置”进行配置。
配置方法
如下图所示，找到Intellij配置中的System Settings，右边的Reopen last project on startup，默认为勾选状态，即每次打开IDE时，会打开上次的项目。将此勾选去掉即可。同时，在Project Opening中还可以进一步配置打开新项目时是否新开一个窗口，是否在当前窗口，是否通过提示的方式让操作者进行选择。

Intellij IDEA生成serialVersionUID
Preferences -> Inspections -> Serialization issues -> Serialization class without 'serialVersionUID' 打上勾
将光标放到类名上，按atl＋enter键，就会提示生成serialVersionUID了
```

## 二、快捷键：

### 快捷键技巧：

1.查找接口的实现类：

IDEA 风格       ctrl + alt +B

2.对应的ctrl+shift+y可以切换为小写。

3.CTRL+ALT+T    把选中的代码放在 *TRY*{} IF{} ELSE{} 里 

4.main方法 ： psvm

5.可以把方法都折叠起来。ctrl + 减号   ， Ctrl + 加号

6.IDEA_查找接口的实现 的快捷键 :  **CTRL+ALT+B**



# 三、MAVEN设置；

给maven 的settings.xml配置文件的profiles标签添加

```xml
<profile>
  <id>jdk-1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```

# 四、IDEA设置

整合maven进来；

![idea设置](images/搜狗截图20180129151045.png)



![images/](images/搜狗截图20180129151112.png)