<script setup>
import { onMounted, ref, watch } from 'vue';
import { useI18n } from 'vue-i18n'

import AppSection from '../../components/AppSection.vue'
import { api } from '../../api'
import MailBox from '../../components/MailBox.vue';

const message = useMessage()

const { t } = useI18n({
    messages: {
        en: {
            title: 'Mailbox',
            refresh: 'Refresh',
            addressQueryTip: 'Filter address (optional)',
            query: 'Apply',
        },
        zh: {
            title: '收件箱',
            refresh: '刷新',
            addressQueryTip: '地址筛选（可选）',
            query: '筛选',
        }
    }
});

const mailBoxKey = ref("")
const addressFilter = ref();
const addressFilterOptions = ref([]);

const queryMail = () => {
    addressFilter.value = addressFilter.value ? addressFilter.value.trim() : addressFilter.value;
    mailBoxKey.value = Date.now();
}

const fetchMailData = async (limit, offset) => {
    return await api.fetch(
        `/user_api/mails`
        + `?limit=${limit}`
        + `&offset=${offset}`
        + (addressFilter.value ? `&address=${addressFilter.value}` : '')
    );
}

const fetchAddresData = async () => {
    try {
        const { results } = await api.fetch(
            `/user_api/bind_address`
        );
        addressFilterOptions.value = results.map((item) => {
            return {
                label: item.name,
                value: item.name
            }
        });
    } catch (error) {
        console.log(error)
        message.error(error.message || "error");
    }
}

const deleteMail = async (curMailId) => {
    await api.fetch(`/user_api/mails/${curMailId}`, { method: 'DELETE' });
};

watch(addressFilter, async (newValue) => {
    queryMail();
});

onMounted(() => {
    fetchAddresData();
});
</script>

<template>
    <AppSection :title="t('title')" :glass="false" class="user-mailbox-section">
        <template #actions>
            <n-button size="small" tertiary @click="queryMail">
                {{ t('refresh') }}
            </n-button>
        </template>

        <n-flex align="center" class="user-mailbox-filter" :wrap="true">
            <n-select v-model:value="addressFilter" :options="addressFilterOptions" clearable
                :placeholder="t('addressQueryTip')" class="user-mailbox-select" />
            <n-button @click="queryMail" type="primary" tertiary>
                {{ t('query') }}
            </n-button>
        </n-flex>
        <MailBox :key="mailBoxKey" :enableUserDeleteEmail="true" :fetchMailData="fetchMailData"
            :deleteMail="deleteMail" :showFilterInput="true" />
    </AppSection>
</template>

<style scoped>
.user-mailbox-filter {
    margin-bottom: 10px;
}

.user-mailbox-select {
    min-width: 260px;
    max-width: 520px;
}
</style>
