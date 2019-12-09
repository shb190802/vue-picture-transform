<template>
  <div ref='container' class="center" :style="containerStyles">
    <div class="img-box" :style="boxStyles">
      <v-touch @pan="pictureMove" @panend="pictureMoveEnd">
        <div class="img" :style="imgStyles"></div>
      </v-touch> 
      <div class="close center icon">
        <img src="./images/icon_close.png" @click="close">
      </div>
      <div class="rotate center icon">
        <v-touch @panstart="pictureRotateStart" @panmove="pictureRotate" @panend="pictureRotateEnd">
          <img src="./images/icon_rotate.png">
        </v-touch>
      </div>
      <div class="scale center icon">
        <v-touch @panstart="pictureScaleStart" @panmove="pictureScale" @panend="pictureScaleEnd">
          <img src="./images/icon_scale.png">
        </v-touch>
      </div>
    </div>
  </div>
</template>
<script>
import VueTouch from 'vue-touch'
import Vue from 'vue'
const PADDING_DEG = 45
export default {
  name: 'Container',
  components: {
    'v-touch': Object.assign(VueTouch.component,{name: 'v-touch'}) // 局部引用
  },
  props: {
    size: {
      type: Number,
      default: 100
    },
    zIndex: {
      type: Number,
      default: 1
    },
    maxScale: {
      type: Number,
      default: 3
    },
    minScale:{
      type: Number,
      default: 0.2
    }
  },
  data(){
    return {
      id: '',// 容器的ID
      imgUrl: '',
      backgroundSize: '',
      // 父容器信息
      parent: {
        left: 0,
        top: 0,
        width: 0,
        height: 0
      },
      // 当前图片容器中心点位置
      position:{
        left: 0,
        top: 0
      },
      move: {
        x: 0,
        y: 0
      },
      scale: {
        sx: 0,
        sy: 0,
        scale: 1
      },
      rotate: {
        sx: 0,
        sy:0,
        deg: 0
      }
    }
  },
  computed:{
    // 外层容器样式，影响移动位置
    containerStyles(){
      return {
        left: this.position.left + this.move.x +  'px',
        top: this.position.top + this.move.y + 'px'
      }
    },
    // 内层容器位置，影响大小和旋转
    boxStyles() {
      return {
        width: this.size * this.scale.scale + 'px',
        height: this.size * this.scale.scale + 'px',
        zIndex: this.zIndex,
        transform: `rotate(${this.rotate.deg}deg)`
      }
    },
    // 图片容器，显示图片大小
    imgStyles() {
      return {
        width: this.size * this.scale.scale + 'px',
        height: this.size * this.scale.scale + 'px',
        zIndex: this.zIndex,
        backgroundImage: `url(${this.imgUrl})`,
        backgroundSize: this.backgroundSize
      }
    }
  },
  mounted(){
    // this.init({})
  },
  methods: {
    init({img,id}){
      this.id = id
      this.$nextTick(() => {
        let parent = this.$refs.container.parentNode
        if(getComputedStyle(parent).position === 'static') parent.style.position = 'relative'

        this.position.left = ~~(parent.offsetWidth / 2) 
        this.position.top = ~~(parent.offsetHeight / 2)
        
        this.parent.width = parent.offsetWidth
        this.parent.height = parent.offsetHeight
        do{
          this.parent.left += parent.offsetLeft
          this.parent.top += parent.offsetTop
          parent = parent.offsetParent
        }while(parent !== document.body)
        this.setImgUrl(img)
      })
    },
    close(){
      console.log('close')
      this.$emit('close',this.id)
    },
    setImgUrl(imgUrl){
      if(imgUrl === this.imgUrl) return 
      if(imgUrl) {
        this.imgUrl = imgUrl
        this.loadImg().then(img => {
          if(img.width > img.height) {
            this.backgroundSize = '100% auto'
          }else {
            this.backgroundSize = 'auto 100%'
          }
        })
      } else {
        throw new Error('Required param imgUrl!')
      }
    },
    loadImg(){
      return new Promise((resolve,reject) => {
        let img = new Image()
        img.onload = function(){
          resolve(this)
        }
        img.onerror = err=>{
          console.log('Load image Error!',err.message)
          reject(err)
        }
        img.src = this.imgUrl
      })
    },
    // 拖动移动
    pictureMove(e){
      this.move.x = e.deltaX
      this.move.y = e.deltaY
    },
    // 移动结束，更新当前位置，清空缓存位置
    pictureMoveEnd(e){
      this.position.left += e.deltaX
      this.position.top += e.deltaY
      this.move.x = 0
      this.move.y = 0
    },
    // 缩放
    pictureScaleStart(e){
      this.scale.sx = e.center.x - this.parent.left
      this.scale.sy = e.center.y - this.parent.top
    },
    pictureScale(e){
      this.scale.scale = this.getScale(e) 
    },
    pictureScaleEnd(e){
      this.scale.scale = this.getScale(e)
    },
    getScale(e){
      let x = e.center.x - this.parent.left - this.position.left
      let y = e.center.y - this.parent.top - this.position.top
      return Math.min(Math.max(Math.sqrt(x*x + y*y) / Math.sqrt(2*(this.size/2)*(this.size/2)),this.minScale),this.maxScale)
    },
    // 旋转
    pictureRotateStart(e){
      this.rotate.sx = e.center.x - this.parent.left
      this.rotate.sy = e.center.y - this.parent.top
    },
    pictureRotate(e){
      this.rotate.deg = this.getDeg(e)
    },
    pictureRotateEnd(e){
      this.rotate.deg = this.getDeg(e)
    },
    getDeg(e){
      let x = e.center.x - this.parent.left - this.position.left
      let y = e.center.y - this.parent.top - this.position.top
      return ~~(Math.atan2(y,x)*180/Math.PI) + PADDING_DEG
    },
    // 获取图片旋转信息
    getTransformInfo(){
      return {
        center: this.position,
        scale: this.scale.scale,
        rotate: this.rotate.deg,
        id: this.id
      }
    }
  }
}
</script>
<style scoped>
.center{
  position: absolute;
  width: 0;
  height: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1;
}
.img-box{
  position: absolute;
  background-position: center;
}
.img-box::before {
  position: absolute;
  left: -50%;
  top: -50%;
  transform: scale(.5);
  width: 200%;
  height: 200%;
  border: solid 1px #ccc;
  content: '';
  z-index: -1;
}
.img-box .close{
  left: 0;
  top: 0;
}
.img-box .rotate{
  right: 0;
  top: 0;
}
.img-box .scale{
  right: 0;
  bottom: 0;
}
.img-box .img {
  width: 100%;
  height: 100%;
}
.img-box .icon {
  transform: scale(.12);
}
</style>