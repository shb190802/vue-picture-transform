

# vue-picture-transform

本组件为VUE移动端，图片自由变换组件，暂时仅支持移动端。

![1](./images/effect/1.png)

#### 安装

```
npm i -S vue-picture-transform
```

| 属性       | 类型   | 默认值 | 说明                                           |
| ---------- | ------ | ------ | ---------------------------------------------- |
| size       | Number | 100    | 操作框大小默认100*100                          |
| zIndex     | Number | 1      | 操作框zIndex                                   |
| maxScale   | Number | 3      | 最大放大倍率                                   |
| minScale   | Number | 0.2    | 最小缩放倍率                                   |
| disabled   | Number | false  | 禁用操作框                                     |
| imgUrl | String |        | 初始化图片地址（可不传，之后调用init方法来传） |
| id     | String |        | 初始化ID（可不传，之后调用init方法来传）       |



| 事件     | 说明             | 参数 | 参数说明             |
| -------- | ---------------- | ---- | -------------------- |
| close    | 点击叉号关闭事件 | id   | 初始化传进来的initId |
| selected | 选中事件         | id   | 初始化传进来的initId |



| 组件方法           | 说明                   | 返回值      |
| ------------------ | ---------------------- | ----------- |
| init(imgUrl,id)    | 设置一张图片和ID       | 空          |
| getTransformInfo() | 获取变变形后的图片信息 | {center:{x,y},width,height,scale,rotate,id,imgUrl,img} |



#### 单个组件使用方式：

```javascrpt
<template>
  <div id="app">
    <VuePictureTransform ref="transform"></VuePictureTransform>
    <button @click="getInfo">getInfo</button>
  </div>
</template>
<script>
import VuePictureTransform from 'vue-picture-transform'
import imgUrl from './assets/logo.png'
export default {
  name: 'app',
  components: {
    VuePictureTransform,
  },
  mounted(){
    this.$refs.transform.init({
      imgUrl: imgUrl
    })
  },
  methods: {
    getInfo(){
      console.log(this.$refs.transform.getTransformInfo())
    }
  }
}
</script>
```



#### 多个组件使用，并拼接到canvas上

```javascript
<template>
      <div ref='avatar' :style="{backgroundImage: 'url('+avatar+')'}">
        <PicTrancform
          ref="decorateList"
          v-for="item in useDecorateList"
          :key="item.id"
          :disabled="!(item.id === active)"
          :initImgUrl="item.img"
          :initId="item.id"
          :zIndex="item.zIndex"
          @close="close(item.id)"
          @selected="selected(item.id)"
        />
      </div>
</template>
<script>
import PicTrancform from 'vue-picture-transform'

export default {
  components: {
    PicTrancform
  },
  data(){
    return {
      useDecorateList: [],// 图片列表
      active:0, //当前操作图片
      avatar: '',// 背景图片地址
      avatarImg: null,// 背景图片imgData
      canvas: null,//画布
      ctx: null,// 上下文
      canvasInfo:{ 
        width: 240,// 画布宽度
        height: 240,// 画布高度
        scale: 2 // 生成图片放大比例
      },
    }
  },
  mounted(){
    this.canvas = this.$refs.canvas // 设置一个canvas元素。
    this.ctx = this.canvas.getContext('2d')
  },
  methods: {
    ...
    getDecorateList(){
      let list = []
      let components = this.$refs.decorateList
      if(Array.isArray(components)) {
        list = components.map(item => {
          return item.getTransformInfo()
        })
      }
      return list
    },
    mergeImg(){
      let canvas = this.canvas
      let ctx = this.ctx
      let {width,height,scale} = this.canvasInfo
      ctx.clearRect(0,0,width*scale,height*scale)
      let avatarImg = this.avatarImg
      ctx.drawImage(avatarImg,0,0,width*scale,height*scale)
      let decorateList = this.getDecorateList()
      decorateList.forEach(decorate => {
        ctx.save() // 保存当前状态
        ctx.translate(decorate.center.x*scale,decorate.center.y*scale)
        ctx.rotate(decorate.rotate*Math.PI/180)
        ctx.scale(decorate.scale,decorate.scale)
        ctx.drawImage(decorate.img,-1* ~~(decorate.width*scale/2),-1* ~~(decorate.height*scale/2),decorate.width*scale,decorate.height*scale)
        ctx.restore()
      })
      // 使用toDataUrl可以获取拼接图片
    }
}
</script>
```

