<script setup>
import { ref, onMounted, watch } from 'vue';
import { useI18n } from 'vue-i18n'

import AppSection from '../../components/AppSection.vue'
import { useGlobalState } from '../../store'
import { api } from '../../api'
import MailBox from '../../components/MailBox.vue';

const { adminMailTabAddress } = useGlobalState()

const { t } = useI18n({
    messages: {
        en: {
            title: 'Mails',
            addressQueryTip: 'Leave blank to query all addresses',
            query: 'Query',
            refresh: 'Refresh',
        },
        zh: {
            title: '邮件',
            addressQueryTip: '留空查询所有地址',
            query: '查询',
            refresh: '刷新',
        }
    }
});

const mailBoxKey = ref("")

const queryMail = () => {
    adminMailTabAddress.value = adminMailTabAddress.value.trim();
    mailBoxKey.value = Date.now();
}

const fetchMailData = async (limit, offset) => {
    return await api.fetch(
        `/admin/mails`
        + `?limit=${limit}`
        + `&offset=${offset}`
        + (adminMailTabAddress.value ? `&address=${adminMailTabAddress.value}` : '')
    );
}

const deleteMail = async (curMailId) => {
    await api.fetch(`/admin/mails/${curMailId}`, { method: 'DELETE' });
};
</script>

<template>
    <AppSection :title="t('title')" :glass="false" class="admin-mails-section">
        <template #actions>
            <n-button size="small" tertiary @click="queryMail">
                {{ t('refresh') }}
            </n-button>
        </template>

        <n-flex vertical size="small">
            <n-input-group class="admin-mails-query">
                <n-input v-model:value="adminMailTabAddress" :placeholder="t('addressQueryTip')"
                    @keydown.enter="queryMail" clearable />
                <n-button @click="queryMail" type="primary" tertiary>
                    {{ t('query') }}
                </n-button>
            </n-input-group>

            <MailBox :key="mailBoxKey" :enableUserDeleteEmail="true" :fetchMailData="fetchMailData"
                :deleteMail="deleteMail" :showFilterInput="true" />
        </n-flex>
    </AppSection>
</template>

<style scoped>
.admin-mails-query {
    max-width: 560px;
}
</style>
