# uni-app

uni-app 因其方便的跨平台能力，现在是通用的小程序开发解决方案。

其一般项目目录结构如下：

```
┌─uniCloud              云空间目录，阿里云为uniCloud-aliyun,腾讯云为uniCloud-tcb（详见uniCloud）
│─components            符合vue组件规范的uni-app组件目录
│  └─comp-a.vue         可复用的a组件
├─utssdk                存放uts文件
├─pages                 业务页面文件存放的目录
│  ├─index
│  │  └─index.vue       index页面
│  └─list
│     └─list.vue        list页面
├─static                存放应用引用的本地静态资源（如图片、视频等）的目录，注意：静态资源只能存放于此
├─uni_modules           存放[uni_module](/uni_modules)。
├─platforms             存放各平台专用页面的目录，详见
├─nativeplugins         App原生语言插件 详见
├─nativeResources       App端原生资源目录
│  └─android            Android原生资源目录 详见
├─hybrid                App端存放本地html文件的目录，详见
├─wxcomponents          存放小程序组件的目录，详见
├─unpackage             非工程代码，一般存放运行或发行的编译结果
├─AndroidManifest.xml   Android原生应用清单文件 详见
├─main.js               Vue初始化入口文件
├─App.vue               应用配置，用来配置App全局样式以及监听 应用生命周期
├─manifest.json         配置应用名称、appid、logo、版本等打包信息，详见
├─pages.json            配置页面路由、导航条、选项卡等页面类信息，详见
└─uni.scss              这里是uni-app内置的常用样式变量
```

## 条件编译

uni-app 可以指定特定一块的代码只编译到某个平台，我们可以通过“魔法注释”来做到这一点。

```vue
<template>
	<view>
    <!-- #ifdef MP-WEIXIN -->
  	<view>小程序</view>
    <!-- #endif -->
    <!-- #ifdef APP-PLUS -->
    <view>app</view>
     <!-- #endif -->
  </view>
</template>
```

## 路由配置

uni-app 不支持 vue router，所以不要想着用它来配置路由，毕竟他和传统的 vue 项目还是有区别的。

配置路由需要在**根目录的 pages.json 中声明**

官方文档：https://uniapp.dcloud.net.cn/collocation/package.html

一个简单示例如下：

```json
{
	"pages": [{
		"path": "pages/index/index",
		"style": {
				"navigationBarTitleText": "关联记录",
				"enablePullDownRefresh": true
		}
	}, {
		"path": "pages/login/login",
		"style": { ...}
	}],
	"subPackages": [{
		"root": "pagesA",
		"pages": [{
			"path": "list/list",
			"style": { ...}
		}]
	}, {
		"root": "pagesB",
		"pages": [{
			"path": "detail/detail",
			"style": { ...}
		}]
	}]
}

```

路由声明后，我们直接在根目录下的 pages 文件夹中创建对应的 vue 文件即可！

其中`style` 

有点像前端新出的 astro 框架的路由规则——基于文件路径对应。