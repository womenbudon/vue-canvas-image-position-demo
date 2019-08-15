<template>
  <div class="wraper">
    <vue-draggable-resizable :w="100" :h="100" @dragging="onDrag" @resizing="onResize" :parent="true">
        <canvas id="canvas-contianer" v-bind:width="canvasConfig.width"  v-bind:height="canvasConfig.height">
        </canvas >
    </vue-draggable-resizable>
  </div>
</template>

<script>
import Vue from 'vue'
import VueDraggableResizable from 'vue-draggable-resizable'

// optionally import default styles
import 'vue-draggable-resizable/dist/VueDraggableResizable.css'

Vue.component('vue-draggable-resizable', VueDraggableResizable)

// 初始化全局变量
let canvas = null, 
ctx = null,
imageEle=null;
// 图像变量
let imageConfig = {
  x: 0,
  y: 0,
  width: 0,
  height: 0,
}

// 获取容器宽高
const getContainerSize = () => {
  const canvas = document.getElementById('canvas-contianer')
  return { containerHeight: canvas.clientHeight, containerWidth: canvas.clientWidth };
}

export default {
    data() {
        return {
          canvasConfig: {
            height: 0,
            width: 0,
          }
        };
    },
   created() {
  },
  mounted() {
    this.init();
    this.setCanvasSize(); 
    this.initImage();
  },
  methods:  {
    init() {
      canvas = document.getElementById('canvas-contianer');
      ctx = canvas.getContext('2d');
    },
    setCanvasSize() { 
      const { containerHeight,containerWidth } = getContainerSize();
      this.canvasConfig.height = containerHeight;
      this.canvasConfig.width = containerWidth;
    },
    initImage() {
      const { x, y }= imageConfig;
      imageEle = new window.Image();
      imageEle.onload = () => {
        ctx.clearRect(0, 0, this.canvasConfig.width, this.canvasConfig.height); // 擦除之前绘制
        ctx.drawImage(
          imageEle,
          x, // 图像位置x
          y, // 图像位置y
          imageEle.width, // 图片真实宽度
          imageEle.height // 图片真实高度
        );
        imageConfig = {
          ...imageConfig,
          width: imageEle.width,
          height: imageEle.height,
        }
    };
    imageEle.src ="https://konvajs.org/assets/yoda.jpg";
    },
    onResize: function (x, y, width, height) {
      this.x = x
      this.y = y
      this.width = width
      this.height = height
    },
    onDrag: function (x, y) {
      this.x = x
      this.y = y
    }
  }
};
</script>

<style lang="less" scoped>
    @import '../style/mixin';
    .wraper{
      height: 100%;
      width: 100%;
      box-sizing: border-box;
      padding: 20px;
    }
    #canvas-contianer{
        height: 100%;
        width: 100%;
        background: #ccc;
        overflow: hidden;
        box-sizing: border-box;
    }
</style>