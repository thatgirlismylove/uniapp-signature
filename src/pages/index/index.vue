<template>
  <view class="content">
    <view class="canvas-container">
      <canvas @touchstart="handleTouch" @touchmove="handleMove" @touchend="handleEnd" class="my-canvas" type="2d"
              canvas-id="myCanvas" id="myCanvas"></canvas>
    </view>
    <view class="options">
      <view class="btn" @click="clear">重签</view>
      <view class="btn" @click="confirm">确认</view>
    </view>
  </view>
</template>

<script setup>
import {reactive, ref} from "vue";
import {onReady} from "@dcloudio/uni-app";

const isDrawing = ref(false); // 是否正在绘制
const ctx = ref(null) // 画布渲染上下文
// 记录最新一次绘制的坐标
const position = reactive({
  x: 0,
  y: 0
})

onReady(() => {
  const query = uni.createSelectorQuery()
  query.select('#myCanvas')
      .fields({node: true, size: true})
      .exec((res) => {

        // Canvas 对象
        const canvas = res[0].node;
        // 渲染上下文
        ctx.value = canvas.getContext('2d')

        // Canvas 画布的实际绘制宽高
        const width = res[0].width
        const height = res[0].height

        // 初始化画布大小
        const dpr = uni.getSystemInfoSync().pixelRatio

        canvas.width = width * dpr
        canvas.height = height * dpr
        ctx.value.scale(dpr, dpr)

        // 设置绘制属性
        ctx.value.strokeStyle = '#000';
        ctx.value.lineWidth = 2;

        drawBackground()
      })
})

const drawBackground = () => {
  // 创建白色矩形背景板
  ctx.value.fillStyle = "#fff"
  ctx.value.fillRect(0, 0, ctx.value.canvas.width, ctx.value.canvas.height)
}

// 开始绘制
const handleTouch = (e) => {
  isDrawing.value = true

  // 获取当前点的坐标
  const x = e.touches[0].x;
  const y = e.touches[0].y;

  position.x = x;
  position.y = y;
}

const handleMove = (e) => {
  if (!isDrawing.value) return;

  const x = e.touches[0].x;
  const y = e.touches[0].y;

  // beginPath() 新建一条路径，生成之后，图形绘制命令被指向到路径上生成路径。
  ctx.value.beginPath();
  // moveTo(x,y) 将笔触移动到指定的坐标 x 以及 y 上。
  ctx.value.moveTo(position.x, position.y);
  // lineTo(x, y) 绘制一条从当前位置到指定 x 以及 y 位置的直线。
  ctx.value.lineTo(x, y);
  // stroke() 使用描边通过线条来绘制当前路径,不需要填充闭合路径
  ctx.value.stroke();
  position.x = x;
  position.y = y;
}

const handleEnd = (e) => {
  isDrawing.value = false
}

// 清除签名
const clear = () => {
  ctx.value.clearRect(0, 0, ctx.value.canvas.width, ctx.value.canvas.height)
  drawBackground()
}

// 确认签名生成图片
const confirm = () => {
  // 获取 canvas 对象
  uni.createSelectorQuery()
      .select('#myCanvas')
      .node()
      .exec((res) => {
        // 获取 Canvas 对象
        const canvas = res[0].node;
        // 生成图片
        uni.canvasToTempFilePath({
          canvas,
          success: function (res) {
            // 在H5平台下，tempFilePath 为 base64
            console.log(res.tempFilePath)

            // 预览图片
            uni.previewImage({
              urls: [res.tempFilePath]
            })

          },
          fail: (res) => {
            console.log(res)
          }
        })
      })
}

const title = ref("Hello")
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.canvas-container {
  border: 1px solid #ccc;
  box-sizing: border-box;
}

.my-canvas {
  width: 100vw;
  height: 90vh;
  background-color: #fff;
}

.options {
  flex: 1;
  display: flex;
  justify-content: space-around;
  align-items: center;
  padding-bottom: env(safe-area-inset-bottom);
  box-sizing: border-box;
  padding-left: 40rpx;
  padding-right: 40rpx;
}

.btn {
  height: 60rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 120rpx;
  background: #4cd964;
  border-radius: 40rpx;
  color: #fff;
}
</style>
