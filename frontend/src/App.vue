<script setup>
import { darkTheme, NGlobalStyle, zhCN } from 'naive-ui'
import { computed, onMounted } from 'vue'
import { useScript } from '@unhead/vue'
import { useI18n } from 'vue-i18n'
import { useGlobalState } from './store'
import { useIsMobile } from './utils/composables'
import Header from './views/Header.vue';
import Footer from './views/Footer.vue';
import { api } from './api'

const {
  isDark, loading, useSideMargin, telegramApp, isTelegram
} = useGlobalState()
const adClient = import.meta.env.VITE_GOOGLE_AD_CLIENT;
const adSlot = import.meta.env.VITE_GOOGLE_AD_SLOT;
const { locale } = useI18n({});
const theme = computed(() => isDark.value ? darkTheme : null)
const localeConfig = computed(() => locale.value == 'zh' ? zhCN : null)
const loadingText = computed(() => (locale.value === 'zh' ? '加载中…' : 'Loading…'))
const isMobile = useIsMobile()
const showSideMargin = computed(() => !isMobile.value && useSideMargin.value);
const showAd = computed(() => !isMobile.value && adClient && adSlot);

// Naive UI 主题覆盖：让全站组件风格更统一、更“高级”
const themeOverrides = computed(() => {
  const fontFamily = 'var(--app-font-sans)'
  const fontFamilyMono = 'var(--app-font-mono)'
  if (isDark.value) {
    return {
      common: {
        fontFamily,
        fontFamilyMono,
        borderRadius: '12px',
        primaryColor: '#60A5FA',
        primaryColorHover: '#93C5FD',
        primaryColorPressed: '#3B82F6',
        primaryColorSuppl: '#60A5FA',
        infoColor: '#38BDF8',
        successColor: '#22C55E',
        warningColor: '#F59E0B',
        errorColor: '#EF4444',
        textColorBase: '#E2E8F0',
        textColor1: '#E2E8F0',
        textColor2: 'rgba(226, 232, 240, 0.72)',
        textColor3: 'rgba(226, 232, 240, 0.56)',
        bodyColor: '#0B1220',
        cardColor: 'rgba(17, 25, 39, 0.72)',
        popoverColor: 'rgba(17, 25, 39, 0.86)',
        modalColor: 'rgba(17, 25, 39, 0.86)',
        borderColor: 'rgba(148, 163, 184, 0.16)',
        dividerColor: 'rgba(148, 163, 184, 0.14)',
        hoverColor: 'rgba(148, 163, 184, 0.10)',
        tableHeaderColor: 'rgba(17, 25, 39, 0.60)',
        closeIconColor: 'rgba(226, 232, 240, 0.72)',
        closeIconColorHover: '#E2E8F0',
        closeIconColorPressed: '#E2E8F0',
      }
    }
  }
  return {
    common: {
      fontFamily,
      fontFamilyMono,
      borderRadius: '12px',
      primaryColor: '#2563EB',
      primaryColorHover: '#1D4ED8',
      primaryColorPressed: '#1E40AF',
      primaryColorSuppl: '#2563EB',
      infoColor: '#0EA5E9',
      successColor: '#16A34A',
      warningColor: '#F59E0B',
      errorColor: '#EF4444',
      textColorBase: '#0F172A',
      textColor1: '#0F172A',
      textColor2: 'rgba(15, 23, 42, 0.68)',
      textColor3: 'rgba(15, 23, 42, 0.52)',
      bodyColor: '#F6F7FB',
      cardColor: 'rgba(255, 255, 255, 0.86)',
      popoverColor: '#FFFFFF',
      modalColor: '#FFFFFF',
      borderColor: 'rgba(15, 23, 42, 0.12)',
      dividerColor: 'rgba(15, 23, 42, 0.10)',
      hoverColor: 'rgba(15, 23, 42, 0.06)',
      tableHeaderColor: 'rgba(248, 250, 252, 0.92)',
      closeIconColor: 'rgba(15, 23, 42, 0.68)',
      closeIconColorHover: '#0F172A',
      closeIconColorPressed: '#0F172A',
    }
  }
})

// Load Google Ad script at top level (not inside onMounted)
if (showAd.value) {
  useScript({
    src: `https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=${adClient}`,
    async: true,
    crossorigin: "anonymous",
  })
}

onMounted(async () => {
  try {
    await api.getUserSettings();
  } catch (error) {
    console.error(error);
  }

  const token = import.meta.env.VITE_CF_WEB_ANALY_TOKEN;

  const exist = document.querySelector('script[src="https://static.cloudflareinsights.com/beacon.min.js"]') !== null
  if (token && !exist) {
    const script = document.createElement('script');
    script.defer = true;
    script.src = 'https://static.cloudflareinsights.com/beacon.min.js';
    script.dataset.cfBeacon = `{ token: ${token} }`;
    document.body.appendChild(script);
  }

  // check if google ad is enabled
  if (showAd.value) {
    (window.adsbygoogle = window.adsbygoogle || []).push({});
    (window.adsbygoogle = window.adsbygoogle || []).push({});
  }


  // check if telegram is enabled
  const enableTelegram = import.meta.env.VITE_IS_TELEGRAM;
  if (
    (typeof enableTelegram === 'boolean' && enableTelegram === true)
    ||
    (typeof enableTelegram === 'string' && enableTelegram === 'true')
  ) {
    await new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = 'https://telegram.org/js/telegram-web-app.js';
      script.onload = resolve;
      script.onerror = reject;
      document.body.appendChild(script);
    });
    telegramApp.value = window.Telegram?.WebApp || {};
    isTelegram.value = !!window.Telegram?.WebApp?.initData;
  }
});
</script>

<template>
  <n-config-provider :locale="localeConfig" :theme="theme" :theme-overrides="themeOverrides">
    <n-global-style />
    <n-spin :description="loadingText" :show="loading">
      <n-notification-provider container-style="margin-top: 76px;">
        <n-message-provider container-style="margin-top: 20px;">
          <div class="app-shell" :class="{ 'app-shell--with-ad': showAd }"
            :style="{ '--app-max-width': showSideMargin ? '1120px' : '1400px' }">
            <aside v-if="showAd" class="app-side">
              <div class="app-side-inner">
                <ins class="adsbygoogle" style="display:block" :data-ad-client="adClient" :data-ad-slot="adSlot"
                  data-ad-format="auto" data-full-width-responsive="true"></ins>
              </div>
            </aside>

            <main class="app-main">
              <Header />
              <div id="main-content" class="app-content" tabindex="-1">
                <div class="app-container">
                  <router-view></router-view>
                </div>
              </div>
              <Footer />
            </main>

            <aside v-if="showAd" class="app-side">
              <div class="app-side-inner">
                <ins class="adsbygoogle" style="display:block" :data-ad-client="adClient" :data-ad-slot="adSlot"
                  data-ad-format="auto" data-full-width-responsive="true"></ins>
              </div>
            </aside>
          </div>
          <n-back-top />
        </n-message-provider>
      </n-notification-provider>
    </n-spin>
  </n-config-provider>
</template>


<style scoped>
.app-shell {
  min-height: 100vh;
  display: grid;
  grid-template-columns: 1fr;
  align-items: start;
}

.app-shell--with-ad {
  grid-template-columns: minmax(180px, 240px) minmax(0, 1fr) minmax(180px, 240px);
  column-gap: 12px;
}

.app-side {
  position: sticky;
  top: 0;
  height: 100vh;
  padding: 12px 0;
  display: flex;
  justify-content: center;
}

.app-side-inner {
  width: 100%;
  max-width: 220px;
  border-radius: var(--app-radius);
  border: 1px solid var(--app-border);
  background: var(--app-surface);
  backdrop-filter: var(--app-backdrop);
  box-shadow: var(--app-shadow-sm);
  overflow: hidden;
}

.app-main {
  min-width: 0;
}

.app-content {
  padding: 16px 0 28px;
  text-align: left;
}
</style>
