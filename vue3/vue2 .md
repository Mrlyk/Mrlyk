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

```vue
<!--**********子组件传 node、data 给父组件***********-->
<template #default='{ node, data }' v-if='$scopedSlots.default'>
  <slot :slotProps='{ node, data }'></slot>
</template>


<!--***********父组件使用子组件传过来的 slotProps *****************-->
<template #default='{ slotProps }'>
  {{ slotProps.node.label + '1' }}
</template>
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







