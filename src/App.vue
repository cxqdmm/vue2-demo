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
    
    <!-- Vant Input 组件区域 -->
    <div class="input-section">
      <van-field
        v-model="inputValue"
        label="用户名"
        placeholder="请输入用户名"
        clearable
        @input="onInputChange"
      />
      
      <van-field··
        v-model="password"
        type="password"
        label="密码"
        placeholder="请输入密码"
        clearable
      />
      
      <van-field
        v-model="message"
        rows="2"
        autosize
        label="留言"
        type="textarea"
        placeholder="请输入留言"
        show-word-limit
        maxlength="50"
      />
      
      <div class="button-group">
        <van-button type="primary" size="large" @click="handleSubmit">
          提交
        </van-button>
        <van-button size="large" @click="handleClear">
          清空
        </van-button>
      </div>
    </div>
    
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
      videoSrc: 'https://iobs.pingan.com.cn/download/ehis-iproductmarket-sf-prd/siWKzIf1.mp4',
      posterSrc: 'https://stg.iobs.pingan.com.cn/download/ehis-group-insurance-csp-cbs-sf-stg/17510221409791804',
      videoStatus: '未播放',
      currentVideoTime: 0,
      // Vant Input 相关数据
      inputValue: '',
      password: '',
      message: ''
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
    },
    // Vant Input 相关方法
    onInputChange(value) {
      console.log('输入值变化:', value)
    },
    handleSubmit() {
      if (!this.inputValue.trim()) {
        this.$toast('请输入用户名')
        return
      }
      if (!this.password.trim()) {
        this.$toast('请输入密码')
        return
      }
      
      this.$toast.success('提交成功!')
      console.log('提交的数据:', {
        username: this.inputValue,
        password: this.password,
        message: this.message
      })
    },
    handleClear() {
      this.inputValue = ''
      this.password = ''
      this.message = ''
      this.$toast('已清空所有输入')
    }
  }
}
</script>

<style scoped>
#app {
  min-height: 100vh;
}

.input-section {
  padding: 20px;
  background-color: #f8f9fa;
  margin: 20px 0;
}

.button-group {
  margin-top: 20px;
  display: flex;
  gap: 10px;
}

.button-group .van-button {
  flex: 1;
}

.content-section {
  height: 3000px;
  padding-bottom: 50px;
}
</style>
