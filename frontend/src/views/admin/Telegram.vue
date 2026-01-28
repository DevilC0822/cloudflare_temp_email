<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useI18n } from 'vue-i18n'

import AppSection from '../../components/AppSection.vue'

// @ts-ignore
import { useGlobalState } from '../../store'
// @ts-ignore
import { api } from '../../api'
// @ts-ignore
const message = useMessage()

const { t } = useI18n({
    messages: {
        en: {
            title: 'Telegram',
            refresh: 'Refresh',
            init: 'Init',
            successTip: 'Success',
            status: 'Check Status',
            enableTelegramAllowList: 'Enable Telegram Allow List(Manually input Chat ID)',
            enable: 'Enable',
            telegramAllowList: 'Telegram Allow List(Manually input telegram Chat ID)',
            manualInputPrompt: 'Type and press Enter to add',
            save: 'Save',
            miniAppUrl: 'Telegram Mini App URL',
            enableGlobalMailPush: 'Enable Global Mail Push(Manually input telegram Chat ID)',
            globalMailPushList: 'Global Mail Push Chat ID List',
            globalMailPushListTip: 'Support chat_id of private chat/group/channel. You can send a message to your bot, then visit this link to see chat_id, https://api.telegram.org/bot<Replace with your BOT TOKEN>/getUpdates',
        },
        zh: {
            title: 'Telegram',
            refresh: '刷新',
            init: '初始化',
            successTip: '成功',
            status: '查看状态',
            enableTelegramAllowList: '启用 Telegram 白名单(手动输入 Chat ID, 回车增加)',
            enable: '启用',
            telegramAllowList: 'Telegram 白名单(手动输入 Chat ID, 回车增加)',
            manualInputPrompt: '输入后按回车键添加',
            save: '保存',
            miniAppUrl: '电报小程序 URL(请输入你部署的电报小程序网页地址)',
            enableGlobalMailPush: '启用全局邮件推送(手动输入邮箱管理员的 telegram Chat ID, 回车增加)',
            globalMailPushList: '全局邮件推送 Chat ID 列表',
            globalMailPushListTip: '支持对话/群组/频道的 Chat ID, 您可以发送一条消息给您的机器人，然后访问此链接来查看 chat_id, https://api.telegram.org/bot<这里替换成您的 BOT TOKEN>/getUpdates',
        }
    }
});

const status = ref({
    fetched: false,
})

const fetchStatus = async () => {
    try {
        const res = await api.fetch(`/admin/telegram/status`)
        Object.assign(status.value, res)
        status.value.fetched = true
    } catch (error) {
        message.error((error as Error).message || "error");
    }
}

const init = async () => {
    try {
        await api.fetch(`/admin/telegram/init`, {
            method: 'POST',
        })
        message.success(t('successTip'))
    } catch (error) {
        message.error((error as Error).message || "error");
    }
}

class TelegramSettings {
    enableAllowList: boolean;
    allowList: string[];
    miniAppUrl: string;
    enableGlobalMailPush: boolean;
    globalMailPushList: string[];

    constructor(
        enableAllowList: boolean, allowList: string[], miniAppUrl: string,
        enableGlobalMailPush: boolean, globalMailPushList: string[]
    ) {
        this.enableAllowList = enableAllowList;
        this.allowList = allowList;
        this.miniAppUrl = miniAppUrl;
        this.enableGlobalMailPush = enableGlobalMailPush;
        this.globalMailPushList = globalMailPushList;
    }
}

const settings = ref(new TelegramSettings(false, [], '', false, []))

const getSettings = async () => {
    try {
        const res = await api.fetch(`/admin/telegram/settings`)
        Object.assign(settings.value, res)
    } catch (error) {
        message.error((error as Error).message || "error");
    }
}

const saveSettings = async () => {
    try {
        await api.fetch(`/admin/telegram/settings`, {
            method: 'POST',
            body: JSON.stringify(settings.value),
        })
        message.success(t('successTip'))
    } catch (error) {
        message.error((error as Error).message || "error");
    }
}

onMounted(async () => {
    await getSettings();
})
</script>

<template>
    <div class="app-center">
        <AppSection :title="t('title')" :glass="false" class="telegram-settings">
            <template #actions>
                <n-button size="small" tertiary @click="getSettings">
                    {{ t('refresh') }}
                </n-button>
                <n-button size="small" tertiary @click="fetchStatus">
                    {{ t('status') }}
                </n-button>
                <n-button size="small" tertiary @click="init">
                    {{ t('init') }}
                </n-button>
                <n-button size="small" type="primary" @click="saveSettings">
                    {{ t('save') }}
                </n-button>
            </template>

            <n-space vertical size="medium" class="telegram-settings__form">
                <n-form-item-row :label="t('enableTelegramAllowList')">
                    <n-input-group>
                        <n-checkbox v-model:checked="settings.enableAllowList" style="width: 20%;">
                            {{ t('enable') }}
                        </n-checkbox>
                        <n-select v-model:value="settings.allowList" filterable multiple tag style="width: 80%;"
                            :placeholder="t('telegramAllowList')">
                            <template #empty>
                                <n-text depth="3">
                                    {{ t('manualInputPrompt') }}
                                </n-text>
                            </template>
                        </n-select>
                    </n-input-group>
                </n-form-item-row>

                <n-form-item-row :label="t('enableGlobalMailPush')">
                    <n-input-group>
                        <n-checkbox v-model:checked="settings.enableGlobalMailPush" style="width: 20%;">
                            {{ t('enable') }}
                        </n-checkbox>
                        <n-select v-model:value="settings.globalMailPushList" filterable multiple tag
                            style="width: 80%;" :placeholder="t('globalMailPushList')">
                            <template #empty>
                                <n-text depth="3">
                                    {{ t('manualInputPrompt') }}
                                </n-text>
                            </template>
                        </n-select>
                    </n-input-group>
                    <template #feedback>
                        <n-text depth="3">
                            {{ t('globalMailPushListTip') }}
                        </n-text>
                    </template>
                </n-form-item-row>

                <n-form-item-row :label="t('miniAppUrl')">
                    <n-input v-model:value="settings.miniAppUrl" />
                </n-form-item-row>
            </n-space>

            <pre v-if="status.fetched" class="telegram-settings__status">{{ JSON.stringify(status, null, 2) }}</pre>
        </AppSection>
    </div>
</template>

<style scoped>
.telegram-settings {
    width: min(980px, 100%);
    text-align: left;
}

.telegram-settings__status {
    font-family: var(--app-font-mono);
    font-size: 12px;
    padding: 10px 12px;
    border: 1px solid var(--app-border);
    border-radius: var(--app-radius-sm);
    background: var(--app-surface-subtle);
    overflow: auto;
}
</style>
