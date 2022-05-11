<template>
  <div class="turntable-wrapper" ref="turntableDom" :style="{ width, height }">
    <div class="turntable" :style="{ transform: rotateAngle, transition: rotateTransition }">
      <canvas id="canvas" ref="canvasDom"> 浏览器版本过低 </canvas>
      <div class="prizes">
        <div class="prize-item" v-for="(item, index) in prizeList" :style="getRotateAngle(index)" :key="index">
          <div class="prize-item-name">{{ item.name }}</div>
          <div class="prize-item-img">
            <img v-if="item.img" :src="item.img" alt="" />
          </div>
        </div>
      </div>
    </div>
    <div class="pointer">
      <slot name="pointer" :start="start">
        <div class="pointer-default" :style="{ cursor: turning ? 'not-allowed' : 'pointer' }" @click="start">
          <div>点击</div>
          <div>抽奖</div>
        </div>
      </slot>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'VueTurntable',
    props: {
      width: {
        // required: true,
        default: '300px'
      },
      height: {
        // required: true,
        default: '300px'
      },
      turnsTime: {
        default: 5
      },
      prizeList: {
        required: true,
        type: Array
      },
      prizeIndex: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        turning: false,
        startRotateDegree: 0, // 开始角度
        rotateAngle: 0,
        rotateTransition: '',
        turnsNumber: 5 // 转动圈数
      }
    },
    created() {},
    mounted() {
      this.$nextTick(this.init)
      window.start = this.start
    },
    watch: {
      prizeList: 'init'
    },
    methods: {
      init() {
        const prizeNum = this.prizeList.length
        const borderColor = '#ff9800'

        const luckdraw = this.$refs.turntableDom
        const canvas = this.$refs.canvasDom
        const ctx = canvas.getContext('2d')

        const canvasW = (canvas.width = luckdraw.clientWidth) // 画板的高度
        const canvasH = (canvas.height = luckdraw.clientHeight) // 画板的宽度
        // translate方法重新映射画布上的 (0,0) 位置
        ctx.translate(0, canvasH)
        // rotate方法旋转当前的绘图，因为文字是和当前扇形中心线垂直的
        ctx.rotate((-90 * Math.PI) / 180)
        // 圆环的外圆的半径,可用来调整圆盘大小来适应外部盒子的大小
        const outRadius = canvasW / 2 - 1
        // 圆环的内圆的半径
        const innerRadius = 0
        const baseAngle = (Math.PI * 2) / prizeNum // 每个奖项所占角度数
        ctx.clearRect(0, 0, canvasW, canvasH) //去掉背景默认色
        ctx.strokeStyle = borderColor // 设置画图线的颜色
        for (let index = 0; index < prizeNum; index++) {
          const angle = index * baseAngle
          if (this.prizeList[index]['color']) {
            ctx.fillStyle = this.prizeList[index]['color'] //设置每个扇形区域的颜色,根据每条数据中单独设置的优先
          } else {
            ctx.fillStyle = index % 2 ? 'rgb(255, 231, 149)' : 'rgb(255, 247, 223)'
          }
          ctx.beginPath() //开始绘制
          // 标准圆弧：arc(x,y,radius,startAngle,endAngle,anticlockwise)
          ctx.arc(canvasW * 0.5, canvasH * 0.5, outRadius, angle, angle + baseAngle, false)
          ctx.arc(canvasW * 0.5, canvasH * 0.5, innerRadius, angle + baseAngle, angle, true)
          ctx.stroke()
          ctx.fill()
          ctx.save()
        }
      },
      start() {
        if (this.turning) return

        this.turning = true
        this.$emit('start')
        this.rotate(this.prizeIndex)
      },
      rotate(index) {
        const turnsTimeNum = +this.turnsTime
        const rotateAngleValue =
          this.startRotateDegree + // 初始角度
          this.turnsNumber * 360 + // 转的圈数*360
          360 - // 最后一圈
          (180 / this.prizeList.length + (360 / this.prizeList.length) * index) - // 对应奖项中间位置的角度
          (this.startRotateDegree % 360) // 当前已超出初始位置的角度
        this.startRotateDegree = rotateAngleValue
        this.rotateAngle = `rotate(${rotateAngleValue}deg)`
        this.rotateTransition = `transform ${turnsTimeNum}s cubic-bezier(0.250, 0.460, 0.455, 0.995)`
        setTimeout(() => {
          this.$emit('end')
          this.turning = false
        }, turnsTimeNum * 1000 + 500)
      },
      // 根据index计算每一格要旋转的角度的样式
      getRotateAngle(index) {
        const angle = (360 / this.prizeList.length) * index + 180 / this.prizeList.length
        return {
          transform: `rotate(${angle}deg)`
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  .turntable-wrapper {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
    overflow: hidden;
    .turntable {
      position: absolute;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      #canvasWx {
        width: 200%;
        height: 100%;
      }
      .mlcanvas {
        margin-left: -50%;
      }
    }
    .prizes {
      position: absolute;
      left: 25%;
      top: 0;
      width: 50%;
      height: 50%;
      .prize-item {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        transform-origin: center bottom;
        &-name {
          position: absolute;
          left: 10px;
          top: 20px;
          width: calc(100% - 20px);
          font-size: 12px;
          text-align: center;
          color: #ff5722;
        }
        &-img {
          position: absolute;
          /*要居中就要50% - 宽度 / 2*/
          left: calc(50% - 30px / 2);
          top: 60px;
          width: 30px;
          height: 30px;
          img {
            display: inline-block;
            width: 100%;
            height: 100%;
          }
        }
      }
    }

    .pointer {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
    .pointer-default {
      cursor: pointer;
      width: 80px;
      height: 80px;
      border-radius: 50%;
      background: red;
      border: 4px solid #fff;
      box-sizing: border-box;
      font-size: 16px;
      font-weight: 600;
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      position: relative;
      &::after {
        content: '';
        position: absolute;
        z-index: -1;
        top: -30px;
        width: 0;
        height: 0;
        border-left: 10px solid transparent;
        border-right: 10px solid transparent;
        border-bottom: 30px solid red;
      }
    }
  }
</style>
