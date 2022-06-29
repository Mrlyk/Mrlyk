# 数据可视化_ECharts  
[toc]


### 一、ECharts 基本概念：系列（series）
> 基本概念：一组数值映射成对应的图  接收一个对象或一个数组，单个对象时只绘制单个图表，数组时绘制混合图表。   

```js
// series示例
series: [{
  type: 'pie',
  center: [0, 0],
  radius: 35,
  data: [
    {name: '分类1', value: 50},
    {name: '分类2', value: 60},
    {name: '分类3', value: 55},
    {name: '分类4', value: 70}
  ]}
        ]
```

### 二、ECharts4.0新特性：dataset  
> 为了方便统一的数据管理，通过dataset传入所有数据，方便更新和渲染  

#### dataset的数据是如何对应的  
```js
const optioons = {
  dataset: {
    dimensions: [ 'label', 'value1', 'value2' ], // 对应 this.dataList 的 key，比如 [{label: '2015', value1: 10, value2: 20}]
    source: this.dataList
  },
  series: [{ // 默认按列顺序对应，也可以 encode 手动声明 类目轴（category）。默认情况下，类目轴对应到 dataset 第一列。
    type: 'pie',
    center: ['65%', 60],
    radius: 35,
    encode: {itemName: 3, value: 4}
  },{
    type: 'line',
    encode: {x: 0, value: 1}
  },{
    type: 'bar',
    encode: {x: 0, value: 2}
  }]
}
```
![dataset数据对应](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/dataset%E6%95%B0%E6%8D%AE%E5%AF%B9%E5%BA%94.png?x-oss-process=image/resize,w_600,m_lfit) 

如上图所示：  
**encode: {行， 列}**  
encode参数用来表示数值对应，第一个参数表示数据的第几项作为x轴，第二个参数表示数据的第几项作为y轴。上方的pie形图，表示第三列作为作为x轴，第四列作为y轴。折线图和柱状图同理。 
*注意的是pie状图的横坐标不是x而是itemName*

### 三、组件  
> ECharts中除了绘图之外其他部分，都可抽象为【组件】。例如，ECharts中至少有这些组件：xAxis（直角坐标系x轴）、yAxis（直角坐标系y轴）、grid（直角坐标系底板）、angleAxis（极坐标系角度轴） 

图示：  

 ![组件](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/%E7%BB%84%E4%BB%B6.png?x-oss-process=image/resize,w_800,m_lfit) 
