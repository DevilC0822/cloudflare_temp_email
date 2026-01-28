<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useI18n } from 'vue-i18n'

import AppSection from './AppSection.vue'

const props = defineProps({
    title: {
        type: String,
        default: ''
    },
    glass: {
        type: Boolean,
        default: false
    },
    fetchData: {
        type: Function,
        default: () => { },
        required: true
    },
    saveSettings: {
        type: Function,
        default: (webhookSettings: WebhookSettings) => { },
        required: true
    },
    testSettings: {
        type: Function,
        default: (webhookSettings: WebhookSettings) => { },
        required: true
    },
})

// @ts-ignore
const message = useMessage()

const { t } = useI18n({
    messages: {
        en: {
            title: 'Webhook',
            successTip: 'Success',
            test: 'Test',
            save: 'Save',
            refresh: 'Refresh',
            notEnabled: 'Webhook is not enabled for you',
            urlMissing: 'URL is required',
            enable: 'Enable',
            messagePusherDemo: 'Fill with Message Pusher Demo',
            messagePusherDoc: 'Message Pusher Doc',
            fillInDemoTip: 'Please modify the URL and other settings to your own',
        },
        zh: {
            title: 'Webhook',
            successTip: '成功',
            test: '测试',
            save: '保存',
            refresh: '刷新',
            notEnabled: 'Webhook 未开启，请联系管理员开启',
            urlMissing: 'URL 不能为空',
            enable: '启用',
            messagePusherDemo: '填入MessagePusher示例',
            messagePusherDoc: 'MessagePusher文档',
            fillInDemoTip: '请修改URL和其他设置为您自己的配置',
        }
    }
});

class WebhookSettings {
    enabled: boolean = false
    url: string = ''
    method: string = 'POST'
    headers: string = JSON.stringify({}, null, 2)
    body: string = JSON.stringify({}, null, 2)
}

const messagePusherDocLink = "https://github.com/songquanpeng/message-pusher";

const messagePusherDemo = {
    enabled: true,
    url: 'https://msgpusher.com/push/username',
    method: 'POST',
    headers: JSON.stringify({
        'Content-Type': 'application/json',
    }, null, 2),
    body: JSON.stringify({
        "token": "token",
        "title": "${subject}",
        "description": "${subject}",
        "content": "*${subject}*\n\nFrom: ${from}\nTo: ${to}\n\n${parsedText}\n"
    }, null, 2),
} as WebhookSettings;

const fillMessagePuhserDemo = () => {
    Object.assign(webhookSettings.value, messagePusherDemo)
    message.success(t('fillInDemoTip'))
}

const webhookSettings = ref<WebhookSettings>(new WebhookSettings())
const enableWebhook = ref(false)

const fetchData = async () => {
    try {
        const res = await props.fetchData()
        Object.assign(webhookSettings.value, res)
        enableWebhook.value = true
    } catch (error) {
        message.error((error as Error).message || "error");
    }
}

const saveSettings = async () => {
    if (!webhookSettings.value.url) {
        message.error(t('urlMissing'))
        return
    }
    try {
        await props.saveSettings(webhookSettings.value)
        message.success(t('successTip'))
    } catch (error) {
        message.error((error as Error).message || "error");
    }
}

const testSettings = async () => {
    if (!webhookSettings.value.url) {
        message.error(t('urlMissing'))
        return
    }
    try {
        await props.testSettings(webhookSettings.value)
        message.success(t('successTip'))
    } catch (error) {
        message.error((error as Error).message || "error");
    }
}

onMounted(async () => {
    await fetchData();
})
</script>

<template>
    <div v-if="enableWebhook" class="app-center">
        <AppSection :title="props.title || t('title')" :glass="glass" class="webhook-component">
            <template #actions>
                <n-button size="small" tertiary @click="fetchData">
                    {{ t('refresh') }}
                </n-button>
                <n-button size="small" tertiary tag="a" :href="messagePusherDocLink" target="_blank">
                    {{ t('messagePusherDoc') }}
                </n-button>
                <n-button size="small" tertiary @click="fillMessagePuhserDemo">
                    {{ t('messagePusherDemo') }}
                </n-button>
                <n-button size="small" tertiary v-if="webhookSettings.enabled" @click="testSettings">
                    {{ t('test') }}
                </n-button>
                <n-button size="small" type="primary" @click="saveSettings">
                    {{ t('save') }}
                </n-button>
            </template>

            <n-form-item-row :label="t('enable')">
                <n-switch v-model:value="webhookSettings.enabled" :round="false" />
            </n-form-item-row>

            <div v-if="webhookSettings.enabled">
                <n-form-item-row label="URL">
                    <n-input v-model:value="webhookSettings.url" />
                </n-form-item-row>
                <n-form-item-row label="METHOD">
                    <n-select v-model:value="webhookSettings.method" tag :options="[
                        { label: 'POST', value: 'POST' },
                    ]" />
                </n-form-item-row>
                <n-form-item-row label="HEADERS">
                    <n-input v-model:value="webhookSettings.headers" type="textarea" :autosize="{ minRows: 3 }" />
                </n-form-item-row>
                <n-form-item-row label="BODY">
                    <n-input v-model:value="webhookSettings.body" type="textarea" :autosize="{ minRows: 3 }" />
                </n-form-item-row>
            </div>
        </AppSection>
    </div>

    <div v-else class="app-center">
        <n-result status="404" :title="t('notEnabled')" />
    </div>
</template>

<style scoped>
.webhook-component {
    width: min(900px, 100%);
}
</style>
