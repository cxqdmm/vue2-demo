<template>
  <div class="video-container" :class="{ 'floating': isFloating }">
    <div class="video-wrapper"  @mouseenter="onMouseEnter" @mouseleave="onMouseLeave">
      <!-- 背景图片 -->
      <div class="poster-container">
        <img :src="poster" alt="Video poster" class="poster-image" />
        <!-- 播放按钮 -->
        <div class="play-button-overlay" v-show="isInitialState && !isPosterOnlyMode" @click="togglePlay">
          <div class="play-btn">
            <van-icon :name="isPlaying ? 'pause' : 'play'" />
          </div>
        </div>
      </div>

      <!-- 视频播放器容器 -->
       <div class="video-player-box" ref="videoWrapper" :style="{ paddingTop: (aspectRatio * 100) + '%' }">
         <div class="video-player-container"  v-show="shouldShowVideoContainer">
           <!-- 关闭按钮 -->
           <van-icon name="cross" class="video-close-btn" @click="closeVideo" />
           <!-- 视频暂停时的播放按钮 -->
           <div class="play-button-overlay" v-show="isPausedWithProgress" @click="togglePlay">
             <div class="play-btn">
               <van-icon name="play" />
             </div>
           </div>
           <video v-if="!isPosterOnlyMode" ref="videoElement" :src="videoSrc" preload="metadata" @play="onPlay" @pause="onPause"
             @timeupdate="onTimeUpdate" @loadedmetadata="onLoadedMetadata" @click="togglePlay" class="video-player">
             您的浏览器不支持视频播放
           </video>
           <!-- 自定义控制栏 -->
           <div class="custom-controls" v-show="showControls || showPlayButton">
             <!-- <div class="controls-left">
               <div class="volume-control">
                 <button class="control-btn volume-btn" @click="toggleMute">
                   <van-icon :name="volumeIcon" />
                 </button>
               </div>
             </div> -->
   
             <!-- 进度条 -->
             <div class="progress-container">
               <div class="progress-bar" @click="onProgressClick" ref="progressBar">
                 <div class="progress-filled" :style="{ width: progressPercentage + '%' }"></div>
               </div>
             </div>
   
             <div class="controls-right">
               <!-- 时间显示 -->
               <div class="time-display">
                 <span class="current-time">{{ formatTime(currentTime) }}</span>
                 <span class="separator">/</span>
                 <span class="total-time">{{ formatTime(duration) }}</span>
               </div>
             </div>
           </div>
         </div>
       </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'VideoPlayer',
  props: {
    videoSrc: {
      type: String,
      default: ''
    },
    poster: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      isFloating: false,
      observer: null,
      isPlaying: false,
      currentTime: 0,
      duration: 0,
      showControls: true,
      showPlayButton: true,
      hideControlsTimer: null,
      volume: 1,
      isMuted: false,
      playbackRate: 1,
      playbackRates: [0.5, 0.75, 1, 1.25, 1.5, 2],
      showRateMenu: false,
      manuallyClosedFloating: false,
      videoWidth: 0,
      videoHeight: 0
    }
  },
  computed: {
    // 是否只显示poster模式（没有视频源）
    isPosterOnlyMode() {
      return !this.videoSrc || this.videoSrc.trim() === ''
    },
    progressPercentage() {
      return this.duration > 0 ? (this.currentTime / this.duration) * 100 : 0
    },
    volumeIcon() {
      return this.isMuted ? 'volume' : 'volume-off';
    },
    // 视频是否已开始播放（有播放进度）
    hasStarted() {
      return this.currentTime > 0
    },
    // 视频是否处于暂停状态且已开始播放
    isPausedWithProgress() {
      return !this.isPlaying && this.hasStarted
    },
    // 视频是否处于初始状态（未开始播放）
    isInitialState() {
      return (!this.isPlaying && !this.hasStarted) && !this.isPosterOnlyMode
    },
    // 视频容器是否应该显示
    shouldShowVideoContainer() {
      return (this.isPlaying || this.hasStarted) && !this.isPosterOnlyMode
    },
    // 视频宽高比
    aspectRatio() {
      return this.videoHeight > 0 ? this.videoHeight / this.videoWidth : 16 / 9
    },
    // 是否为横屏视频
    isLandscape() {
      return this.aspectRatio > 1
    },
    // 是否为竖屏视频
    isPortrait() {
      return this.aspectRatio < 1
    }
  },
  mounted() {
    // 只在非poster-only模式下初始化视频相关功能
    if (!this.isPosterOnlyMode) {
      this.initIntersectionObserver()
      // 初始化音量
      this.$nextTick(() => {
        if (this.$refs.videoElement) {
          this.$refs.videoElement.volume = this.volume
        }
      })
      // 添加全局点击事件监听器
      document.addEventListener('click', this.handleGlobalClick)
    }
  },
  beforeDestroy() {
    // 只在非poster-only模式下清理视频相关功能
    if (!this.isPosterOnlyMode) {
      if (this.observer) {
        this.observer.disconnect();
      }
      this.clearHideTimer();
      document.removeEventListener('click', this.handleGlobalClick);
    }
  },
  methods: {
    initIntersectionObserver() {
      // 创建Intersection Observer来监听视频是否在视口中
      this.observer = new IntersectionObserver(
        (entries) => {
          entries.forEach((entry) => {
            // 当视频完全离开视口时，启用悬浮模式（除非用户手动关闭了悬浮模式）
            if (entry.intersectionRatio === 0 && !this.manuallyClosedFloating) {
              this.isFloating = true
            } else if (entry.intersectionRatio > 0) {
              // 当视频重新进入视口时，关闭悬浮模式并重置手动关闭标志
              this.isFloating = false
              this.manuallyClosedFloating = false
            }
          })
        },
        {
          threshold: [0, 0.1] // 监听完全离开和部分进入
        }
      )

      // 开始观察视频容器
      this.observer.observe(this.$refs.videoWrapper)
    },

    onPlay() {
      this.isPlaying = true
      this.showPlayButton = false
      this.showControls = true
      this.startHideTimer()
      this.$emit('play')
    },

    onPause() {
      this.isPlaying = false
      this.showPlayButton = true
      this.showControls = true
      this.clearHideTimer()
      // 暂停时不重置手动关闭标志，保持悬浮状态
      this.$emit('pause')
    },

    onTimeUpdate() {
      this.currentTime = this.$refs.videoElement.currentTime
      this.$emit('timeupdate', this.currentTime)
    },

    onLoadedMetadata() {
      this.duration = this.$refs.videoElement.duration
      this.videoWidth = this.$refs.videoElement.videoWidth
      this.videoHeight = this.$refs.videoElement.videoHeight
      this.$emit('metadata', {
        duration: this.duration,
        width: this.videoWidth,
        height: this.videoHeight,
        aspectRatio: this.aspectRatio
      })
    },

    formatTime(seconds) {
      if (isNaN(seconds)) return '0:00'
      const minutes = Math.floor(seconds / 60)
      const remainingSeconds = Math.floor(seconds % 60)
      return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`
    },

    onProgressClick(event) {
      const progressBar = this.$refs.progressBar
      const rect = progressBar.getBoundingClientRect()
      const clickX = event.clientX - rect.left
      const progressWidth = rect.width
      const clickPercentage = clickX / progressWidth
      const newTime = clickPercentage * this.duration

      this.$refs.videoElement.currentTime = newTime
      this.currentTime = newTime
    },

    // 音量控制方法
    toggleMute() {
      this.isMuted = !this.isMuted
      this.$refs.videoElement.muted = this.isMuted
    },

    // 倍速控制方法
    toggleRateMenu() {
      this.showRateMenu = !this.showRateMenu
    },

    setPlaybackRate(rate) {
      this.playbackRate = rate
      this.$refs.videoElement.playbackRate = rate
      this.showRateMenu = false
    },

    // 处理全局点击事件
    handleGlobalClick(event) {
      // 检查是否点击在倍速菜单外部
      if (this.showRateMenu && !event.target.closest('.playback-rate')) {
        this.showRateMenu = false
      }
    },

    closeFloating() {
      this.isFloating = false
      this.manuallyClosedFloating = true
      // 不自动暂停视频，让用户自己控制
      // this.$refs.videoElement.pause()
    },

    closeVideo() {
      // 暂停视频
      this.$refs.videoElement.pause()
      // 重置播放时间到开头
      this.$refs.videoElement.currentTime = 0
      this.currentTime = 0
      // 重置播放状态
      this.isPlaying = false
      this.showPlayButton = true
      this.showControls = true
      // 关闭悬浮模式
      this.isFloating = false
      this.manuallyClosedFloating = false
      // 清除定时器
      this.clearHideTimer()
      this.$emit('close')
    },

    togglePlay() {
      if (this.isPlaying) {
        this.$refs.videoElement.pause()
      } else {
        this.$refs.videoElement.play()
      }
    },

    onMouseEnter() {
      if (this.isPlaying) {
        this.showPlayButton = true
        this.showControls = true
        this.clearHideTimer()
      }
    },

    onMouseLeave() {
      if (this.isPlaying) {
        this.startHideTimer()
      }
    },

    startHideTimer() {
      this.clearHideTimer()
      this.hideControlsTimer = setTimeout(() => {
        if (this.isPlaying) {
          this.showPlayButton = false
          this.showControls = false
        }
      }, 2000)
    },

    clearHideTimer() {
      if (this.hideControlsTimer) {
        clearTimeout(this.hideControlsTimer)
        this.hideControlsTimer = null
      }
    },

    // 外部控制方法
    play() {
      this.$refs.videoElement.play()
    },

    pause() {
      this.$refs.videoElement.pause()
    },

    setCurrentTime(time) {
      this.$refs.videoElement.currentTime = time
    }
  }
}
</script>

<style scoped>
.video-container {
  width: 100%;
  position: relative;
}

.video-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
}

.poster-container {
  position: relative;
  width: 100%;
  height: 100%;
}

.poster-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.play-button-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 4;
}

.video-player {
  position: relative;
  border-radius: 5px;
  width: 100%;
  height: auto;
  object-fit: cover;
  display: block;
}

/* 非悬浮状态下的视频样式 */
.video-container:not(.floating) {
  height: auto;
}

.video-container:not(.floating) .poster-container {
  position: relative;
  width: 100%;
}

.video-container:not(.floating) .poster-image {
  width: 100%;
  height: auto;
  display: block;
  object-fit: cover;
}

.video-container:not(.floating) .video-player {
  position: relative;
  width: 100%;
  height: auto;
  object-fit: cover;
  display: block;
}

.close-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  background: rgba(0, 0, 0, 0.6);
  color: white;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 14px;
}

.close-btn:hover {
  background: rgba(0, 0, 0, 0.8);
}

.play-btn {
  width: 80px;
  height: 80px;
  background: rgba(0, 0, 0, 0.7);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  backdrop-filter: blur(10px);
}

.play-btn:hover {
  background: rgba(0, 0, 0, 0.9);
}

.play-btn .van-icon {
  color: white;
  font-size: 32px;
}

/* 悬浮状态下的播放按钮样式 */
.video-container.floating .play-btn {
  width: 40px;
  height: 40px;
}

.video-container.floating .play-btn .van-icon {
  font-size: 16px;
}
.video-player-box {
  position: absolute;
  top: 20px;
  left: 20px;
  right: 20px;
  border-radius: 10px;
  height: 0;
  z-index: 2;
}
/* 视频播放器容器样式 */
.video-player-container {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  border-radius: 10px;
  height: auto;

}


/* 悬浮状态下的视频播放器容器样式 */
.video-container.floating .video-player-container {
  position: fixed;
  top: 28px;
  left: 28px;
  width: 160px;
  height: 90px;
  border-radius: 6px;
  z-index: 1001;
}

.video-close-btn {
  position: absolute;
  font-size: 12px;
  top: 0px;
  right: 0px;
  z-index: 10;
  color: white;
  border-radius: 50%;
  padding: 8px;
  cursor: pointer;
}

/* 自定义控制栏样式 */
.custom-controls {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.8), rgba(0, 0, 0, 0.4), transparent);
  padding: 16px 12px 12px;
  z-index: 5;
  display: flex;
  align-items: center;
  border-radius: 0 0 5px 5px;
  gap: 8px;
}

.video-container.floating .custom-controls {
  padding: 12px 16px 8px;
  gap: 6px;
}

.controls-left,
.controls-right {
  display: flex;
  align-items: center;
  gap: 8px;
}

.controls-left {
  flex-shrink: 0;
}

.controls-right {
  flex-shrink: 0;
}

/* 控制按钮样式 */
.control-btn {
  background: none;
  border: none;
  color: white;
  cursor: pointer;
  padding: 8px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 32px;
  height: 32px;
}


.video-container.floating .control-btn {
  min-width: 28px;
  height: 28px;
  padding: 6px;
}

/* 音量控制样式 */
.volume-control {
  display: flex;
  align-items: center;
}

/* 倍速控制样式 */
.playback-rate {
  position: relative;
}

.rate-btn {
  font-size: 12px;
  font-weight: bold;
  min-width: 40px;
}

.rate-menu {
  position: absolute;
  bottom: 100%;
  right: 0;
  background: rgba(0, 0, 0, 0.9);
  border-radius: 4px;
  padding: 4px 0;
  margin-bottom: 8px;
  min-width: 60px;
}

.rate-option {
  padding: 8px 12px;
  color: white;
  cursor: pointer;
  font-size: 12px;
  text-align: center;
}

.rate-option:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.rate-option.active {
  background-color: #b4b0ae;
  color: white;
}

.progress-container {
  flex: 1;
  margin: 0;
}

/* 进度条样式 */
.progress-bar {
  width: 100%;
  height: 4px;
  background: #59524f;
  border-radius: 2px;
  position: relative;
  cursor: pointer;
}

.progress-filled {
  height: 100%;
  background: #b4b0ae;
  border-radius: 2px;
}

/* 时间显示样式 */
.time-display {
  display: flex;
  align-items: center;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.95);
  font-weight: 400;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.8);
  white-space: nowrap;
}

.video-container.floating .time-display {
  font-size: 10px;
}

.current-time,
.total-time {
  font-family: 'Segoe UI', system-ui, sans-serif;
  letter-spacing: 0.3px;
  text-align: center;
}

.separator {
  color: rgba(255, 255, 255, 0.7);
  margin: 0 2px;
}
</style>