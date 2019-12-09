# vue-picture-transform
Vue移动端，图片自由变换组件

使用方式：
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
      img: imgUrl
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

![1](./images/effect/1.png)