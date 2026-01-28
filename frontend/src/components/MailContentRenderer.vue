<script setup>
import { ref } from "vue";
import { useI18n } from 'vue-i18n'
import { CloudDownloadRound, ReplyFilled, ForwardFilled, FullscreenRound } from '@vicons/material'
import ShadowHtmlComponent from "./ShadowHtmlComponent.vue";
import AiExtractInfo from "./AiExtractInfo.vue";
import { getDownloadEmlUrl } from '../utils/email-parser';
import { utcToLocalDate } from '../utils';
import { useGlobalState } from '../store';

const { preferShowTextMail, useIframeShowMail, useUTCDate } = useGlobalState();

const { t } = useI18n({
  messages: {
    en: {
      delete: 'Delete',
      deleteMailTip: 'Are you sure you want to delete mail?',
      attachments: 'View Attachments',
      downloadMail: 'Download Mail',
      from: 'From',
      to: 'To',
      reply: 'Reply',
      forward: 'Forward',
      showTextMail: 'Show Text Mail',
      showHtmlMail: 'Show HTML Mail',
      saveToS3: 'Save to S3',
      size: 'Size',
      fullscreen: 'Fullscreen',
    },
    zh: {
      delete: '删除',
      deleteMailTip: '确定要删除邮件吗?',
      attachments: '查看附件',
      downloadMail: '下载邮件',
      from: '发件人',
      to: '收件人',
      reply: '回复',
      forward: '转发',
      showTextMail: '显示纯文本邮件',
      showHtmlMail: '显示HTML邮件',
      saveToS3: '保存到S3',
      size: '大小',
      fullscreen: '全屏',
    }
  }
});

const props = defineProps({
  mail: {
    type: Object,
    required: true
  },
  showEMailTo: {
    type: Boolean,
    default: true
  },
  enableUserDeleteEmail: {
    type: Boolean,
    default: false
  },
  showReply: {
    type: Boolean,
    default: false
  },
  showSaveS3: {
    type: Boolean,
    default: false
  },
  // 回调函数 props
  onDelete: {
    type: Function,
    default: () => { }
  },
  onReply: {
    type: Function,
    default: () => { }
  },
  onForward: {
    type: Function,
    default: () => { }
  },
  onSaveToS3: {
    type: Function,
    default: () => { }
  }
});

const showTextMail = ref(preferShowTextMail.value);
const showAttachments = ref(false);
const curAttachments = ref([]);
const attachmentLoding = ref(false);
const showFullscreen = ref(false);

const handleDelete = () => {
  props.onDelete();
};

const handleViewAttachments = () => {
  curAttachments.value = props.mail.attachments;
  showAttachments.value = true;
};

const handleReply = () => {
  props.onReply();
};

const handleForward = () => {
  props.onForward();
};


const handleSaveToS3 = async (filename, blob) => {
  attachmentLoding.value = true;
  try {
    await props.onSaveToS3(filename, blob);
  } finally {
    attachmentLoding.value = false;
  }
};

</script>

<template>
  <div class="mail-content-renderer">
    <div class="mail-header">
      <div class="mail-meta">
        <n-text depth="3" class="mail-meta-line">
          #{{ mail.id }} · {{ utcToLocalDate(mail.created_at, useUTCDate.value) }}
        </n-text>
        <div class="mail-meta-line">
          <n-text depth="3" class="mail-meta-key">{{ t('from') }}</n-text>
          <n-ellipsis class="mail-meta-value" :tooltip="true">
            {{ mail.source }}
          </n-ellipsis>
        </div>
        <div v-if="showEMailTo" class="mail-meta-line">
          <n-text depth="3" class="mail-meta-key">{{ t('to') }}</n-text>
          <n-ellipsis class="mail-meta-value" :tooltip="true">
            {{ mail.address }}
          </n-ellipsis>
        </div>
      </div>

      <n-flex align="center" justify="end" :wrap="true" class="mail-actions">
        <n-popconfirm v-if="enableUserDeleteEmail" @positive-click="handleDelete">
          <template #trigger>
            <n-button tertiary type="error" size="small">{{ t('delete') }}</n-button>
          </template>
          {{ t('deleteMailTip') }}
        </n-popconfirm>

        <n-button v-if="mail.attachments && mail.attachments.length > 0" size="small" tertiary type="info"
          @click="handleViewAttachments">
          {{ t('attachments') }}
        </n-button>

        <n-button tag="a" target="_blank" tertiary type="info" size="small" :download="mail.id + '.eml'"
          :href="getDownloadEmlUrl(mail.raw)">
          <template #icon>
            <n-icon :component="CloudDownloadRound" />
          </template>
          {{ t('downloadMail') }}
        </n-button>

        <n-button v-if="showReply" size="small" tertiary type="info" @click="handleReply">
          <template #icon>
            <n-icon :component="ReplyFilled" />
          </template>
          {{ t('reply') }}
        </n-button>

        <n-button v-if="showReply" size="small" tertiary type="info" @click="handleForward">
          <template #icon>
            <n-icon :component="ForwardFilled" />
          </template>
          {{ t('forward') }}
        </n-button>

        <n-button size="small" tertiary type="info" @click="showTextMail = !showTextMail">
          {{ showTextMail ? t('showHtmlMail') : t('showTextMail') }}
        </n-button>

        <n-button size="small" tertiary type="info" @click="showFullscreen = true">
          <template #icon>
            <n-icon :component="FullscreenRound" />
          </template>
          {{ t('fullscreen') }}
        </n-button>
      </n-flex>
    </div>

    <!-- AI 提取信息 -->
    <AiExtractInfo :metadata="mail.metadata" />

    <!-- 邮件内容 -->
    <div class="mail-content mail-safe">
      <pre v-if="showTextMail" class="mail-text">{{ mail.text }}</pre>
      <iframe v-else-if="useIframeShowMail" :srcdoc="mail.message" class="mail-iframe">
      </iframe>
      <ShadowHtmlComponent v-else :key="mail.id" :htmlContent="mail.message" class="mail-html" />
    </div>
  </div>

  <n-drawer v-model:show="showFullscreen" width="100%" placement="bottom" :trap-focus="false" :block-scroll="false"
    style="height: 100vh;">
    <n-drawer-content :title="mail.subject" closable>
      <div class="fullscreen-mail-content mail-safe">
        <pre v-if="showTextMail" class="mail-text">{{ mail.text }}</pre>
        <iframe v-else-if="useIframeShowMail" :srcdoc="mail.message" class="mail-iframe">
        </iframe>
        <ShadowHtmlComponent v-else :key="mail.id" :htmlContent="mail.message" class="mail-html" />
      </div>
    </n-drawer-content>
  </n-drawer>

  <!-- 附件模态框 -->
  <n-modal v-model:show="showAttachments" preset="dialog" title="Dialog">
    <template #header>
      <div>{{ t('attachments') }}</div>
    </template>
    <n-spin v-model:show="attachmentLoding">
      <n-list hoverable clickable>
        <n-list-item v-for="row in curAttachments" v-bind:key="row.id">
          <n-thing class="center" :title="row.filename">
            <template #description>
              <n-space>
                <n-tag type="info">
                  Size: {{ row.size }}
                </n-tag>
                <n-button v-if="showSaveS3" @click="handleSaveToS3(row.filename, row.blob)" ghost type="info"
                  size="small">
                  {{ t('saveToS3') }}
                </n-button>
              </n-space>
            </template>
          </n-thing>
          <template #suffix>
            <n-button tag="a" target="_blank" tertiary type="info" size="small" :download="row.filename"
              :href="row.url">
              <n-icon :component="CloudDownloadRound" />
            </n-button>
          </template>
        </n-list-item>
      </n-list>
    </n-spin>
  </n-modal>
</template>

<style scoped>
.mail-content-renderer {
  display: flex;
  flex-direction: column;
  gap: 10px;
  min-width: 0;
}

.mail-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
  min-width: 0;
}

.mail-meta {
  min-width: 0;
}

.mail-meta-line {
  display: flex;
  align-items: baseline;
  gap: 8px;
  min-width: 0;
  line-height: 1.35;
}

.mail-meta-key {
  flex: 0 0 auto;
  font-size: 12px;
  letter-spacing: 0.02em;
}

.mail-meta-value {
  min-width: 0;
  max-width: 100%;
}

.mail-actions {
  flex: 0 0 auto;
  gap: 8px;
}

.mail-content {
  margin-top: 10px;
  min-width: 0;
  overflow: auto;
}

.mail-text {
  white-space: pre-wrap;
  word-wrap: break-word;
  margin: 0;
  padding: 0;
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
}

.mail-iframe {
  width: 100%;
  height: 100%;
  border: none;
  min-height: 400px;
}

.mail-html {
  width: 100%;
  min-width: 0;
}

.center {
  text-align: center;
}

.fullscreen-mail-content {
  height: calc(100vh - 120px);
  overflow: auto;
}

.fullscreen-mail-content .mail-iframe {
  min-height: calc(100vh - 120px);
}
</style>
