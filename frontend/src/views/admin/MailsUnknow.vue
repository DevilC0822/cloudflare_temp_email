<script setup>
import { useI18n } from 'vue-i18n'

import AppSection from '../../components/AppSection.vue'
import { api } from '../../api'
import MailBox from '../../components/MailBox.vue';

const { t } = useI18n({
    messages: {
        en: {
            title: 'Mails Without Recipient',
        },
        zh: {
            title: '无收件人邮件',
        }
    }
})

const fetchMailUnknowData = async (limit, offset) => {
    return await api.fetch(
        `/admin/mails_unknow`
        + `?limit=${limit}`
        + `&offset=${offset}`
    );
}

const deleteMail = async (curMailId) => {
    await api.fetch(`/admin/mails/${curMailId}`, { method: 'DELETE' });
};
</script>

<template>
    <AppSection :title="t('title')" :glass="false">
        <MailBox :enableUserDeleteEmail="true" :fetchMailData="fetchMailUnknowData" :deleteMail="deleteMail" />
    </AppSection>
</template>
