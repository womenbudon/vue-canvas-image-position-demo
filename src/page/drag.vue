<template>
  <div class="wraper">
    <canvas id="canvas-contianer" v-bind:width="canvasConfig.width"  v-bind:height="canvasConfig.height">
    </canvas >
  </div>
</template>

<script>

// 初始化全局变量
let canvas = null, 
ctx = null,
imageEle=null,
timeOutEvent =0;
// 图像变量
let imageConfig = {
  x: 0,
  y: 0,
  width: 0,
  height: 0,
  scale: 1,
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
    this.canvasEventsInit();
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
        // 计算缩放比例 根据图片宽高和容器宽高计算
        const scale = this.computedScale();
        ctx.clearRect(0, 0, this.canvasConfig.width, this.canvasConfig.height); // 擦除之前绘制
        ctx.drawImage(
          imageEle,
          x, // 图像位置x
          y, // 图像位置y
          imageEle.width*scale, // 图片真实宽度
          imageEle.height*scale// 图片真实高度
        );
        imageConfig = {
          ...imageConfig,
          width: imageEle.width*scale,
          height: imageEle.height*scale,
        }
    };
    imageEle.src ="https://103.36.30.143:32443/iotapp/fs/downloadFile/iotMapManage/394371/1558318762521.PNG";
    },
    computedScale() {
      const canvasWdith = this.canvasConfig.width,
      canvasHeight = this.canvasConfig.height,
      imageWidth = imageEle.width,
      imageHeight = imageEle.height;
      let scale = 1; // 默认缩放比例为1
      scale = (canvasWdith/imageWidth <= canvasHeight/imageHeight) ? canvasWdith/imageWidth : canvasHeight/imageHeight;
      if(scale > 1){
        scale = 1;
      }
      return scale;
    },
    isInImageArea(axVal, yxVal) {
      // 判断是否在图片区域内
      const { x, y, width, height } = imageConfig;
      const rect = new Path2D();
      rect.rect(x, y, width, height);
      return ctx.isPointInPath(rect,axVal,yxVal);
    },
    longPress() {
      timeOutEvent = 0;
    },
    drag(dx, dy) {
      const _self = this;
      canvas.onmousemove = function (ev) {  //移动移动
        var e = ev||event;
        const ax = e.layerX; // 鼠标距离原点偏移
        const ay = e.layerY;
        const moveX = ax - dx ; // 获取x轴移动距离
        const moveY = ay - dy ; // 获取y轴移动距离

        clearTimeout(timeOutEvent);
        timeOutEvent = 0;

        //鼠标移动每一帧都清楚画布内容，然后重新画圆
        ctx.clearRect(0, 0,
         _self.canvasConfig.width,
        _self.canvasConfig.height); // 擦除之前绘制
        
        // 重新绘制
        ctx.drawImage(
          imageEle,
          imageConfig.x + moveX, // 图像位置x
          imageConfig.y + moveY, // 图像位置y
          imageConfig.width, // 图片真实宽度
          imageConfig.height // 图片真实高度
        );
      };
      canvas.onmouseup = function (ev) { // 鼠标移开
         var e = ev||event;
        const ax = e.layerX;
        const ay = e.layerY;
        const moveX = ax - dx; // 获取x轴移动距离
        const moveY = ay - dy; // 获取y轴移动距离

          canvas.onmousemove = null;
          canvas.onmouseup = null;
          canvas.style.cursor = 'default';
          // 更新图片的 x y 
          imageConfig = {
            ...imageConfig,
            x: imageConfig.x + moveX,
            y: imageConfig.y + moveY,
          } 
      };
    },
    canvasEventsInit() {
      // 拖动
      const _self = this;
      canvas.onmousedown = (ev) =>{
         ev = ev || window.event;  
        const x = ev.layerX;
        const y = ev.layerY;
        if(!_self.isInImageArea(x,y)) {
          return;
        }
        canvas.style.cursor = 'pointer';
        timeOutEvent = setTimeout(this.longPress,500);
        ev.preventDefault();
        this.drag(x,y);
      }
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