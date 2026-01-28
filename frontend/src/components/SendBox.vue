<script setup>
import { watch, onMounted, ref, computed } from "vue";
import { useMessage } from 'naive-ui'
import { useI18n } from 'vue-i18n'
import { useElementSize } from '@vueuse/core'
import { useGlobalState } from '../store'
import { useIsMobile } from '../utils/composables'
import { utcToLocalDate } from '../utils';

const message = useMessage()
const isMobile = useIsMobile()

const rootRef = ref(null)
const { width: rootWidth } = useElementSize(rootRef)

const LIST_MIN_PX = 320
const DETAIL_MIN_PX = 520
const SPLIT_MIN_TOTAL = LIST_MIN_PX + DETAIL_MIN_PX + 64

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
const showDetailDrawer = ref(false)
const showCode = ref(false)

const multiActionMode = ref(false)
const showMultiActionDelete = ref(false)
const multiActionDeleteProgress = ref({ percentage: 0, tip: '0/0' })

const useSplitView = computed(() => {
  if (isMobile.value) return false
  return (rootWidth.value || 0) >= SPLIT_MIN_TOTAL
})

const splitState = computed(() => {
  const width = rootWidth.value || 0
  if (!width) {
    return {
      min: 0.25,
      max: 0.75,
      size: mailboxSplitSize.value
    }
  }
  const min = Math.max(0.18, LIST_MIN_PX / width)
  const max = Math.min(0.82, (width - DETAIL_MIN_PX) / width)
  const size = Math.min(Math.max(mailboxSplitSize.value, min), max)
  return { min, max, size }
})

const { t } = useI18n({
  messages: {
    en: {
      success: 'Success',
      refresh: 'Refresh',
      showCode: 'Change View Original Code',
      pleaseSelectMail: "Please select a mail to view.",
      noSubject: 'No Subject',
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
      noSubject: '无主题',
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
    if (useSplitView.value && !curMail.value && data.value.length > 0) {
      curMail.value = data.value[0];
      showCode.value = false
    }
  } catch (error) {
    message.error(error.message || "error");
    console.error(error);
  }
};

const clickRow = async (row) => {
  curMail.value = row;
  showCode.value = false
  if (!useSplitView.value) {
    showDetailDrawer.value = true
  }
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
    showDetailDrawer.value = false
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

watch(useSplitView, (split) => {
  if (split) {
    showDetailDrawer.value = false
    if (!curMail.value && data.value.length > 0) {
      curMail.value = data.value[0]
      showCode.value = false
    }
    return
  }
  showDetailDrawer.value = false
})

watch(showDetailDrawer, (show) => {
  if (!show && !useSplitView.value) {
    curMail.value = null
  }
})

onMounted(async () => {
  await refresh();
});
</script>

<template>
  <div ref="rootRef">
    <div v-if="useSplitView" class="left">
      <div class="sendbox-toolbar">
        <div class="sendbox-toolbar__group">
          <template v-if="multiActionMode">
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
          </template>
          <template v-else>
            <n-button v-if="showMultiActionMode" @click="multiActionModeClick(true)" type="primary" tertiary>
              {{ t('multiAction') }}
            </n-button>
          </template>
        </div>
        <div class="sendbox-toolbar__group sendbox-toolbar__group--right">
          <n-pagination class="sendbox-pager" v-model:page="page" v-model:page-size="pageSize" :item-count="count"
            :page-sizes="[20, 50, 100]" show-size-picker />
          <n-button @click="refresh" type="primary" tertiary>
            {{ t('refresh') }}
          </n-button>
        </div>
      </div>
      <n-split class="sendbox-split" direction="horizontal" :max="splitState.max" :min="splitState.min"
        :size="splitState.size" :pane1-style="{ minWidth: '0' }" :pane2-style="{ minWidth: '0', padding: '8px' }"
        :on-update:size="onSpiltSizeChange">
        <template #1>
          <div class="mail-list-scroll">
            <n-list hoverable clickable>
              <n-list-item v-for="row in data" v-bind:key="row.id" @click="() => clickRow(row)"
                :class="mailItemClass(row)">
                <template #prefix v-if="multiActionMode">
                  <n-checkbox v-model:checked="row.checked" @click.stop />
                </template>
                <div class="send-row__content">
                  <div class="send-row__top">
                    <n-ellipsis class="send-row__subject" :tooltip="true">
                      {{ row.subject || t('noSubject') }}
                    </n-ellipsis>
                    <n-text depth="3" class="send-row__time">
                      {{ utcToLocalDate(row.created_at, useUTCDate) }}
                    </n-text>
                  </div>
                  <div class="send-row__meta">
                    <n-text depth="3" class="send-row__id">
                      #{{ row.id }}
                    </n-text>
                    <n-ellipsis class="send-row__to" :tooltip="true">
                      {{ showEMailFrom ? (row.address + " → " + row.to_mail) : row.to_mail }}
                    </n-ellipsis>
                  </div>
                </div>
              </n-list-item>
            </n-list>
          </div>
        </template>
        <template #2>
          <n-card :bordered="false" embedded v-if="curMail" class="mail-item mail-content-scroll sendbox-detail">
            <template #header>
              <n-ellipsis class="sendbox-detail__title" :tooltip="true">
                {{ curMail.subject || t('noSubject') }}
              </n-ellipsis>
            </template>
            <template #header-extra>
              <n-flex align="center" justify="end" :wrap="true" class="sendbox-detail__actions">
                <n-button size="small" tertiary type="info" @click="showCode = !showCode">
                  {{ t('showCode') }}
                </n-button>
                <n-popconfirm v-if="enableUserDeleteEmail" @positive-click="deleteMail">
                  <template #trigger>
                    <n-button tertiary type="error" size="small">{{ t('delete') }}</n-button>
                  </template>
                  {{ t('deleteMailTip') }}
                </n-popconfirm>
              </n-flex>
            </template>

            <div class="sendbox-detail__meta">
              <n-text depth="3" class="sendbox-detail__meta-line">
                #{{ curMail.id }} · {{ utcToLocalDate(curMail.created_at, useUTCDate) }}
              </n-text>
              <n-text depth="3" class="sendbox-detail__meta-line" v-if="showEMailFrom">
                <n-ellipsis style="max-width: 100%;" :tooltip="true">
                  FROM: {{ curMail.address }}
                </n-ellipsis>
              </n-text>
              <n-text depth="3" class="sendbox-detail__meta-line">
                <n-ellipsis style="max-width: 100%;" :tooltip="true">
                  TO: {{ curMail.to_mail }}
                </n-ellipsis>
              </n-text>
            </div>

            <div class="sendbox-detail__body mail-safe">
              <pre v-if="showCode">{{ curMail.raw }}</pre>
              <pre v-else-if="!curMail.is_html">{{ curMail.content }}</pre>
              <div v-else v-html="curMail.content" class="sendbox-detail__html"></div>
            </div>
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
        <n-pagination class="sendbox-pager sendbox-pager--compact" v-model:page="page" v-model:page-size="pageSize"
          :item-count="count" simple size="small" />
        <n-button @click="refresh" size="small" type="primary">
          {{ t('refresh') }}
        </n-button>
      </div>
      <div class="mail-list-scroll mail-list-scroll--mobile">
        <n-list hoverable clickable>
          <n-list-item v-for="row in data" v-bind:key="row.id" @click="() => clickRow(row)">
            <div class="send-row__content">
              <div class="send-row__top">
                <n-ellipsis class="send-row__subject" :tooltip="true">
                  {{ row.subject || t('noSubject') }}
                </n-ellipsis>
                <n-text depth="3" class="send-row__time">
                  {{ utcToLocalDate(row.created_at, useUTCDate) }}
                </n-text>
              </div>
              <div class="send-row__meta">
                <n-text depth="3" class="send-row__id">
                  #{{ row.id }}
                </n-text>
                <n-ellipsis class="send-row__to" :tooltip="true">
                  {{ showEMailFrom ? (row.address + " → " + row.to_mail) : row.to_mail }}
                </n-ellipsis>
              </div>
            </div>
          </n-list-item>
        </n-list>
      </div>
      <n-drawer v-model:show="showDetailDrawer" width="100%" placement="bottom" :trap-focus="false"
        :block-scroll="false"
        style="height: 80vh;">
        <n-drawer-content :title="curMail ? (curMail.subject || t('noSubject')) : ''" closable>
          <n-card :bordered="false" embedded style="overflow: auto;">
            <div class="sendbox-detail__drawer-head">
              <n-flex align="center" justify="end" :wrap="true" class="sendbox-detail__actions">
                <n-button size="small" tertiary type="info" @click="showCode = !showCode">
                  {{ t('showCode') }}
                </n-button>
                <n-popconfirm v-if="enableUserDeleteEmail" @positive-click="deleteMail">
                  <template #trigger>
                    <n-button tertiary type="error" size="small">{{ t('delete') }}</n-button>
                  </template>
                  {{ t('deleteMailTip') }}
                </n-popconfirm>
              </n-flex>
            </div>
            <div class="sendbox-detail__meta">
              <n-text depth="3" class="sendbox-detail__meta-line">
                #{{ curMail.id }} · {{ utcToLocalDate(curMail.created_at, useUTCDate) }}
              </n-text>
              <n-text depth="3" class="sendbox-detail__meta-line" v-if="showEMailFrom">
                <n-ellipsis style="max-width: 100%;" :tooltip="true">
                  FROM: {{ curMail.address }}
                </n-ellipsis>
              </n-text>
              <n-text depth="3" class="sendbox-detail__meta-line">
                <n-ellipsis style="max-width: 100%;" :tooltip="true">
                  TO: {{ curMail.to_mail }}
                </n-ellipsis>
              </n-text>
            </div>
            <div class="sendbox-detail__body mail-safe">
              <pre v-if="showCode">{{ curMail.raw }}</pre>
              <pre v-else-if="!curMail.is_html">{{ curMail.content }}</pre>
              <div v-else v-html="curMail.content" class="sendbox-detail__html"></div>
            </div>
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

.sendbox-split {
  min-width: 0;
}

.center {
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.mail-item {
  height: 100%;
}

.mail-list-scroll {
  overflow-y: auto;
  overflow-x: hidden;
  height: calc(100vh - 240px);
  padding: 2px;
  border-radius: var(--app-radius);
  background: var(--app-surface-2);
  border: 1px solid var(--app-border);
}

.sendbox-toolbar {
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  flex-wrap: wrap;
}

.sendbox-toolbar__group {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  min-width: 0;
}

.sendbox-toolbar__group--right {
  justify-content: flex-end;
  flex: 1 1 420px;
}

.sendbox-pager {
  display: inline-flex;
}

.sendbox-pager--compact {
  margin-right: 6px;
}

.mail-list-scroll--mobile {
  height: calc(100vh - 320px);
}

.send-row__content {
  width: 100%;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.send-row__top {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 10px;
  min-width: 0;
}

.send-row__subject {
  font-weight: 600;
  flex: 1 1 auto;
  min-width: 0;
  max-width: 100%;
}

.send-row__time {
  flex: 0 0 auto;
  font-size: 12px;
  white-space: nowrap;
}

.send-row__meta {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 0;
}

.send-row__id {
  flex: 0 0 auto;
  font-size: 12px;
  white-space: nowrap;
}

.send-row__to {
  flex: 1 1 auto;
  min-width: 0;
  max-width: 100%;
}

.mail-row {
  border-radius: 12px;
  cursor: pointer;
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

.sendbox-detail__actions {
  gap: 8px;
}

.sendbox-detail__title {
  min-width: 0;
  max-width: 100%;
  font-weight: 600;
}

.sendbox-detail__drawer-head {
  margin-bottom: 8px;
}

.sendbox-detail__meta {
  margin-bottom: 10px;
}

.sendbox-detail__meta-line {
  display: block;
  line-height: 1.35;
}

.sendbox-detail__body {
  min-width: 0;
}

.sendbox-detail__body pre {
  margin: 0;
}

.sendbox-detail__html {
  margin: 0;
}

:deep(.n-list),
:deep(.n-list-item__main) {
  min-width: 0;
}

:deep(.n-list-item__main) {
  width: 100%;
}

:deep(.n-ellipsis) {
  max-width: 100%;
}

:global(html.dark) .mail-row--active {
  background: rgba(96, 165, 250, 0.14);
}

pre {
  white-space: pre-wrap;
  word-wrap: break-word;
}
</style>
