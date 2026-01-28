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

const { loading } = useGlobalState()

const { t } = useI18n({
    messages: {
        en: {
            title: 'Webhook Access',
            refresh: 'Refresh',
            successTip: 'Success',
            enableAllowList: 'Enable Allow List (Restrict webhook access to specific users)',
            webhookAllowList: 'Webhook Allow List(Enter the mail address that is allowed to use webhook and enter)',
            manualInputPrompt: 'Type and press Enter to add',
            save: 'Save',
            notEnabled: 'Webhook is not enabled',
        },
        zh: {
            title: 'Webhook 权限',
            refresh: '刷新',
            successTip: '成功',
            enableAllowList: '启用白名单 (限制 webhook 访问权限，只有白名单中的用户可以使用)',
            webhookAllowList: 'Webhook 白名单(请输入允许使用webhook 的邮箱地址, 回车增加)',
            manualInputPrompt: '输入后按回车键添加',
            save: '保存',
            notEnabled: 'Webhook 未开启',
        }
    }
});

class WebhookSettings {
    enableAllowList: boolean;
    allowList: string[];

    constructor(enableAllowList: boolean, allowList: string[]) {
        this.enableAllowList = enableAllowList;
        this.allowList = allowList;
    }
}

const webhookSettings = ref(new WebhookSettings(false, []))
const webhookEnabled = ref(false)
const errorInfo = ref('')

const getSettings = async () => {
    try {
        const res = await api.fetch(`/admin/webhook/settings`)
        Object.assign(webhookSettings.value, res)
        webhookEnabled.value = true
    } catch (error) {
        errorInfo.value = (error as Error).message || "error";
    }
}

const saveSettings = async () => {
    try {
        await api.fetch(`/admin/webhook/settings`, {
            method: 'POST',
            body: JSON.stringify(webhookSettings.value),
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
    <div v-if="webhookEnabled" class="app-center">
        <AppSection :title="t('title')" :glass="false" class="admin-webhook-settings">
            <template #actions>
                <n-button size="small" tertiary @click="getSettings" :loading="loading">
                    {{ t('refresh') }}
                </n-button>
                <n-button size="small" @click="saveSettings" type="primary" :loading="loading">
                    {{ t('save') }}
                </n-button>
            </template>

            <n-form-item-row :label="t('enableAllowList')">
                <n-switch v-model:value="webhookSettings.enableAllowList" :round="false" />
            </n-form-item-row>
            <n-form-item-row :label="t('webhookAllowList')">
                <n-select v-model:value="webhookSettings.allowList" filterable multiple tag
                    :placeholder="t('webhookAllowList')">
                    <template #empty>
                        <n-text depth="3">
                            {{ t('manualInputPrompt') }}
                        </n-text>
                    </template>
                </n-select>
            </n-form-item-row>
        </AppSection>
    </div>

    <div v-else class="app-center">
        <n-result status="404" :title="t('notEnabled')" :description="errorInfo" />
    </div>
</template>

<style scoped>
.admin-webhook-settings {
    width: min(900px, 100%);
    text-align: left;
}
</style>
