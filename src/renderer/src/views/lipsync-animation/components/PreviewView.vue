<template>
  <div class="preview">
    <div class="preview-header">{{ $t('common.lipSync.preview.title') }}</div>
    <div class="preview-body">
      <div class="avatar-preview" v-if="model">
        <video 
          ref="avatarVideo"
          class="avatar-video" 
          :src="localUrl.addFileProtocol(model.video_path)"
          :poster="localUrl.addFileProtocol(model.video_path)"
          @loadedmetadata="onVideoLoaded"
        />
        
        <!-- Audio controls overlay -->
        <div class="audio-controls" v-if="audioUrl">
          <t-button 
            class="play-btn"
            :theme="state.isPlaying ? 'default' : 'primary'"
            @click="action.togglePlayback"
            :disabled="state.isLoading"
          >
            <template #icon>
              <PlayIcon v-if="!state.isPlaying" />
              <PauseIcon v-else />
            </template>
            {{ state.isPlaying ? $t('common.lipSync.preview.pause') : $t('common.lipSync.preview.play') }}
          </t-button>
          
          <div class="audio-info" v-if="state.audioInfo">
            <span class="duration">{{ formatTime(state.currentTime) }} / {{ formatTime(state.duration) }}</span>
          </div>
        </div>
        
        <!-- Progress bar -->
        <div class="progress-bar" v-if="audioUrl && state.duration > 0">
          <div 
            class="progress-fill" 
            :style="{ width: (state.currentTime / state.duration * 100) + '%' }"
          ></div>
        </div>
      </div>
      
      <!-- Placeholder when no avatar selected -->
      <div class="placeholder" v-else>
        <div class="placeholder-icon">🎭</div>
        <div class="placeholder-text">{{ $t('common.lipSync.preview.selectAvatarPrompt') }}</div>
      </div>
      
      <!-- Audio element for playback -->
      <audio 
        ref="audioElement"
        v-if="audioUrl"
        :src="audioUrl"
        @loadedmetadata="onAudioLoaded"
        @timeupdate="onTimeUpdate"
        @ended="onAudioEnded"
        @error="onAudioError"
      />
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, watch, onUnmounted } from 'vue'
import { PlayIcon, PauseIcon } from 'tdesign-icons-vue-next'
import { localUrl } from '@renderer/utils'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

const props = defineProps({
  model: {
    type: Object,
    default: null
  },
  audioUrl: {
    type: String,
    default: null
  }
})

const avatarVideo = ref(null)
const audioElement = ref(null)

const state = reactive({
  isPlaying: false,
  isLoading: false,
  currentTime: 0,
  duration: 0,
  audioInfo: null
})

const action = {
  async togglePlayback() {
    if (!audioElement.value || !avatarVideo.value) return
    
    state.isLoading = true
    
    try {
      if (state.isPlaying) {
        await audioElement.value.pause()
        avatarVideo.value.pause()
        state.isPlaying = false
      } else {
        // Sync video and audio playback
        avatarVideo.value.currentTime = audioElement.value.currentTime
        
        await Promise.all([
          audioElement.value.play(),
          avatarVideo.value.play()
        ])
        
        state.isPlaying = true
      }
    } catch (error) {
      console.error('Playback error:', error)
    } finally {
      state.isLoading = false
    }
  },
  
  stopPlayback() {
    if (audioElement.value) {
      audioElement.value.pause()
      audioElement.value.currentTime = 0
    }
    if (avatarVideo.value) {
      avatarVideo.value.pause()
      avatarVideo.value.currentTime = 0
    }
    state.isPlaying = false
    state.currentTime = 0
  }
}

const onVideoLoaded = () => {
  if (avatarVideo.value) {
    // Set up video for lip-sync preview
    avatarVideo.value.loop = true
    avatarVideo.value.muted = true // We use the audio element for sound
  }
}

const onAudioLoaded = () => {
  if (audioElement.value) {
    state.duration = audioElement.value.duration
    state.audioInfo = {
      duration: audioElement.value.duration
    }
  }
}

const onTimeUpdate = () => {
  if (audioElement.value) {
    state.currentTime = audioElement.value.currentTime
    
    // Sync avatar video with audio
    if (avatarVideo.value && Math.abs(avatarVideo.value.currentTime - state.currentTime) > 0.1) {
      avatarVideo.value.currentTime = state.currentTime
    }
  }
}

const onAudioEnded = () => {
  state.isPlaying = false
  action.stopPlayback()
}

const onAudioError = (error) => {
  console.error('Audio error:', error)
  state.isPlaying = false
}

const formatTime = (seconds) => {
  if (!seconds || isNaN(seconds)) return '0:00'
  const mins = Math.floor(seconds / 60)
  const secs = Math.floor(seconds % 60)
  return `${mins}:${secs.toString().padStart(2, '0')}`
}

// Watch for audio URL changes
watch(() => props.audioUrl, (newUrl) => {
  if (newUrl) {
    action.stopPlayback()
  }
})

// Cleanup on unmount
onUnmounted(() => {
  action.stopPlayback()
})

</script>

<style lang="less" scoped>
.preview {
  display: flex;
  height: 100%;
  flex-direction: column;
  
  &-header {
    font-weight: 500;
    padding: 18px;
    font-size: 14px;
    color: #ffffff;
    line-height: 22px;
    text-align: center;
    border-bottom: 1px solid #000000;
  }
  
  &-body {
    flex: 1;
    padding: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
  }
}

.avatar-preview {
  position: relative;
  width: 100%;
  max-width: 400px;
  border-radius: 12px;
  overflow: hidden;
  background: linear-gradient(180deg, #b8c2ce 0%, #e2e6f0 100%);
  
  .avatar-video {
    width: 100%;
    height: auto;
    display: block;
    object-fit: contain;
    max-height: 500px;
  }
  
  .audio-controls {
    position: absolute;
    bottom: 60px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    align-items: center;
    gap: 12px;
    background: rgba(0, 0, 0, 0.8);
    padding: 12px 16px;
    border-radius: 8px;
    backdrop-filter: blur(10px);
    
    .play-btn {
      font-size: 14px;
      
      &[theme="primary"] {
        background: #434AF9;
        border-color: #434AF9;
        
        &:hover:not(:disabled) {
          background: #5A60FA;
          border-color: #5A60FA;
        }
      }
    }
    
    .audio-info {
      color: #FFFFFF;
      font-size: 12px;
      
      .duration {
        font-family: monospace;
      }
    }
  }
  
  .progress-bar {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 3px;
    background: rgba(255, 255, 255, 0.2);
    
    .progress-fill {
      height: 100%;
      background: #434AF9;
      transition: width 0.1s ease;
    }
  }
}

.placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #666;
  text-align: center;
  
  .placeholder-icon {
    font-size: 64px;
    margin-bottom: 16px;
    opacity: 0.5;
  }
  
  .placeholder-text {
    font-size: 16px;
    line-height: 1.5;
  }
}
</style>