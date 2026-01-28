<script setup>
import { ref } from 'vue'
import { useI18n } from 'vue-i18n'

import AppSection from '../../components/AppSection.vue'
import { useGlobalState } from '../../store'
import { api } from '../../api'
import SendBox from '../../components/SendBox.vue';

const { adminSendBoxTabAddress } = useGlobalState()

const { t } = useI18n({
    messages: {
        en: {
            title: 'Sendbox',
            query: 'Query',
            queryTip: 'Please input address to query, leave blank to query all',
            refresh: 'Refresh',
        },
        zh: {
            title: '发件箱',
            query: '查询',
            queryTip: '请输入地址查询, 留空则查询所有',
            refresh: '刷新',
        }
    }
});

const sendBoxKey = ref(0)

const querySendBox = () => {
    adminSendBoxTabAddress.value = adminSendBoxTabAddress.value.trim();
    sendBoxKey.value = Date.now();
}

const fetchData = async (limit, offset) => {
    const address = adminSendBoxTabAddress.value.trim();
    return await api.fetch(
        `/admin/sendbox?limit=${limit}&offset=${offset}`
        + (address ? `&address=${address}` : '')
    );
}

const deleteSenboxMail = async (curMailId) => {
    await api.fetch(`/admin/sendbox/${curMailId}`, { method: 'DELETE' });
};
</script>

<template>
    <AppSection :title="t('title')" :glass="false" class="admin-sendbox-section">
        <template #actions>
            <n-button size="small" tertiary @click="querySendBox">
                {{ t('refresh') }}
            </n-button>
        </template>

        <n-flex vertical size="small">
            <n-input-group class="admin-sendbox-query">
                <n-input v-model:value="adminSendBoxTabAddress" :placeholder="t('queryTip')"
                    @keydown.enter="querySendBox" />
                <n-button @click="querySendBox" type="primary" tertiary>
                    {{ t('query') }}
                </n-button>
            </n-input-group>

            <SendBox :key="sendBoxKey" class="admin-sendbox-list" :enableUserDeleteEmail="true"
                :deleteMail="deleteSenboxMail" :fetchMailData="fetchData" :showEMailFrom="true" />
        </n-flex>
    </AppSection>
</template>

<style scoped>
.admin-sendbox-query {
    max-width: 560px;
}
</style>
