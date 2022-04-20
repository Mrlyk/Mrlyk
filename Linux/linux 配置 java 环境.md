# linux 配置 java 环境记录

java 官方下载地址：https://www.oracle.com/java/technologies/downloads/#java8

#### 1、下载文件

根据服务器架构选择要下载的目标版本，一般 64 位的压缩包即可

![image-20220324145423976](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220324145423976.png?x-oss-process=image/resize,w_800,m_lfit) 

#### 2、解压文件

```shell
tar -zxvf xxxx.gzip -C /usr/local/java
```

如果是下载的二进制包还需要进行编译

#### 3、添加环境变量

```.profile
export JAVA_HOME=/usr/local/java/jdk-18
export PATH=${JAVA_HOME}/bin:$PATH
```

