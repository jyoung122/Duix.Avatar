<template>
  <div class="avatar-select">
    <div class="select-header">{{ $t('common.lipSync.avatarSelect.title') }}</div>
    <div class="select-body">
      <!-- Search input -->
      <t-input 
        class="model-search" 
        v-model="state.search" 
        :placeholder="$t('common.lipSync.avatarSelect.searchPlaceholder')"
        @change="action.searchList"
      >
        <template #prefix-icon>
          <SearchIcon />
        </template>
      </t-input>
      
      <div class="model-scroll noscrollbar">
        <!-- Avatar list -->
        <div class="model-list">
          <div 
            class="model-list__item" 
            v-for="model in filteredModels" 
            :key="model.id"
            @click="action.selectModel(model)" 
            :class="{ '--active': data.model?.id === model.id }"
          >
            <video class="video" :src="localUrl.addFileProtocol(model.video_path)" />
            <div class="name" :title="model.name">{{ model.name }}</div>
            <div class="selected-indicator" v-if="data.model?.id === model.id">
              <CheckIcon />
            </div>
          </div>
        </div>
        
        <!-- Empty state -->
        <div class="empty-state" v-if="filteredModels.length === 0">
          <div class="empty-icon">👤</div>
          <div class="empty-text">{{ $t('common.lipSync.avatarSelect.noAvatarsFound') }}</div>
          <t-button theme="primary" size="small" @click="action.createAvatar">
            {{ $t('common.lipSync.avatarSelect.createAvatar') }}
          </t-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, onMounted, computed } from 'vue'
import { SearchIcon, CheckIcon } from 'tdesign-icons-vue-next'
import { useRouter } from 'vue-router'
import { localUrl } from '@renderer/utils'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()
const router = useRouter()

const data = defineModel({})

const state = reactive({
  search: '',
  models: []
})

const props = defineProps({
  query: {
    type: Function,
    required: true
  }
})

const emits = defineEmits(['query'])

const filteredModels = computed(() => {
  if (!state.search) return state.models
  return state.models.filter(model => 
    model.name.toLowerCase().includes(state.search.toLowerCase())
  )
})

const action = {
  async searchList() {
    // The filtering is handled by computed property
  },
  
  selectModel(model) {
    data.value.model = model
    data.value.model_id = model.id
  },
  
  createAvatar() {
    router.push('/video/edit')
  },
  
  async loadModels() {
    try {
      const models = await props.query()
      state.models = models
    } catch (error) {
      console.error('Failed to load avatars:', error)
      state.models = []
    }
  }
}

onMounted(() => {
  action.loadModels()
})

</script>

<style lang="less" scoped>
.avatar-select {
  display: flex;
  height: 100%;
  flex-direction: column;
  
  .select-header {
    font-weight: 500;
    padding: 18px;
    font-size: 14px;
    color: #ffffff;
    line-height: 22px;
    text-align: center;
    border-bottom: 1px solid #000000;
  }
  
  .select-body {
    flex: 1;
    padding: 20px;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }
  
  .model-search {
    margin-bottom: 16px;
    
    :deep(.t-input__inner) {
      background: #0C0E10;
      border-color: #333;
      color: #FFFFFF;
      
      &::placeholder {
        color: #666;
      }
      
      &:focus {
        border-color: #434AF9;
      }
    }
    
    :deep(.t-input__prefix) {
      color: #666;
    }
  }
  
  .model-scroll {
    flex: 1;
    overflow-y: auto;
    
    &.noscrollbar {
      scrollbar-width: none;
      -ms-overflow-style: none;
      
      &::-webkit-scrollbar {
        display: none;
      }
    }
  }
  
  .model-list {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 12px;
    
    &__item {
      position: relative;
      background: #0C0E10;
      border-radius: 8px;
      overflow: hidden;
      cursor: pointer;
      transition: all 0.2s ease;
      border: 2px solid transparent;
      
      &:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
      }
      
      &.--active {
        border-color: #434AF9;
        
        .selected-indicator {
          display: flex;
        }
      }
      
      .video {
        width: 100%;
        height: 120px;
        object-fit: cover;
        background: linear-gradient(180deg, #b8c2ce 0%, #e2e6f0 100%);
      }
      
      .name {
        padding: 8px 12px;
        font-size: 12px;
        color: #FFFFFF;
        text-align: center;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
      
      .selected-indicator {
        display: none;
        position: absolute;
        top: 8px;
        right: 8px;
        width: 20px;
        height: 20px;
        background: #434AF9;
        border-radius: 50%;
        align-items: center;
        justify-content: center;
        color: white;
        font-size: 12px;
      }
    }
  }
  
  .empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 200px;
    color: #666;
    
    .empty-icon {
      font-size: 48px;
      margin-bottom: 16px;
    }
    
    .empty-text {
      margin-bottom: 16px;
      font-size: 14px;
    }
  }
}
</style>