<script setup>
import { ref } from 'vue';
import { useRegisterSW } from 'virtual:pwa-register/vue';
import { useToastStore } from '../stores/toast.js';

const { showToast } = useToastStore();

const {
  needRefresh,
  updateServiceWorker,
} = useRegisterSW({
  onRegistered(registration) {
    console.log('PWA: Service Worker 已注册');
  },
  onRegisterError(error) {
    console.error('PWA: Service Worker 注册失败', error);
  },
  onNeedRefresh() {
    console.log('PWA: 发现新版本');
    showToast('发现新版本，点击更新按钮获取最新功能', 'info', 8000);
  },
});

const handleUpdate = async () => {
  try {
    await updateServiceWorker(true);
  } catch (error) {
    console.error('更新失败:', error);
    showToast('更新失败，请刷新页面重试', 'error');
  }
};

const dismissUpdate = () => {
  needRefresh.value = false;
};
</script>

<template>
  <Transition name="update-banner">
    <div
      v-if="needRefresh"
      class="fixed bottom-4 left-4 right-4 z-50 mx-auto max-w-md sm:left-auto sm:max-w-sm"
    >
      <div class="bg-gradient-to-r from-indigo-500 to-purple-600 backdrop-blur-lg border border-white/20 rounded-xl shadow-2xl overflow-hidden">
        <div class="p-4">
          <div class="flex items-center gap-3">
            <!-- 更新图标 -->
            <div class="flex-shrink-0 w-8 h-8 bg-white/20 rounded-full flex items-center justify-center">
              <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
              </svg>
            </div>
            
            <!-- 消息内容 -->
            <div class="flex-1 min-w-0">
              <p class="text-sm font-medium text-white">
                新版本可用
              </p>
              <p class="text-xs text-white/80 mt-1">
                发现应用更新，立即获取最新功能
              </p>
            </div>
            
            <!-- 操作按钮 -->
            <div class="flex gap-2">
              <button
                @click="handleUpdate"
                class="px-3 py-1.5 bg-white/20 hover:bg-white/30 text-white text-sm font-medium rounded-lg transition-colors focus:outline-none focus:ring-2 focus:ring-white/50"
              >
                更新
              </button>
              <button
                @click="dismissUpdate"
                class="p-1.5 hover:bg-white/20 text-white rounded-lg transition-colors focus:outline-none focus:ring-2 focus:ring-white/50"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style scoped>
.update-banner-enter-active {
  transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

.update-banner-leave-active {
  transition: all 0.3s ease-in;
}

.update-banner-enter-from {
  transform: translateY(100%) scale(0.95);
  opacity: 0;
}

.update-banner-leave-to {
  transform: translateY(100%) scale(0.95);
  opacity: 0;
}
</style>