

# vue2 

> 核心与部分易忘知识点记录

[toc]

## 插槽

#### 插槽内容

```vue
/*============test.vue============*/
<template>
  <div>
    <slot name="header"></slot>
    <slot></slot>
    <slot name="footer"></slot>
  </div>
</template>

<script>
export default {
  mounted () {
    console.log(this.$slots, '$slots')
    console.log(this.$scopedSlots, '$scopedSlots')
  }
}
</script>
/*============parent.vue=======*/
<template>
  <test>
    <div slot="header">slot: header</div>
    <div slot-scope="scope">slot: {{scope}}</div>
    <div slot="footer">slot: footer</div>
  </test>
</template>

<script>
import Test from './test.vue'
export default {
  components: {
    Test
  }
}
</script>
```

**$slots** 

![image-20220228193216602](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220228193216602.png?x-oss-process=image/resize,w_400,m_lfit)  

#### 作用域插槽

直接看两个例子

```vue
// slot-scope 弃用写法
<el-table-column label="状态" prop="mg_state">
  <template slot-scope="scope">
		<el-switch v-model="scope.row.mg_state">
  	</el-switch>
  </template>
</el-table-column>

// v-slot写法
<template v-slot:default="scope">
	<el-switch v-model="scope.row.mg_state">
  </el-switch>
</template>
```

```vue
<!--**********子组件传 node、data 给父组件***********-->
<template #default='{ node, data }' v-if='$scopedSlots.default'>
  <slot :slotProps='{ node, data }'></slot>
</template>


<!--***********父组件使用子组件传过来的 slotProps *****************-->
<template #default='{ slotProps }'> // 或者 <template v-slot:default='{slotProps}'></template>
  {{ slotProps.node.label + '1' }}
</template>

<!--如果是具名作用域插槽，将 default 改为名字 --》

```

## 部分 API

#### vm.$attrs & vm.$listeners

创建高阶组件时可以使用 

- $atts： 顾名思义是属性

传递给孙组件的，父组件中未在 props 中显示接收的数据，如下

```vue
// grand-father.vue
<script>
  import Father from 'father'
  export default {
    components: {
      Father
    },
    data() {
      return {
        a: '1',
        b: '2',
        c: '3'
      }
    }
  }
</script>
<template>
	<father :a="a" :b="b" c="c"></father>
</template>

/*====================================*/
// father.vue
<script>
  import Son from 'son'
  export default {
    props: {
      a: String
    }
    components: {
      Son
    }
  }
</script>
<template>
	<son v-bind="$attrs"></son>
</template>

/*====================================*/
// son.vue
<script>
  export default {
    props: {
      a: String
    }
    components: {
      Son
    }
  }
</script>
<template>
<!-- 可以通过 $attrs 接收到 father 未接首的数据 -->
	<div>
    {{ $attrs.b }} 
    {{ $attrs.c }}
  </div>
</template>
```



#### provide、inject 响应式传值

`provide` 刻意被设置为不可响应的，他的传值方式是直接传递一个浅拷贝的复制，所以如果是普通类型的数据无法实时获取。

如果想要实时获取最新值有以下几个常用方法：

1. 将要传递的普通值通过一个对象包裹，在值变更时通过`this._provided.object.xxx = xxx` 来赋值。因为是浅拷贝，所以这样能获取到最新的值。
2. 传递一个函数而不是数据对象，在要获取最新值时调用这个函数（推荐的做法）
3. 传递整个 vue 实例（`this`），这个和第一个的原理类似，不过不需要实时赋值了，但不推荐！
