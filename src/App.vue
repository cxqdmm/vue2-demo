<template>
  <div id="app">

    
    <!-- 视频组件 - 位于页面顶部 -->
    <VideoPlayer
      :video-src="videoSrc"
      :poster="posterSrc"
      @play="onVideoPlay"
      @pause="onVideoPause"
      @timeupdate="onVideoTimeUpdate"
    />
    <div class="content-section"></div>
  </div>
</template>

<script>
import VideoPlayer from './components/VideoPlayer.vue'

export default {
  name: 'App',
  components: {
    VideoPlayer
  },
  data() {
    return {
      count: 0,
      videoSrc: '',
      // videoSrc: 'https://iobs.pingan.com.cn/download/ehis-iproductmarket-sf-prd/siWKzIf1.mp4',
      posterSrc: 'https://stg.iobs.pingan.com.cn/download/ehis-group-insurance-csp-cbs-sf-stg/17510221409791804',
      videoStatus: '未播放',
      currentVideoTime: 0
    }
  },
  methods: {
    increment() {
      this.count++
    },
    decrement() {
      this.count--
    },
    showToast() {
      this.$toast('Hello Vant2!')
    },
    onVideoPlay() {
      this.videoStatus = '播放中'
    },
    onVideoPause() {
      this.videoStatus = '已暂停'
    },
    onVideoTimeUpdate(time) {
      this.currentVideoTime = Math.floor(time)
      this.videoStatus = `播放中 ${this.formatTime(time)}`
    },
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60)
      const secs = Math.floor(seconds % 60)
      return `${mins}:${secs.toString().padStart(2, '0')}`
    }
  }
}
</script>

<style scoped>
#app {
  min-height: 100vh;
}

.content-section {
  height: 3000px;
  padding-bottom: 50px;
}
</style>
