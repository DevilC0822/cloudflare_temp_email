<script setup>
import '@wangeditor/editor/dist/css/style.css'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'
import { useI18n } from 'vue-i18n'
import { computed, onMounted, onBeforeUnmount, ref, shallowRef } from 'vue'
import AdminContact from '../common/AdminContact.vue'

import AppSection from '../../components/AppSection.vue'
import { useGlobalState } from '../../store'
import { api } from '../../api'

const message = useMessage()
const isPreview = ref(false)
const editorRef = shallowRef()


const { settings, sendMailModel, indexTab, userSettings } = useGlobalState()

const { t } = useI18n({
    locale: 'zh',
    messages: {
        en: {
            title: 'Send Mail',
            successSend: 'Please check your sendbox. If failed, please check your balance or try again later.',
            fromName: 'From',
            fromNamePlaceholder: 'Name (optional)',
            toName: 'To',
            toNamePlaceholder: 'Name (optional)',
            toMailPlaceholder: 'Email address',
            subject: 'Subject',
            options: 'Format',
            edit: 'Edit',
            preview: 'Preview',
            content: 'Body',
            send: 'Send',
            requestAccess: 'Request Access',
            requestAccessTip: 'You need to request access to send mail, if have request, please contact admin.',
            send_balance: 'Balance',
            text: 'Text',
            html: 'HTML',
            'rich text': 'Rich Text',
            tooLarge: 'Too large file, please upload file less than 1MB.',
        },
        zh: {
            title: '发送邮件',
            successSend: '请查看您的发件箱, 如果失败, 请检查您的余额或稍后重试。',
            fromName: '发件人',
            fromNamePlaceholder: '名称（可选）',
            toName: '收件人',
            toNamePlaceholder: '名称（可选）',
            toMailPlaceholder: '邮箱地址',
            subject: '主题',
            options: '格式',
            edit: '编辑',
            preview: '预览',
            content: '内容',
            send: '发送',
            requestAccess: '申请权限',
            requestAccessTip: '暂无发件额度，可申请或联系管理员。',
            send_balance: '剩余额度',
            text: '文本',
            html: 'HTML',
            'rich text': '富文本',
            tooLarge: '文件过大, 请上传小于1MB的文件。',
        }
    }
});

const contentTypes = [
    { label: t('text'), value: 'text' },
    { label: t('html'), value: 'html' },
    { label: t('rich text'), value: 'rich' },
]

const send = async () => {
    try {
        await api.fetch(`/api/send_mail`,
            {
                method: 'POST',
                body:
                    JSON.stringify({
                        from_name: sendMailModel.value.fromName,
                        to_name: sendMailModel.value.toName,
                        to_mail: sendMailModel.value.toMail,
                        subject: sendMailModel.value.subject,
                        is_html: sendMailModel.value.contentType != 'text',
                        content: sendMailModel.value.content,
                    })
            })
        sendMailModel.value = {
            fromName: "",
            toName: "",
            toMail: "",
            subject: "",
            contentType: 'text',
            content: "",
        }
    } catch (error) {
        message.error(error.message || "error");
        return
    }
    message.success(t("successSend"));
    indexTab.value = 'sendbox'
}

const requestAccess = async () => {
    try {
        await api.fetch(`/api/requset_send_mail_access`,
            {
                method: 'POST',
                body: JSON.stringify({})
            }
        )
        message.success(t("success"))
        await api.getSettings();
    } catch (error) {
        message.error(error.message || "error");
    }
}

const toolbarConfig = {
    excludeKeys: ["uploadVideo"]
}

const editorConfig = {
    MENU_CONF: {
        'uploadImage': {
            async customUpload() {
                message.error(t('tooLarge'))
            },
            maxFileSize: 1 * 1024 * 1024,
            base64LimitSize: 1 * 1024 * 1024,
        }
    }
}

onBeforeUnmount(() => {
    const editor = editorRef.value
    if (editor == null) return
    editor.destroy()
})

const handleCreated = (editor) => {
    editorRef.value = editor;
}

onMounted(async () => {
    // make sure user_id is fetched
    if (!userSettings.value.user_id) await api.getUserSettings(message);
    await api.getSettings();
})

const balanceText = computed(() => {
    const balance = settings.value?.send_balance;
    if (typeof balance !== 'number') return '';
    return `${t('send_balance')}：${balance}`;
})
</script>

<template>
    <div class="app-center" v-if="settings.address">
        <AppSection :title="t('title')" :description="balanceText" :glass="false" class="sendmail-section">
            <template #actions>
                <n-button v-if="settings.send_balance && settings.send_balance > 0" type="primary" size="small"
                    @click="send">
                    {{ t('send') }}
                </n-button>
            </template>

            <div v-if="!settings.send_balance || settings.send_balance <= 0">
                <n-alert type="warning" :show-icon="false" :bordered="false">
                    {{ t('requestAccessTip') }}
                    <n-button type="primary" tertiary @click="requestAccess" size="small">{{ t('requestAccess')
                        }}</n-button>
                </n-alert>
                <AdminContact />
            </div>
            <div v-else>
                <div class="sendmail-form">
                    <n-form :model="sendMailModel">
                        <n-form-item :label="t('fromName')" label-placement="top">
                            <n-input-group>
                                <n-input v-model:value="sendMailModel.fromName" :placeholder="t('fromNamePlaceholder')" />
                                <n-input :value="settings.address" disabled />
                            </n-input-group>
                        </n-form-item>
                        <n-form-item :label="t('toName')" label-placement="top">
                            <n-input-group>
                                <n-input v-model:value="sendMailModel.toName" :placeholder="t('toNamePlaceholder')" />
                                <n-input v-model:value="sendMailModel.toMail" :placeholder="t('toMailPlaceholder')" />
                            </n-input-group>
                        </n-form-item>
                        <n-form-item :label="t('subject')" label-placement="top">
                            <n-input v-model:value="sendMailModel.subject" />
                        </n-form-item>
                        <n-form-item :label="t('options')" label-placement="top">
                            <n-flex align="center" justify="space-between" class="sendmail-options">
                                <n-radio-group v-model:value="sendMailModel.contentType">
                                    <n-radio-button v-for="option in contentTypes" :key="option.value"
                                        :value="option.value" :label="option.label" />
                                </n-radio-group>
                                <n-button v-if="sendMailModel.contentType != 'text'" @click="isPreview = !isPreview"
                                    tertiary>
                                    {{ isPreview ? t('edit') : t('preview') }}
                                </n-button>
                            </n-flex>
                        </n-form-item>
                        <n-form-item :label="t('content')" label-placement="top">
                            <n-card :bordered="false" embedded v-if="isPreview">
                                <div v-html="sendMailModel.content" />
                            </n-card>
                            <div v-else-if="sendMailModel.contentType == 'rich'" class="sendmail-rich-editor">
                                <Toolbar class="sendmail-rich-toolbar" :defaultConfig="toolbarConfig"
                                    :editor="editorRef" mode="default" />
                                <Editor style="height: 500px; overflow-y: hidden;" v-model="sendMailModel.content"
                                    :defaultConfig="editorConfig" mode="default" @onCreated="handleCreated" />
                            </div>
                            <n-input v-else type="textarea" v-model:value="sendMailModel.content" :autosize="{
                                minRows: 3
                            }" />
                        </n-form-item>
                    </n-form>
                </div>
            </div>
        </AppSection>
    </div>
</template>

<style scoped>
.sendmail-section {
    width: min(900px, 100%);
}

.sendmail-form {
    text-align: left;
}

.sendmail-options {
    width: 100%;
}

.sendmail-rich-editor {
    border: 1px solid var(--app-border);
    border-radius: var(--app-radius-sm);
    overflow: hidden;
}

.sendmail-rich-toolbar {
    border-bottom: 1px solid var(--app-border);
}
</style>
