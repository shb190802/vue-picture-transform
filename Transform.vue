<template>
  <div ref='container' class="center" :style="containerStyles">
    <div class="img-box" :class="{active:!disabled}" :style="boxStyles">
      <v-touch
        :panOptions="{threshold:2}"
        @panstart="pictureMove"
        @panmove="pictureMove"
        @panend="pictureMoveEnd"
        @tap="doSelected">
        <div class="img" :style="imgStyles"></div>
      </v-touch> 
      <div v-if="!disabled" class="close center icon">
        <v-touch @tap="close">
          <span class="img-icon close"></span>
          <!-- <img src="./images/icon_close.png"> -->
        </v-touch>
      </div>
      <div v-if="!disabled"  class="rotate center icon">
        <v-touch
          :panOptions="{threshold:2}"
          @panstart="pictureRotateStart"
          @panmove="pictureRotate"
          @panend="pictureRotateEnd">
          <span class="img-icon rotate"></span>
          <!-- <img src="./images/icon_rotate.png"> -->
        </v-touch>
      </div>
      <div v-if="!disabled"  class="scale center icon">
        <v-touch
          :panOptions="{threshold:2}"
          @panstart="pictureScaleStart"
          @panmove="pictureScale"
          @panend="pictureScaleEnd">
          <span class="img-icon scale"></span>
          <!-- <img src="./images/icon_scale.png"> -->
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
    },
    disabled: {
      type: Boolean,
      default: false
    },
    imgUrl: {
      type: String,
      default: ''
    },
    id: {
      type: String | Number,
      default: ''
    }
  },
  data(){
    return {
      useId: '',// 容器的ID
      useImgUrl: '',
      img: null,
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
        top: 0,
        width: 100,
        height: 100
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
        top: this.position.top + this.move.y + 'px',
        zIndex: this.zIndex
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
        backgroundImage: `url(${this.useImgUrl})`,
        backgroundSize: this.backgroundSize
      }
    }
  },
  mounted(){
    if(this.imgUrl) {
      this.init({imgUrl: this.imgUrl,id: this.id})
    }
  },
  methods: {
    init({imgUrl,id}){
      this.useId = id
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
        this.setImgUrl(imgUrl)
      })
    },
    close(){
      this.$emit('close',this.useId)
    },
    doSelected(){
      this.$emit('selected',this.useId)
    },
    setImgUrl(imgUrl){
      if(imgUrl === this.useImgUrl) return 
      this.img = null
      if(imgUrl) {
        this.useImgUrl = imgUrl
        this.loadImg().then(img => {
          this.img = img
          if(img.width > img.height) {
            this.backgroundSize = '100% auto'
            this.position.width = 100
            this.position.height = Math.round(100*img.height / img.width)
          }else {
            this.backgroundSize = 'auto 100%'
            this.position.height = 100
            this.position.width = Math.round(100*img.width / img.height)
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
        img.src = this.useImgUrl
      })
    },
    // 拖动移动
    pictureMove(e){
      if(this.disabled) return
      this.move.x = e.deltaX
      this.move.y = e.deltaY
    },
    // 移动结束，更新当前位置，清空缓存位置
    pictureMoveEnd(e){
      if(this.disabled) return
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
      let docScrollTop = document.documentElement.scrollTop || document.body.scrollTop
      let x = e.center.x - this.parent.left - this.position.left
      let y = e.center.y - this.parent.top - this.position.top + docScrollTop
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
      let docScrollTop = document.documentElement.scrollTop || document.body.scrollTop
      let x = e.center.x - this.parent.left - this.position.left
      let y = e.center.y - this.parent.top - this.position.top + docScrollTop
      return ~~(Math.atan2(y,x)*180/Math.PI) + PADDING_DEG
    },
    // 获取图片旋转信息
    getTransformInfo(){
      return {
        center:{
          x: this.position.left,
          y: this.position.top
        },
        width: this.position.width,
        height: this.position.height,
        scale: this.scale.scale,
        rotate: this.rotate.deg,
        id: this.useId,
        imgUrl: this.useImgUrl,
        img: this.img
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
.active::before {
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
  background-repeat: no-repeat;
  background-position: center;
}
.img-box .icon {
  /* transform: scale(.15); */
}
.img-icon {
  position: relative;
  display: inline-block;
  width: 19px;
  height: 19px;
  background-size: 100% 100%;
  background-repeat: no-repeat;
}
.img-icon.close{
  background-image: url('./images/icon_close.png');
}
.img-icon.rotate{
  background-image: url('./images/icon_rotate.png');
}
.img-icon.scale{
  background-image: url('./images/icon_scale.png');
}
</style>