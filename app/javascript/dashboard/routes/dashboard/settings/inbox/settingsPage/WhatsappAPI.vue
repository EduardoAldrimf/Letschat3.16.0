<template>
  <div class="iframe-container">
    <LoadingState
      v-if="iframeLoading"
      :message="$t('Carregando Integração...')"
      class="dashboard-app_loading-container"
    />
    
    <iframe
      v-show="!iframeLoading"
      :src="iframeUrl"
      frameborder="0"
      class="iframe-content"
      title="Chat Manager"
      @load="handleIframeLoad"
      @error="handleIframeError"
    ></iframe>
  </div>
</template>

<script>
import LoadingState from 'dashboard/components/widgets/LoadingState.vue';

export default {
  name: 'IframeComponent',
  components: {
    LoadingState,
  },
  data() {
    return {
      iframeLoading: true,
      iframeUrl: import.meta.env.VITE_MANAGER || 'https://whatsapp-nu-one.vercel.app',
      loadTimeout: null
    };
  },
  mounted() {
    // Fallback timeout caso o iframe não carregue
    this.loadTimeout = setTimeout(() => {
      this.iframeLoading = false;
    }, 5000); // 5 segundos de timeout
  },
  methods: {
    handleIframeLoad() {
      clearTimeout(this.loadTimeout);
      this.iframeLoading = false;
    },
    handleIframeError() {
      clearTimeout(this.loadTimeout);
      this.iframeLoading = false;
      console.error('Erro ao carregar o iframe');
    }
  },
  beforeUnmount() {
    clearTimeout(this.loadTimeout);
  }
};
</script>

<style scoped>
.iframe-container {
  position: relative;
  width: 100%;
  height: 100vh;
}

.iframe-content {
  width: 100%;
  height: 100%;
  border: none;
}

.dashboard-app_loading-container {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>