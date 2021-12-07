# vue-cascade-swiper
层叠滚动轮播卡片，类微信朋友圈层叠卡片效果

演示视频

<video id="video" controls="" preload="none"  style="width:100%">
      <source id="mp4" src="./demo.mp4" type="video/mp4">
</video>

目录下有演示视频

可以复制下来自行做定制

## 基本原理

核心一共有四个元素 
prev-item cur-item next-item after-next-item (按显示层级从高到低进行排序)

用户触摸的时候计算一个动画进度 默认为0 最大为1 最小为-1

-1代表prev-item移动到 cur-item 的位置，同理  cur-item 移动到 next-item 的位置

1则正好相反 next-item 移动到 cur-item 的位置，after-next-item 移动到 next-item 的位置

接着只需要写每个元素在不同进度的时候对应的缩放和偏移既可，触摸结束时，变更一下位置索引(由于动画已经结束，不会看到任何变化)