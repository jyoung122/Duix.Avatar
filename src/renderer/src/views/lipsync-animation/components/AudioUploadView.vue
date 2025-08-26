<template>
  <div class="audio-upload">
    <div class="upload-header">{{ $t('common.lipSync.audioUpload.title') }}</div>
    <div class="upload-body">
      
      <!-- Audio upload area -->
      <div class="upload-area" v-if="!select.uploaded">
        <div class="upload-dropzone" @click="action.uploadAudio" @drop="action.onDrop" @dragover="action.onDragOver">
          <div class="upload-icon">🎵</div>
          <div class="upload-text">{{ $t('common.lipSync.audioUpload.dropOrClick') }}</div>
          <div class="upload-subtext">{{ $t('common.lipSync.audioUpload.supportedFormats') }}</div>
          <t-button class="upload-btn" theme="primary" :loading="state.uploading">
            <template #icon>
              <AddIcon />
            </template>
            {{ $t('common.lipSync.audioUpload.selectFile') }}
          </t-button>
        </div>
      </div>
      
      <!-- Uploaded audio info -->
      <div class="uploaded-info" v-else>
        <div class="audio-file">
          <div class="file-icon">🎵</div>
          <div class="file-details">
            <div class="file-name" :title="select.uploaded.name">{{ select.uploaded.name }}</div>
            <div class="file-meta">
              <span class="duration">{{ select.uploaded.duration }}</span>
              <span class="size">{{ formatFileSize(select.uploaded.size) }}</span>
            </div>
          </div>
          <div class="file-actions">
            <t-button class="listen-btn" size="small" @click="action.listen">
              <template #icon>
                <PlayIcon v-if="!state.isPlaying" />
                <PauseIcon v-else />
              </template>
              {{ state.isPlaying ? $t('common.lipSync.audioUpload.pause') : $t('common.lipSync.audioUpload.listen') }}
            </t-button>
            <t-button class="delete-btn" size="small" theme="danger" @click="action.deleteAudio">
              <template #icon>
                <DeleteIcon />
              </template>
            </t-button>
          </div>
        </div>
        
        <!-- Audio waveform visualization (placeholder) -->
        <div class="waveform-container">
          <div class="waveform-placeholder">
            <div class="waveform-bars">
              <div class="bar" v-for="i in 50" :key="i" :style="{ height: Math.random() * 100 + '%' }"></div>
            </div>
          </div>
        </div>
        
        <!-- Lip-sync settings -->
        <div class="lipsync-settings">
          <div class="settings-title">{{ $t('common.lipSync.audioUpload.lipSyncSettings') }}</div>
          
          <div class="setting-item">
            <label>{{ $t('common.lipSync.audioUpload.sensitivity') }}</label>
            <t-slider 
              v-model="state.sensitivity" 
              :min="0" 
              :max="100" 
              :step="1"
              :marks="{ 0: '0%', 50: '50%', 100: '100%' }"
            />
          </div>
          
          <div class="setting-item">
            <label>{{ $t('common.lipSync.audioUpload.smoothing') }}</label>
            <t-slider 
              v-model="state.smoothing" 
              :min="0" 
              :max="100" 
              :step="1"
              :marks="{ 0: '0%', 50: '50%', 100: '100%' }"
            />
          </div>
        </div>
      </div>
      
      <!-- Audio element for playback -->
      <audio 
        ref="audioElement"
        v-if="select.uploaded"
        :src="select.uploaded.audioUrl"
        @ended="onAudioEnded"
        @error="onAudioError"
      />
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { AddIcon, PlayIcon, PauseIcon, DeleteIcon } from 'tdesign-icons-vue-next'
import { MessagePlugin } from 'tdesign-vue-next'
import { Client } from '@renderer/client'
import { millisecondsToTime } from '@renderer/utils'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

const select = defineModel({})
const audioElement = ref(null)

const state = reactive({
  uploading: false,
  isPlaying: false,
  sensitivity: 75,
  smoothing: 50
})

const action = {
  async uploadAudio() {
    const filePath = await Client.file.selectAudio()
    if (filePath) {
      state.uploading = true
      try {
        const info = await Client.file.getAudioInfo(filePath)
        if (action.check(info)) {
          select.value.uploaded = {
            name: info.name,
            audioUrl: filePath,
            duration: millisecondsToTime(info.duration * 1000),
            size: info.size || 0
          }
          select.value.audioUrl = filePath
          select.value.audio_path = filePath
        }
      } catch (err) {
        console.error('Audio upload error:', err)
        MessagePlugin.error(t('common.lipSync.audioUpload.uploadError'))
      } finally {
        state.uploading = false
      }
    }
  },
  
  check(audioInfo) {
    if (!audioInfo.isOK) {
      MessagePlugin.error(audioInfo.msg || t('common.lipSync.audioUpload.uploadError'))
      return false
    }
    if (audioInfo.duration > 60 * 30) { // 30 minutes max
      MessagePlugin.error(t('common.lipSync.audioUpload.durationError'))
      return false
    }
    return true
  },
  
  deleteAudio() {
    if (state.isPlaying && audioElement.value) {
      audioElement.value.pause()
      state.isPlaying = false
    }
    select.value.uploaded = null
    select.value.audioUrl = null
    select.value.audio_path = null
  },
  
  async listen() {
    if (!audioElement.value) return
    
    try {
      if (state.isPlaying) {
        audioElement.value.pause()
        state.isPlaying = false
      } else {
        await audioElement.value.play()
        state.isPlaying = true
      }
    } catch (error) {
      console.error('Audio playback error:', error)
      MessagePlugin.error(t('common.lipSync.audioUpload.playbackError'))
    }
  },
  
  onDrop(e) {
    e.preventDefault()
    const files = e.dataTransfer.files
    if (files.length > 0) {
      action.handleFile(files[0])
    }
  },
  
  onDragOver(e) {
    e.preventDefault()
  },
  
  async handleFile(file) {
    // Handle dropped file
    const supportedTypes = ['audio/mp3', 'audio/wav', 'audio/flac', 'audio/m4a', 'audio/mpeg', 'audio/x-wav']
    if (!supportedTypes.includes(file.type)) {
      MessagePlugin.error(t('common.lipSync.audioUpload.unsupportedFormat'))
      return
    }
    
    state.uploading = true
    try {
      // Convert file to path format expected by the app
      const audioUrl = URL.createObjectURL(file)
      const info = {
        name: file.name,
        duration: 0, // Would need to be calculated
        size: file.size,
        isOK: true
      }
      
      select.value.uploaded = {
        name: info.name,
        audioUrl: audioUrl,
        duration: formatTime(info.duration),
        size: info.size
      }
      select.value.audioUrl = audioUrl
      select.value.audio_path = audioUrl
      
    } catch (error) {
      console.error('File handling error:', error)
      MessagePlugin.error(t('common.lipSync.audioUpload.uploadError'))
    } finally {
      state.uploading = false
    }
  }
}

const onAudioEnded = () => {
  state.isPlaying = false
}

const onAudioError = (error) => {
  console.error('Audio error:', error)
  state.isPlaying = false
  MessagePlugin.error(t('common.lipSync.audioUpload.playbackError'))
}

const formatFileSize = (bytes) => {
  if (!bytes) return '0 B'
  const k = 1024
  const sizes = ['B', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return parseFloat((bytes / Math.pow(k, i)).toFixed(1)) + ' ' + sizes[i]
}

const formatTime = (seconds) => {
  if (!seconds || isNaN(seconds)) return '0:00'
  const mins = Math.floor(seconds / 60)
  const secs = Math.floor(seconds % 60)
  return `${mins}:${secs.toString().padStart(2, '0')}`
}

</script>

<style lang="less" scoped>
.audio-upload {
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
    overflow-y: auto;
  }
}

.upload-area {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  
  .upload-dropzone {
    border: 2px dashed #333;
    border-radius: 12px;
    padding: 40px 20px;
    text-align: center;
    cursor: pointer;
    transition: all 0.2s ease;
    width: 100%;
    max-width: 400px;
    
    &:hover {
      border-color: #434AF9;
      background: rgba(67, 74, 249, 0.05);
    }
    
    .upload-icon {
      font-size: 48px;
      margin-bottom: 16px;
      opacity: 0.7;
    }
    
    .upload-text {
      font-size: 16px;
      color: #FFFFFF;
      margin-bottom: 8px;
    }
    
    .upload-subtext {
      font-size: 12px;
      color: #666;
      margin-bottom: 24px;
    }
    
    .upload-btn {
      font-size: 14px;
    }
  }
}

.uploaded-info {
  .audio-file {
    display: flex;
    align-items: center;
    background: #0C0E10;
    border-radius: 8px;
    padding: 16px;
    margin-bottom: 20px;
    
    .file-icon {
      font-size: 24px;
      margin-right: 12px;
    }
    
    .file-details {
      flex: 1;
      min-width: 0;
      
      .file-name {
        font-size: 14px;
        color: #FFFFFF;
        font-weight: 500;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        margin-bottom: 4px;
      }
      
      .file-meta {
        display: flex;
        gap: 12px;
        font-size: 12px;
        color: #666;
        
        .duration, .size {
          font-family: monospace;
        }
      }
    }
    
    .file-actions {
      display: flex;
      gap: 8px;
      
      .t-button {
        font-size: 12px;
        
        &.listen-btn {
          background: #434AF9;
          border-color: #434AF9;
          
          &:hover {
            background: #5A60FA;
            border-color: #5A60FA;
          }
        }
      }
    }
  }
  
  .waveform-container {
    margin-bottom: 24px;
    
    .waveform-placeholder {
      background: #0C0E10;
      border-radius: 8px;
      padding: 20px;
      
      .waveform-bars {
        display: flex;
        align-items: end;
        justify-content: space-between;
        height: 60px;
        gap: 2px;
        
        .bar {
          background: linear-gradient(to top, #434AF9, #5A60FA);
          width: 3px;
          border-radius: 1px;
          min-height: 2px;
          opacity: 0.7;
        }
      }
    }
  }
  
  .lipsync-settings {
    .settings-title {
      font-size: 14px;
      color: #FFFFFF;
      font-weight: 500;
      margin-bottom: 16px;
    }
    
    .setting-item {
      margin-bottom: 20px;
      
      label {
        display: block;
        font-size: 12px;
        color: #FFFFFF;
        margin-bottom: 8px;
      }
      
      :deep(.t-slider) {
        .t-slider__rail {
          background: #333;
        }
        
        .t-slider__track {
          background: #434AF9;
        }
        
        .t-slider__button {
          border-color: #434AF9;
          background: #434AF9;
        }
        
        .t-slider__mark-text {
          color: #666;
          font-size: 10px;
        }
      }
    }
  }
}
</style>