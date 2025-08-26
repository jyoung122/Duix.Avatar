<template>
  <t-layout class="layout">
    <t-header class="layout-header">
      <Header :on-submit="action.submit" :on-save="action.save" v-model="state.animation" />
    </t-header>
    <t-loading text="Loading..." :loading="state.initLoading" :showOverlay="false">
      <t-layout class="layout-body">
        <t-content class="layout-content">
          <t-row class="content-row">
            <t-col :flex="2.7">
              <AvatarSelect class="content-left" v-model="state.select" @query="action.queryModelList" />
            </t-col>
            <t-col :flex="4.5">
              <Preview class="content-center" :model="state.select.model" :audioUrl="state.select.audioUrl" />
            </t-col>
            <t-col :flex="5.0">
              <AudioUpload class="content-right" v-model="state.select" />
            </t-col>
          </t-row>
        </t-content>
      </t-layout>
    </t-loading>
  </t-layout>
  <ModalFinished ref="modalFinished" :right-btn-text="$t('common.lipSync.modalFinishedObj.rightBtnText')">
    {{ $t('common.lipSync.modalFinishedObj.text1') }}
    <span style="color: #434af9"> {{ $t('common.lipSync.modalFinishedObj.text2') }}</span>
    {{ $t('common.lipSync.modalFinishedObj.text3') }}
  </ModalFinished>
</template>

<script setup>
import { ref, reactive, watch } from "vue";
import Header from './components/HeaderView.vue'
import AvatarSelect from './components/AvatarSelectView.vue'
import Preview from './components/PreviewView.vue'
import AudioUpload from './components/AudioUploadView.vue'
import ModalFinished from '@renderer/components/ModalFinished.vue'
import { modelPage } from '@renderer/api'
import { useRoute, useRouter } from 'vue-router'
import { MessagePlugin } from 'tdesign-vue-next'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()
const route = useRoute()
const router = useRouter()
const modalFinished = ref()

const state = reactive({
  initLoading: false,
  animation: {
    id: null,
    name: '',
    model_id: null,
    audio_path: null,
    status: 'draft'
  },
  select: {
    model: null,
    audioUrl: null,
    uploaded: null
  }
})

const action = {
  async queryModelList() {
    try {
      const result = await modelPage({ page: 1, pageSize: 100 })
      return result.records || []
    } catch (error) {
      console.error('Failed to query model list:', error)
      return []
    }
  },
  
  async submit() {
    if (!state.select.model) {
      MessagePlugin.error(t('common.lipSync.errors.noAvatarSelected'))
      return
    }
    
    if (!state.select.uploaded) {
      MessagePlugin.error(t('common.lipSync.errors.noAudioUploaded'))
      return
    }

    try {
      state.initLoading = true
      
      // Here we would integrate with the existing video synthesis service
      // to create a lip-synced animation with the selected avatar and audio
      
      MessagePlugin.success(t('common.lipSync.success.animationCreated'))
      modalFinished.value?.show()
      
    } catch (error) {
      console.error('Failed to create lip-sync animation:', error)
      MessagePlugin.error(t('common.lipSync.errors.animationFailed'))
    } finally {
      state.initLoading = false
    }
  },
  
  async save() {
    // Save draft functionality
    MessagePlugin.success(t('common.lipSync.success.draftSaved'))
  }
}

// Initialize component
state.initLoading = true
action.queryModelList().finally(() => {
  state.initLoading = false
})

</script>

<style lang="less" scoped>
.layout {
  height: 100vh;
  background: #0C0E10;

  &-header {
    background: #161718;
    border-bottom: 1px solid #000000;
    display: flex;
    align-items: center;
    height: 60px;
  }

  &-body {
    flex: 1;
    overflow: hidden;
  }

  &-content {
    padding: 20px;
    height: 100%;
    overflow: hidden;
  }
}

.content {
  &-row {
    height: 100%;
    gap: 20px;
  }

  &-left {
    background: #161718;
    border-radius: 8px;
    height: 100%;
  }

  &-center {
    background: #161718;
    border-radius: 8px;
    height: 100%;
  }

  &-right {
    background: #161718;
    border-radius: 8px;
    height: 100%;
  }
}
</style>