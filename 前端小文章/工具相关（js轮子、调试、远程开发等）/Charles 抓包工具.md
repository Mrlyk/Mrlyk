# Charles 抓包工具

Charles 通过将自己设置成系统的网络访问代理服务器，使得所有的网络访问请求都通过它来完成，从而实现了网络封包的截取和分析。

和 Wireshark 分析网络的工具不同，我们用它来

- 截取 Http 和 Https 网络封包
- 支持重发网络请求，方便后端调试
- 支持修改网络请求参数
- 支持网络请求的截获并动态修改
- 支持模拟慢速网络
- 支持转发网络请求

[toc]

无论是 PC 还是移动端抓包，**都需要设置代理**。Charles 通过代理网络来完成抓包请求。所以我们从设置代理开始，看看后面 charles 能做什么。

## 设置代理

#### PC

PC 以 Mac 为例，在网络选项中开启网页代理并配置端口（默认是开启的）

![image-20220524095304550](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524095304550.png?x-oss-process=image/resize,w_600,m_lfit) 

之后在 Charles 的 Proxy 选项中设置同样的端口

![image-20220524095634911](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524095634911.png?x-oss-process=image/resize,w_400,m_lfit) ![image-20220524095647730](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524095647730.png?x-oss-process=image/resize,w_400,m_lfit) 

这样就可以监听到 PC 发出的请求，之后进行处理了。

#### 移动端

移动端思路差不多，设置起来稍微麻烦点，具体可以查看参考文章。基本步骤如下

1. 开启 charles proxy `Enable transparent HTTP proxying` 
2. 设置手机 wifi 的 ip 地址和端口和 pc 相同（要在同一 wifi 下）
3. 安装 charles 证书：在 charles -> help -> ssl proxying -> install charles root certificate
4. 设置 SSL 代理：通过主菜单打开 **Proxy | SSL Proxy Settings** 弹窗，勾选 `Enable SSL proxying`。host 和 port 都设置为*即可
5. 手机安装证书，手机访问`chls.pro/ssl`安装即可，安装后信任

*安卓要更麻烦一点，需要手动生成证书，具体查看参考文章*



**注意：charles 的代理和自己的小飞机代理会冲突，只能开启其中一个！** 

##  常用功能

#### 模拟慢网络

在主界面上开启`start throttling`，之后在 proxy 中配置`Throttles Settings`。

默认针对所有网络开启，也可以只配置选中的 host

#### 断点网络请求

在主界面上开启`enable breakpoints`，之后在 proxy 中配置`Breakpoint Settings`。

之后添加要拦截的请求即可。断点后我们也可以对请求、响应进行修改，**要注意响应的超时时间**。

修改响应：在 breakpoint 界面的 Edit Response 中直接修改即可。然后点击 Excute 生效！

#### 远程/本地映射

可以通过远程/本地映射功能，将请求转发到远程/本地服务。

**透过这一点我们可以让生产的代码也具有 sourcemap 来调试**，即将生产的 sourcemap 请求都转发到测试环境或者本地服务。

开启这个功能只需要在对应的请求上右键 -> map local & map remote。然后配置对应的服务地址即可。

- map local: 将响应映射到本地 json 文件
- map remote: 将响应映射到远程服务

#### 请求重写

可以修改请求头、响应头，相应信息。我们可以用它来 mock 请求。

选中要处理的请求后，在`Tools`中，选择 `Rewrite`选项，之后弹出如下设置栏。左侧添加分组

- 右侧上半部分配置要拦截的请求
- 右侧下半部分配置要修改的类型（body、header、status...）

![image-20220524110528196](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524110528196.png?x-oss-process=image/resize,w_600,m_lfit) 

**注意：可以在主界面的 tools 中配置`no caching`，防止读取缓存而重写不生效。**

## 参考文章

Charles 使用指南：https://juejin.cn/post/7024884866781020167