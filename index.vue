<template>
  <view
    class="cascade-swiper"
    @touchstart="handleTouchStart"
    @touchmove="handleTouchMove"
    @touchend="handleTouchEnd"
  >
    <view
      class="item item-pre"
      :style="'background:' + prevItem.color + ';' + preElementStyle + animationStyle"
    >{{prevItem.text}}</view>
    <view
      class="item item-cur"
      :style="'background:' + curItem.color + ';' + curElementStyle  + animationStyle"
    >{{curItem.text}}</view>
    <view
      class="item item-next"
      :style="'background:' + nextItem.color + ';' + nextElementStyle  + animationStyle"
    >{{nextItem.text}}</view>
    <view
      class="item item-after-next"
      :style="'background:' + afterNextItem.color + ';' + afterNextElementStyle  + animationStyle"
    >{{afterNextItem.text}}</view>
  </view>
</template>

<script>

const safeRange = (val, start, end) => {
  if (val < start) {
    return start
  } else if (val > end) {
    return end
  } else {
    return val
  }
}

export default {
  name: '',
  data () {
    return {
      cardList: [
        { color: 'red', text: 'A' },
        { color: 'green', text: 'B' },
        { color: 'blue', text: 'C' },
        { color: 'cyan', text: 'D' },
      ],
      currentIndex: 0,
      slidePercent: 0, // 滑动进度 从 -1(代表前一张) 到 1(代表后一张)
      onAnimation: false,
      duration: 0.3
    }
  },
  computed: {
    prevItem () {
      return this.cardList[this.getPrevIndex(this.currentIndex)]
    },
    curItem () {
      return this.cardList[this.currentIndex]
    },
    nextItem () {
      return this.cardList[this.getNextIndex(this.currentIndex)]
    },
    // 下下个元素
    afterNextItem () {
      return this.cardList[this.getNextIndex(this.getNextIndex(this.currentIndex))]
    },
    // 通过滑动进度计算元素的样式，缩放偏移
    nextElementStyle () {
      // 当进度处于偏向后一张时，越接近1 则缩放从0.95趋向1 向上偏移从-25%趋向0
      if (this.slidePercent >= 0) {
        const scale = 0.95 + (0.05 * this.slidePercent)
        const Y = -25 + this.slidePercent * 25
        return `transform:scale(${scale}) translateY(${Y}%);`
      } else {
        // 当进度处于偏向前一张时，越接近-1 则缩放从0.95趋向0.9
        const percent = -this.slidePercent
        const scale = 0.95 - (0.05 * percent)
        return `transform:scale(${scale}) translateY(-25%);`
      }
    },
    afterNextElementStyle () {
      // 当进度偏向后一张事，越接近1，则缩放从0.9偏向0.95
      if (this.slidePercent >= 0) {
        const scale = 0.9 + (0.05 * this.slidePercent)
        return `transform:scale(${scale}) translateY(-25%);`
      } else {
        // 偏向前一张时，直接不可见
        return 'transform:scale(0);'
      }
    },
    curElementStyle () {
      // 当进度处于偏向后一张时，越接近1 向下偏移逐渐接近100%
      if (this.slidePercent >= 0) {
        const top = this.slidePercent * 100
        return `transform:translateY(${top}%);`
      } else {
        // 当进度处于偏向前一张时，越接近-1 则缩放从1趋向0.95 向上偏移从0趋向25%
        const percent = -this.slidePercent
        const scale = 1 - (0.05 * percent)
        const Y = percent * -25
        return `transform:scale(${scale}) translateY(${Y}%);`
      }
    },
    preElementStyle () {
      // 偏向后一张时，完全不显示
      if (this.slidePercent >= 0) {
        return `transform:translateY(100%);`
      } else {
        // 当进度处于偏向前一张时，越接近-1 向上偏移从100%趋向0
        const percent = -this.slidePercent
        const Y = 100 - percent * 100
        return `transform:translateY(${Y}%);`
      }
    },
    animationStyle () {
      if (this.onAnimation) {
        return `transition: all ${this.duration}s`
      } else {
        return ''
      }
    }
  },
  mounted () {
    setTimeout(() => {
      const query = this.createSelectorQuery()
      query.select('.item-cur').boundingClientRect((res) => {
        this.standItemHeight = res.height
      })
      query.exec()
    })
  },
  methods: {
    // 获取这个索引的前一个项
    getPrevIndex (index) {
      return --index < 0 ? this.cardList.length - 1 : index
    },
    // 获取这个索引的后一个项目
    getNextIndex (index) {
      return ++index === this.cardList.length ? 0 : index
    },
    handleTouchStart (e) {
      if (this.onMove || this.onAnimation) return
      this.onMove = true
      this.touchStartY = e.touches[0].pageY
      this.slidePercent = 0
    },
    handleTouchMove (e) {
      if (!this.onMove || this.onAnimation) return
      const currentY = e.touches[0].pageY
      const diff = currentY - this.touchStartY
      let newPercent = safeRange(diff / this.standItemHeight, -1, 1)
      this.slidePercent = newPercent
    },
    handleTouchEnd (e) {
      if (!this.onMove || this.onAnimation) return
      this.onMove = false
      this.startSlideAnimation()
    },
    // 启动结束动画
    startSlideAnimation () {
      const currentPercent = this.slidePercent
      let nextIndex = this.currentIndex

      this.onAnimation = true
      // 前一张
      if (currentPercent < -0.4) {
        this.slidePercent = -1
        nextIndex = this.getPrevIndex(this.currentIndex)
      } else if (currentPercent > 0.4) {
        // 后一张
        this.slidePercent = 1
        nextIndex = this.getNextIndex(this.currentIndex)
      } else {
        // 还原
        this.slidePercent = 0
      }

      setTimeout(() => {
        this.onAnimation = false
        this.currentIndex = nextIndex
        this.slidePercent = 0
      }, this.duration * 1000)
    }
  }
}
</script>

<style lang="scss" scoped>
.cascade-swiper {
  width: 700rpx;
  margin: 0 auto;
  height: 200rpx;
  position: relative;
  overflow: hidden;
  margin-top: 100rpx;
  background-color: #eee;
  .item {
    width: 100%;
    height: 160rpx;
    background-color: #999;
    color: #fff;
    font-size: 60rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    position: absolute;
    left: 0rpx;
    bottom: 0rpx;
    border-radius: 10rpx;
    &-pre {
      z-index: 30;
    }
    &-cur {
      z-index: 20;
    }
    &-next {
      z-index: 10;
      transform: scale(0.95) translateY(-40rpx);
    }
    &-after-next {
      z-index: 1;
    }
  }
}
</style>
