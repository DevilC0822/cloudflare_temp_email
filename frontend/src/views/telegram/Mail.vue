<script setup>
import { useRoute } from 'vue-router'

import { useGlobalState } from '../../store'
import { api } from '../../api'
import { onMounted, watch } from 'vue';
import { processItem } from '../../utils/email-parser'
import { utcToLocalDate } from '../../utils';

const { telegramApp, loading, useUTCDate } = useGlobalState()
const route = useRoute()

const curMail = ref({});

watch(telegramApp, async () => {
    if (telegramApp.value.initData) {
        curMail.value = await fetchMailData();
    }
});

const fetchMailData = async () => {
    try {
        const res = await api.fetch(`/telegram/get_mail`, {
            method: 'POST',
            body: JSON.stringify({
                initData: telegramApp.value.initData,
                mailId: route.query.mail_id
            })
        });
        loading.value = true;
        return await processItem(res);
    }
    catch (error) {
        console.error(error);
        return {};
    }
    finally {
        loading.value = false;
    }
};

onMounted(async () => {
    curMail.value = await fetchMailData();
});
</script>

<template>
    <div class="center">
        <div v-if="curMail.message" class="mail-wrapper app-glass">
            <n-space size="small" wrap>
                <n-tag type="info">ID: {{ curMail.id }}</n-tag>
                <n-tag type="info">{{ utcToLocalDate(curMail.created_at, useUTCDate) }}</n-tag>
                <n-tag type="info">FROM: {{ curMail.source }}</n-tag>
                <n-tag v-if="curMail.address" type="info">TO: {{ curMail.address }}</n-tag>
            </n-space>
            <iframe :srcdoc="curMail.message" class="mail-iframe"></iframe>
        </div>
    </div>
</template>


<style scoped>
.center {
    display: flex;
    text-align: left;
    place-items: center;
    justify-content: center;
    min-height: 60vh;
}

.mail-wrapper {
    width: min(900px, 100%);
    padding: 12px;
}

.mail-iframe {
    margin-top: 10px;
    width: 100%;
    height: 70vh;
    border: 1px solid var(--app-border);
    border-radius: var(--app-radius-sm);
    background: var(--app-surface-solid);
}
</style>
