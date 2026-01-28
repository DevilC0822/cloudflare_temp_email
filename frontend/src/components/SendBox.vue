<script setup>
import { watch, onMounted, ref, computed } from "vue";
import { useMessage } from 'naive-ui'
import { useI18n } from 'vue-i18n'
import { useGlobalState } from '../store'
import { useIsMobile } from '../utils/composables'
import { utcToLocalDate } from '../utils';

const message = useMessage()
const isMobile = useIsMobile()

const props = defineProps({
  enableUserDeleteEmail: {
    type: Boolean,
    default: false,
    required: false
  },
  showEMailFrom: {
    type: Boolean,
    default: false
  },
  fetchMailData: {
    type: Function,
    default: () => { },
    required: true
  },
  deleteMail: {
    type: Function,
    default: () => { },
    required: false
  },
})

const { isDark, mailboxSplitSize, loading, useUTCDate } = useGlobalState()
const data = ref([])

const count = ref(0)
const page = ref(1)
const pageSize = ref(20)

const curMail = ref(null);
const showCode = ref(false)

const multiActionMode = ref(false)
const showMultiActionDelete = ref(false)
const multiActionDeleteProgress = ref({ percentage: 0, tip: '0/0' })

const { t } = useI18n({
  messages: {
    en: {
      success: 'Success',
      refresh: 'Refresh',
      showCode: 'Change View Original Code',
      pleaseSelectMail: "Please select a mail to view.",
      delete: 'Delete',
      deleteMailTip: 'Are you sure you want to delete mail?',
      multiAction: 'Batch',
      cancelMultiAction: 'Exit Batch',
      selectAll: 'Select Page',
      unselectAll: 'Clear',
    },
    zh: {
      success: '成功',
      refresh: '刷新',
      showCode: '切换查看元数据',
      pleaseSelectMail: "请选择一封邮件查看。",
      delete: '删除',
      deleteMailTip: '确定要删除邮件吗?',
      multiAction: '批量',
      cancelMultiAction: '退出批量',
      selectAll: '全选本页',
      unselectAll: '清空',
    }
  }
});

watch([page, pageSize], async ([page, pageSize], [oldPage, oldPageSize]) => {
  if (page !== oldPage || pageSize !== oldPageSize) {
    await refresh();
  }
})

const refresh = async () => {
  try {
    const { results, count: totalCount } = await props.fetchMailData(
      pageSize.value, (page.value - 1) * pageSize.value
    );
    data.value = results.map((item) => {
      try {
        const data = JSON.parse(item.raw);
        if (data.version == "v2") {
          item.to_mail = data.to_name ? `${data.to_name} <${data.to_mail}>` : data.to_mail;
          item.subject = data.subject;
          item.is_html = data.is_html;
          item.content = data.content;
          item.raw = JSON.stringify(data, null, 2);
        } else {
          item.to_mail = data?.personalizations?.map(
            (p) => p.to?.map((t) => t.email).join(',')
          ).join(';');
          item.subject = data.subject;
          item.is_html = (data.content[0]?.type != 'text/plain');
          item.content = data.content[0]?.value;
          item.raw = JSON.stringify(data, null, 2);
        }
      } catch (error) {
        console.log(error);
      }
      return item;
    });
    if (totalCount > 0) {
      count.value = totalCount;
    }
    if (!isMobile.value && !curMail.value && data.value.length > 0) {
      curMail.value = data.value[0];
    }
  } catch (error) {
    message.error(error.message || "error");
    console.error(error);
  }
};

const clickRow = async (row) => {
  curMail.value = row;
};

const mailItemClass = (row) => {
  if (curMail.value && row.id == curMail.value.id) {
    return 'mail-row mail-row--active';
  }
  return 'mail-row';
};

const onSpiltSizeChange = (size) => {
  mailboxSplitSize.value = size;
}

const deleteMail = async () => {
  try {
    await props.deleteMail(curMail.value.id);
    message.success(t("success"));
    curMail.value = null;
    await refresh();
  } catch (error) {
    message.error(error.message || "error");
  }
};

const showMultiActionMode = computed(() => {
  return props.enableUserDeleteEmail;
});

const multiActionModeClick = (enableMulti) => {
  if (enableMulti) {
    data.value.forEach((item) => {
      item.checked = false;
    });
    multiActionMode.value = true;
  } else {
    multiActionMode.value = false;
    data.value.forEach((item) => {
      item.checked = false;
    });
  }
}

const multiActionSelectAll = (checked) => {
  data.value.forEach((item) => {
    item.checked = checked;
  });
}

const multiActionDeleteMail = async () => {
  try {
    loading.value = true;
    const selectedMails = data.value.filter((item) => item.checked);
    if (selectedMails.length === 0) {
      message.error(t('pleaseSelectMail'));
      return;
    }
    multiActionDeleteProgress.value = {
      percentage: 0,
      tip: `0/${selectedMails.length}`
    };
    for (const [index, mail] of selectedMails.entries()) {
      await props.deleteMail(mail.id);
      showMultiActionDelete.value = true;
      multiActionDeleteProgress.value = {
        percentage: Math.floor((index + 1) / selectedMails.length * 100),
        tip: `${index + 1}/${selectedMails.length}`
      };
    }
    message.success(t("success"));
    await refresh();
  } catch (error) {
    message.error(error.message || "error");
  } finally {
    loading.value = false;
    showMultiActionDelete.value = true;
  }
}

onMounted(async () => {
  await refresh();
});
</script>

<template>
  <div>
    <div v-if="!isMobile" class="left">
      <div class="sendbox-toolbar">
        <n-space v-if="multiActionMode">
          <n-button @click="multiActionModeClick(false)" tertiary>
            {{ t('cancelMultiAction') }}
          </n-button>
          <n-button @click="multiActionSelectAll(true)" tertiary>
            {{ t('selectAll') }}
          </n-button>
          <n-button @click="multiActionSelectAll(false)" tertiary>
            {{ t('unselectAll') }}
          </n-button>
          <n-popconfirm v-if="enableUserDeleteEmail" @positive-click="multiActionDeleteMail">
            <template #trigger>
              <n-button tertiary type="error">{{ t('delete') }}</n-button>
            </template>
            {{ t('deleteMailTip') }}
          </n-popconfirm>
        </n-space>
        <n-space v-else>
          <n-button v-if="showMultiActionMode" @click="multiActionModeClick(true)" type="primary" tertiary>
            {{ t('multiAction') }}
          </n-button>
          <div style="display: inline-block; margin-right: 10px;">
            <n-pagination v-model:page="page" v-model:page-size="pageSize" :item-count="count"
              :page-sizes="[20, 50, 100]" show-size-picker />
          </div>
          <n-button @click="refresh" type="primary" tertiary>
            {{ t('refresh') }}
          </n-button>
        </n-space>
      </div>
      <n-split direction="horizontal" :max="0.75" :min="0.25" :default-size="mailboxSplitSize"
        :on-update:size="onSpiltSizeChange">
        <template #1>
          <div class="mail-list-scroll">
            <n-list hoverable clickable>
              <n-list-item v-for="row in data" v-bind:key="row.id" @click="() => clickRow(row)"
                :class="mailItemClass(row)">
                <template #prefix v-if="multiActionMode">
                  <n-checkbox v-model:checked="row.checked" />
                </template>
                <n-thing :title="row.subject">
                  <template #description>
                    <n-text depth="3" class="sendbox-meta">
                      #{{ row.id }} · {{ utcToLocalDate(row.created_at, useUTCDate) }}
                    </n-text>
                    <n-text depth="3" class="sendbox-meta">
                      <n-ellipsis style="max-width: 520px;">
                        {{ showEMailFrom ? (row.address + " → " + row.to_mail) : row.to_mail }}
                      </n-ellipsis>
                    </n-text>
                  </template>
                </n-thing>
              </n-list-item>
            </n-list>
          </div>
        </template>
        <template #2>
          <n-card :bordered="false" embedded v-if="curMail" class="mail-item mail-content-scroll" :title="curMail.subject">
            <n-space>
              <n-tag type="info">
                ID: {{ curMail.id }}
              </n-tag>
              <n-tag type="info">
                {{ utcToLocalDate(curMail.created_at, useUTCDate) }}
              </n-tag>
              <n-tag type="info">
                FROM: {{ curMail.address }}
              </n-tag>
              <n-tag type="info">
                TO: {{ curMail.to_mail }}
              </n-tag>
              <n-button size="small" tertiary type="info" @click="showCode = !showCode">
                {{ t('showCode') }}
              </n-button>
              <n-popconfirm v-if="enableUserDeleteEmail" @positive-click="deleteMail">
                <template #trigger>
                  <n-button tertiary type="error" size="small">{{ t('delete') }}</n-button>
                </template>
                {{ t('deleteMailTip') }}
              </n-popconfirm>
            </n-space>
            <pre v-if="showCode" style="margin-top: 10px;">{{ curMail.raw }}</pre>
            <pre v-else-if="!curMail.is_html" style="margin-top: 10px;">{{ curMail.content }}</pre>
            <div v-else v-html="curMail.content" style="margin-top: 10px;"></div>
          </n-card>
          <n-card :bordered="false" embedded class="mail-item" v-else>
            <n-result status="info" :title="t('pleaseSelectMail')">
            </n-result>
          </n-card>
        </template>
      </n-split>
    </div>
    <div class="left" v-else>
      <div class="center">
        <div style="display: inline-block; margin-right: 10px;">
          <n-pagination v-model:page="page" v-model:page-size="pageSize" :item-count="count" simple size="small" />
        </div>
        <n-button @click="refresh" size="small" type="primary">
          {{ t('refresh') }}
        </n-button>
      </div>
      <div class="mail-list-scroll mail-list-scroll--mobile">
        <n-list hoverable clickable>
          <n-list-item v-for="row in data" v-bind:key="row.id" @click="() => clickRow(row)">
            <n-thing :title="row.subject">
              <template #description>
                <n-text depth="3" class="sendbox-meta">
                  #{{ row.id }} · {{ utcToLocalDate(row.created_at, useUTCDate) }}
                </n-text>
                <n-text depth="3" class="sendbox-meta">
                  <n-ellipsis style="max-width: 520px;">
                    {{ showEMailFrom ? (row.address + " → " + row.to_mail) : row.to_mail }}
                  </n-ellipsis>
                </n-text>
              </template>
            </n-thing>
          </n-list-item>
        </n-list>
      </div>
      <n-drawer v-model:show="curMail" width="100%" placement="bottom" :trap-focus="false" :block-scroll="false"
        style="height: 80vh;">
        <n-drawer-content :title="curMail ? curMail.subject : ''" closable>
          <n-card :bordered="false" embedded style="overflow: auto;">
            <n-space>
              <n-tag type="info">
                ID: {{ curMail.id }}
              </n-tag>
              <n-tag type="info">
                {{ utcToLocalDate(curMail.created_at, useUTCDate) }}
              </n-tag>
              <n-tag type="info">
                FROM: {{ curMail.address }}
              </n-tag>
              <n-tag type="info">
                TO: {{ curMail.to_mail }}
              </n-tag>
              <n-button size="small" tertiary type="info" @click="showCode = !showCode">
                {{ t('showCode') }}
              </n-button>
              <n-popconfirm v-if="enableUserDeleteEmail" @positive-click="deleteMail">
                <template #trigger>
                  <n-button tertiary type="error" size="small">{{ t('delete') }}</n-button>
                </template>
                {{ t('deleteMailTip') }}
              </n-popconfirm>
            </n-space>
            <pre v-if="showCode" style="margin-top: 10px;">{{ curMail.raw }}</pre>
            <pre v-else-if="!curMail.is_html" style="margin-top: 10px;">{{ curMail.content }}</pre>
            <div v-else v-html="curMail.content" style="margin-top: 10px;"></div>
          </n-card>
        </n-drawer-content>
      </n-drawer>
    </div>
  </div>
</template>

<style scoped>
.left {
  text-align: left;
}

.center {
  text-align: center;
}

.mail-item {
  height: 100%;
}

.mail-list-scroll {
  overflow: auto;
  height: calc(100vh - 240px);
  padding: 2px;
  border-radius: var(--app-radius);
  background: var(--app-surface-2);
  border: 1px solid var(--app-border);
}

.sendbox-toolbar {
  margin-bottom: 10px;
}

.sendbox-meta {
  display: block;
  line-height: 1.3;
}

.mail-list-scroll--mobile {
  height: calc(100vh - 320px);
}

.mail-row {
  border-radius: 12px;
  transition: background var(--app-duration) var(--app-ease), border-color var(--app-duration) var(--app-ease);
}

.mail-row--active {
  background: rgba(37, 99, 235, 0.10);
  border: 1px solid var(--app-border-strong);
}

.mail-content-scroll {
  overflow: auto;
  max-height: calc(100vh - 240px);
}

:global(html.dark) .mail-row--active {
  background: rgba(96, 165, 250, 0.14);
}

pre {
  white-space: pre-wrap;
  word-wrap: break-word;
}
</style>
