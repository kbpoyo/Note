### Maven简介

![](http://imgs.kbpoyo.top/imgs/Maven_202207092055278.png)

![](http://imgs.kbpoyo.top/imgs/Maven_202207092059027.png)


### 下载与安装

**下载**
[Maven官网](https://maven.apache.org/)
[Maven – Download Apache Maven](https://maven.apache.org/download.cgi)

**安装**
- Maven环境变量配置
	- 依赖*java*，需要配置*JAVA_HOME*
	- 设置*Maven*自身运行环境，需要配置*MAVEN_HOME*
	- 测试环境配置结果：命令行输入*MVN*指令

### Maven基础概念

**中央仓库**
[Central Repository: (maven.org)](https://repo1.maven.org/maven2/)

- **仓库**：用于存储资源，包含各种*jar*包
- **仓库分类**：
	- *本地仓库*：自己电脑上存储资源的仓库，连接远程仓库获取资源
	- *远程仓库*：非本机上的仓库，为本地仓库提供资源
		- 中央仓库：Maven团队维护，存储所有资源的仓库
		- 私服：部门/公司范围内存储资源的仓库，从中央仓库获取资源
			- 私服的作用：
				- 保存具有版权的资源，包含购买或自主研发的jar
				- 一定范围内共享资源，仅对内部开放，不对外开放

![](http://imgs.kbpoyo.top/imgs/Maven_202207092109247.png)

- **坐标**
	- 什么是坐标：
		- Maven中的坐标用于描述仓库中资源的位置
	- 坐标的主要组成
		- *groupid*：定义当前Maven项目隶属组织名称（通常是域名反写，例如：org.mybatis)
		- *artifactid*：定义当前Maven项目的名称（通常是模块名称）
		- *version*：定义当前项目版本号
	- 坐标的作用：
		- 使用唯一标识，定位资源的位置，通过该标识可以将资源的识别与下载工作交由电脑完成

**本地仓库配置**
修改maven安装目录下*conf/settings.xml*，配置自己的本地仓库
![](http://imgs.kbpoyo.top/imgs/Maven_202207092121164.png)
![](http://imgs.kbpoyo.top/imgs/Maven_202207091720888.png)


**远程仓库配置**
- Maven默认连接的远程仓库位置
![](http://imgs.kbpoyo.top/imgs/Maven_202207092126495.png)

- 在settings.xml文件中配置*阿里云镜像仓库*
![](http://imgs.kbpoyo.top/imgs/Maven_202207092127262.png)

**全局setting与用户setting的区别**
- 全局setting定义了当前计算机中的Maven公共配置
- 用户setting定义了当前用户的配置



### 手工制作Maven项目

### IDEA生成Maven项目

**配置Maven**
![](http://imgs.kbpoyo.top/imgs/Maven_202207092131211.png)

**原型创建Java项目**
![](http://imgs.kbpoyo.top/imgs/Maven_202207092132065.png)

**原型创建Web项目**
![](http://imgs.kbpoyo.top/imgs/Maven_202207092133196.png)



### 依赖管理

- **依赖配置**
	- 依赖指当前项目运行所需要的jar，一个项目可以设置多个依赖
	- 格式：
		- ![](http://imgs.kbpoyo.top/imgs/Maven_202207092136273.png)
- **依赖传递**
	- 直接依赖：当前项目中通过依赖配置建立的依赖关系
	- 间接依赖：被依赖的资源如果依赖其他资源，则当前项目间接依赖其他资源
		- ![](http://imgs.kbpoyo.top/imgs/Maven_202207092138772.png)
- **依赖传递冲突问题**
	- 路径优先：当前依赖中出现相同资源时，层级越深，优先级越低，层级越浅优先级越高
	- 声明优先：当资源在相同层级被依赖时，其父级依赖的配置顺序靠前的覆盖配置顺序靠后的
	- 特殊优先：当同级配置了相同资源的不同版本，后配置的覆盖先配置的
- **可选依赖**
	- 对外隐藏当前所依赖的资源--不透明
		- ![](http://imgs.kbpoyo.top/imgs/Maven_202207092144298.png)
- **排除依赖**
	- 主动断开依赖的资源，被排除的资源无需指定版本（去除依赖中不需要的资源）--不需要
		- ![](http://imgs.kbpoyo.top/imgs/Maven_202207092146762.png)
- **依赖范围**
	- 依赖的jar默认情况可以在任何地方使用，可以通过*scope*标签设定其作用范围
	- 作用范围
		![](http://imgs.kbpoyo.top/imgs/Maven_202207092150406.png)

- **依赖范围的传递性**
	- 带有依赖范围的资源在进行传递时，作用范围将受到影响
	- ![](http://imgs.kbpoyo.top/imgs/Maven_202207092151159.png)
	

### 生命周期与插件

**项目构建生命周期**
>Maven 构建生命周期描述的时一次构建过程中经历了多少个事件

- **三套生命周期**
	- *clean*：清理工作
	- *default*：核心工作，例如编译，测试，打包，部署等
	- *site*：产生报告，发布站点等
- **clean的生命周期**
	- ![](http://imgs.kbpoyo.top/imgs/Maven_202207092200954.png)
- **site的生命周期**
	- ![](http://imgs.kbpoyo.top/imgs/Maven_202207092201160.png)
- **default的生命周期**
	- ![](http://imgs.kbpoyo.top/imgs/Maven_202207092202500.png)

>生命周期中的任意指令被执行，都会从生命周期开始的每一条指令执行到被执行指令处


**插件**
![](http://imgs.kbpoyo.top/imgs/Maven_202207092204990.png)
