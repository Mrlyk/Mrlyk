# midway faas

Midway 是阿里巴巴 - 淘宝前端架构团队，基于渐进式理念研发的 Node.js 框架，通过自研的依赖注入容器，搭配各种上层模块，组合出适用于不同场景的解决方案。

文档：https://midwayjs.org/docs/quickstart

Midway 为 Serverless 场景单独开发的技术栈，以 `@midwayjs/faas` 为代表的模块，使用轻量的方式接入不同的 Serverless 平台。

## 项目结构

常用的有：

- `controller` Web Controller 目录
- `middleware` 中间件目录
- `filter` 过滤器目录
- `aspect` 拦截器
- `service` 服务逻辑目录
- `entity` 或 `model` 数据库实体目录
- `config` 业务的配置目录
- `util` 工具类存放的目录
- `decorator` 自定义装饰器目录
- `interface.ts` 业务的 ts 定义文件

## 依赖注入

文档：https://midway.alibaba-inc.com/docs/container

在项目中能看到很多 `@Provide()`，`@Inject()` 这样的装饰器，Midway 是如何实现这些装饰器的呢？

我们先看看依赖注入是如何使用的，下面是一个例子

```ts
import { Provide, Inject } from '@midwayjs/core';

@Provide()               // <------ 暴露一个 Class
export class B {
  //...
}

@Provide()
export class A {

  @Inject()
  b: B;                  // <------ 这里的属性使用 Class

  //...
  @Inject('bTest')			// <------ 如果声明了参数，则会查找 BTest 类的实例 bTest 作为注入对象
  b: B
}
```

Midway 会自动使用 B 作为 b 这个属性的类型，在容器中实例化它。

为了保证实例的唯一性，Midway 会自动创建一个唯一的 uuid 关联这个 Class，同时这个 uuid 我们称为 **依赖注入标识符**。

默认情况：

- 1、 `@Provide` 会自动生成一个 uuid 作为依赖注入标识符
- 2、 `@Inject` 根据类型的 uuid 来查找

Midway 会默认 Provide 一些值，方便业务直接使用，比如框架的上下文 `ctx` ，可以直接注入使用。

```ts
@Provide()
export class BaseService {

  @Inject()
  ctx: any;

  async getUser() {
    console.log(this.ctx);
  }
}
```



#### 依赖注入原理

 Midway 体系启动阶段，会创建一个依赖注入容器（MidwayContainer），扫描所有用户代码（src）中的文件，将拥有 `@Provide` 装饰器的 Class，保存到容器中。

```typescript
/***** 下面为 Midway 内部代码 *****/

const container = new MidwayContainer();
container.bind(UserController);
container.bind(UserService);
```

这里的依赖注入容器类似于一个 Map。Map 的 key 是类对应的标识符（比如 **类名的驼峰形式字符串**），Value 则是 **类本身**。

![image-20231103114012019](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20231103114012019.png?x-oss-process=image/resize,w_600,m_lfit) 

在请求时，会动态实例化这些 Class，并且处理属性的赋值，比如下面的伪代码。

```typescript
/***** 下面为依赖注入容器伪代码 *****/
const userService = new UserService();
const userController = new UserController();

userController.userService = userService;
```

扩展阅读，编写一个 IOC 容器：https://mp.weixin.qq.com/s/g07BByYS6yD3QkLsA7zLYQ 

所有的 `@Provide` 出来的 Class 的作用域都为 **请求作用域**。这意味着这些 Class ，会在**每一次请求第一次调用时被实例化（new），请求结束后实例销毁**

## f.yml 配置文件

经过阿里集团标准化小组的讨论，结合社区 `serverless.yml` 的已有设计，通过一个描述文件 (**f.yml**) 来描述整个仓库中的函数信息，现有的发布构建工具，运行时都会基于此文件进行各种处理。

文档：https://midway.alibaba-inc.com/docs/serverless/serverless_yml#functions

主要是了解其中的  functions 配置，下面是一个示例：

```yaml
functions:
  hello1:
    handler: index.handler1
    events:
      - http:                   # http 触发器
          path: /foo
          method: get
```

这个 function 配置中

- handler: 事件的处理函数
- events: 事件的触发器
  - path：触发的路径
  - method：触发的请求方法

总的表示当收到 client 的带有 /foo 路径的 get 请求时，会触发 `index.handler1` 方法。

`index.handler1`方法又会在依赖中通过`@Provide()`暴露出来，比如下面的：

```ts
@Provide() // 暴露这个依赖
export class IndexHandler { // 依赖名不重要，主要是其中的实例方法
  @Inject()
  hsf?: any;

  @Handler('index.handler1') // 这里则要和 f.yml 中声明的处理方法对应起来
  getData() {
    this.ctx = 'Hello'
  }
}
```

