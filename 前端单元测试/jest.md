

# jest

配合 ESLint 使用时需要在`env`中打开`jest: true` 配置

### 安装

`npm i jest -D`

## 使用

需要使用`jest.config.js` 或 `jest.config.json`或放在 package.json 中配置一些参数

`npx jest`

### config 重要参数说明

```js
{
	moduleNameMapper: { // 路径映射，在配置了 webpack 别名时有用
  	'^@/(.*)$': '<rootDir>/src/$1'
  },
  testEnvironment: 'jsdom', // 测试环境，默认 node 环境。使用该参数配置为浏览器环境
  testPathIgnorePatterns: ['/node_modules/']， // 忽略的文件
}
```



### jest 测试脚本语法

- describe 分组

- test || it 测试

  ```js
  test('设置 cookie', () => {
      expect(cookieManagerLocal.set('cookie1', 'test1')).toBeUndefined()
    }) // 第一个参数是 name，第二个参数是测试函数。 toBeXXXX 是期望返回值
  
  test('设置 cookie', () => {
      expect(cookieManagerLocal.get('cookie1')).toBe('test1')
    })
  ```

- 异步脚本
