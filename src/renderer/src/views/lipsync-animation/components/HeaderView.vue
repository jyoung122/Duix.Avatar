<template>
  <div class="header">
    <div class="header-left">
      <t-button variant="text" @click="action.goBack">
        <template #icon>
          <ChevronLeftIcon />
        </template>
        {{ $t('common.lipSync.headerView.headerBackText') }}
      </t-button>
    </div>
    <div class="header-center">
      <span>{{ $t('common.lipSync.headerView.title') }}</span>
    </div>
    <div class="header-right">
      <t-button theme="default" @click="onSave" :disabled="!hasChanges">
        {{ $t('common.lipSync.headerView.saveDraftText') }}
      </t-button>
      <t-button theme="primary" @click="onSubmit" :disabled="!canSubmit">
        {{ $t('common.lipSync.headerView.createAnimationText') }}
      </t-button>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { ChevronLeftIcon } from 'tdesign-icons-vue-next'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()
const router = useRouter()

const props = defineProps({
  onSubmit: {
    type: Function,
    required: true
  },
  onSave: {
    type: Function,
    required: true
  }
})

const animation = defineModel({})

const hasChanges = computed(() => {
  return animation.value?.name || animation.value?.model_id || animation.value?.audio_path
})

const canSubmit = computed(() => {
  return animation.value?.model_id && animation.value?.audio_path
})

const action = {
  goBack() {
    router.push('/home')
  }
}

</script>

<style lang="less" scoped>
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  height: 100%;
  width: 100%;

  &-left {
    flex: 1;
    display: flex;
    align-items: center;

    .t-button {
      color: #FFFFFF;
      font-size: 14px;
      
      &:hover {
        background: rgba(255, 255, 255, 0.1);
      }
    }
  }

  &-center {
    flex: 1;
    text-align: center;
    font-weight: 500;
    font-size: 16px;
    color: #FFFFFF;
  }

  &-right {
    flex: 1;
    display: flex;
    justify-content: flex-end;
    gap: 12px;

    .t-button {
      font-size: 14px;
      height: 32px;
      padding: 0 16px;
      
      &[theme="default"] {
        background: #2B3B52;
        border-color: #2B3B52;
        color: #FFFFFF;
        
        &:hover:not(:disabled) {
          background: #3A4A62;
          border-color: #3A4A62;
        }
      }
      
      &[theme="primary"] {
        background: #434AF9;
        border-color: #434AF9;
        
        &:hover:not(:disabled) {
          background: #5A60FA;
          border-color: #5A60FA;
        }
      }
      
      &:disabled {
        opacity: 0.5;
        cursor: not-allowed;
      }
    }
  }
}
</style>