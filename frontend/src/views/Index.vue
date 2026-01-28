<script setup>
import { computed, defineAsyncComponent, onMounted, ref, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute, useRouter } from 'vue-router'

import { useGlobalState } from '../store'
import { api } from '../api'
import { useIsMobile } from '../utils/composables'
import { FullscreenExitOutlined } from '@vicons/material'

import AddressBar from './index/AddressBar.vue';
import AppSection from '../components/AppSection.vue';
import MailBox from '../components/MailBox.vue';
import SendBox from '../components/SendBox.vue';
import AutoReply from './index/AutoReply.vue';
import AccountSettings from './index/AccountSettings.vue';
import Appearance from './common/Appearance.vue';
import Webhook from './index/Webhook.vue';
import Attachment from './index/Attachment.vue';
import About from './common/About.vue';
import SimpleIndex from './index/SimpleIndex.vue';

const { loading, settings, openSettings, indexTab, globalTabplacement, useSimpleIndex } = useGlobalState()
const message = useMessage()
const route = useRoute()
const router = useRouter()
const isMobile = useIsMobile()

const SendMail = defineAsyncComponent(() => {
  loading.value = true;
  return import('./index/SendMail.vue')
    .finally(() => loading.value = false);
});

const { t } = useI18n({
  messages: {
    en: {
      mailbox: 'Inbox',
      sendbox: 'Outbox',
      sendmail: 'Compose',
      auto_reply: 'Auto Reply',
      accountSettings: 'Account',
      appearance: 'Appearance',
      about: 'About',
      s3Attachment: 'Attachments',
      saveToS3Success: 'Saved to S3',
      webhookSettings: 'Webhook',
      query: 'Query',
      refresh: 'Refresh',
      enterSimpleMode: 'Simple Mode',
    },
    zh: {
      mailbox: '收件',
      sendbox: '发件',
      sendmail: '写信',
      auto_reply: '自动回复',
      accountSettings: '账户',
      appearance: '外观',
      about: '关于',
      s3Attachment: '附件',
      saveToS3Success: '已保存到 S3',
      webhookSettings: 'Webhook',
      query: '查询',
      refresh: '刷新',
      enterSimpleMode: '极简模式',
    }
  }
});

const fetchMailData = async (limit, offset) => {
  if (mailIdQuery.value > 0) {
    const singleMail = await api.fetch(`/api/mail/${mailIdQuery.value}`);
    if (singleMail) return { results: [singleMail], count: 1 };
    return { results: [], count: 0 };
  }
  return await api.fetch(`/api/mails?limit=${limit}&offset=${offset}`);
};

const deleteMail = async (curMailId) => {
  await api.fetch(`/api/mails/${curMailId}`, { method: 'DELETE' });
};

const deleteSenboxMail = async (curMailId) => {
  await api.fetch(`/api/sendbox/${curMailId}`, { method: 'DELETE' });
};

const fetchSenboxData = async (limit, offset) => {
  return await api.fetch(`/api/sendbox?limit=${limit}&offset=${offset}`);
};

const saveToS3 = async (mail_id, filename, blob) => {
  try {
    const { url } = await api.fetch(`/api/attachment/put_url`, {
      method: 'POST',
      body: JSON.stringify({ key: `${mail_id}/${filename}` })
    });
    // upload to s3 by formdata
    const formData = new FormData();
    formData.append(filename, blob);
    await fetch(url, {
      method: 'PUT',
      body: formData
    });
    message.success(t('saveToS3Success'));
  } catch (error) {
    console.error(error);
    message.error(error.message || "save to s3 error");
  }
}

const mailBoxKey = ref("")
const mailIdQuery = ref("")
const showMailIdQuery = ref(false)

const indexTabType = computed(() => {
  const placement = globalTabplacement.value;
  return placement === 'top' || placement === 'bottom' ? 'segment' : 'bar';
});

const getAvailableTabs = () => {
  const tabs = new Set(['mailbox', 'sendbox', 'sendmail', 'accountSettings', 'appearance']);
  if (openSettings.value.enableAutoReply) tabs.add('auto_reply');
  if (openSettings.value.enableWebhook) tabs.add('webhook');
  if (openSettings.value.isS3Enabled) tabs.add('s3_attachment');
  if (openSettings.value.enableIndexAbout) tabs.add('about');
  return tabs;
};

const syncIndexTabFromRoute = () => {
  // mail_id 查询场景优先展示收件箱，避免用户打开链接看不到结果
  if (route.query.mail_id) return;
  const tab = route.query.tab;
  if (typeof tab !== 'string') return;
  const available = getAvailableTabs();
  if (available.has(tab)) indexTab.value = tab;
};

const queryMail = () => {
  mailBoxKey.value = Date.now();
}

const sendBoxKey = ref(0)
const refreshSendBox = () => {
  sendBoxKey.value = Date.now();
}

watch(() => route.query.tab, () => {
  syncIndexTabFromRoute();
});

watch(
  () => [
    openSettings.value.enableAutoReply,
    openSettings.value.enableWebhook,
    openSettings.value.isS3Enabled,
    openSettings.value.enableIndexAbout,
  ],
  () => {
    syncIndexTabFromRoute();
  }
);

watch(indexTab, async (tab) => {
  if (!tab) return;
  if (route.query.tab === tab) return;
  await router.replace({ query: { ...route.query, tab } });
});

watch(route, () => {
  if (!route.query.mail_id) {
    showMailIdQuery.value = false;
    mailIdQuery.value = "";
    queryMail();
  }
})

onMounted(() => {
  if (route.query.mail_id) {
    showMailIdQuery.value = true;
    mailIdQuery.value = route.query.mail_id;
    indexTab.value = 'mailbox';
    queryMail();
  }
  syncIndexTabFromRoute();
})
</script>

<template>
  <section class="app-page">
    <div v-if="useSimpleIndex" class="index-simple">
      <SimpleIndex />
    </div>

    <div v-else class="index-normal">
      <AddressBar />

      <div v-if="settings.address" class="app-panel app-glass">
        <n-tabs :type="indexTabType" v-model:value="indexTab" :placement="globalTabplacement" animated>
          <template #prefix v-if="!isMobile">
            <n-button @click="useSimpleIndex = true" tertiary size="small">
              <template #icon>
                <n-icon>
                  <FullscreenExitOutlined />
                </n-icon>
              </template>
              {{ t('enterSimpleMode') }}
            </n-button>
          </template>

          <n-tab-pane name="mailbox" :tab="t('mailbox')">
            <AppSection :title="t('mailbox')" :glass="false" class="index-mailbox-section">
              <template #actions>
                <n-button size="small" tertiary @click="queryMail">
                  {{ t('refresh') }}
                </n-button>
              </template>

              <div v-if="showMailIdQuery" class="mail-query">
                <n-input-group>
                  <n-input v-model:value="mailIdQuery" />
                  <n-button @click="queryMail" type="primary" tertiary>
                    {{ t('query') }}
                  </n-button>
                </n-input-group>
              </div>
              <MailBox :key="mailBoxKey" :showEMailTo="false" :showReply="true" :showSaveS3="openSettings.isS3Enabled"
                :saveToS3="saveToS3" :enableUserDeleteEmail="openSettings.enableUserDeleteEmail"
                :fetchMailData="fetchMailData" :deleteMail="deleteMail" :showFilterInput="true" />
            </AppSection>
          </n-tab-pane>

          <n-tab-pane name="sendbox" :tab="t('sendbox')">
            <AppSection :title="t('sendbox')" :glass="false" class="index-sendbox-section">
              <template #actions>
                <n-button size="small" tertiary @click="refreshSendBox">
                  {{ t('refresh') }}
                </n-button>
              </template>

              <SendBox :key="sendBoxKey" :fetchMailData="fetchSenboxData"
                :enableUserDeleteEmail="openSettings.enableUserDeleteEmail" :deleteMail="deleteSenboxMail" />
            </AppSection>
          </n-tab-pane>

          <n-tab-pane name="sendmail" :tab="t('sendmail')">
            <SendMail />
          </n-tab-pane>

          <n-tab-pane name="accountSettings" :tab="t('accountSettings')">
            <AccountSettings />
          </n-tab-pane>

          <n-tab-pane name="appearance" :tab="t('appearance')">
            <Appearance :showUseSimpleIndex="true" />
          </n-tab-pane>

          <n-tab-pane v-if="openSettings.enableAutoReply" name="auto_reply" :tab="t('auto_reply')">
            <AutoReply />
          </n-tab-pane>

          <n-tab-pane v-if="openSettings.enableWebhook" name="webhook" :tab="t('webhookSettings')">
            <Webhook />
          </n-tab-pane>

          <n-tab-pane v-if="openSettings.isS3Enabled" name="s3_attachment" :tab="t('s3Attachment')">
            <Attachment />
          </n-tab-pane>

          <n-tab-pane v-if="openSettings.enableIndexAbout" name="about" :tab="t('about')">
            <About />
          </n-tab-pane>
        </n-tabs>
      </div>
    </div>
  </section>
</template>

<style scoped>
.mail-query {
  margin-bottom: 10px;
}
</style>
