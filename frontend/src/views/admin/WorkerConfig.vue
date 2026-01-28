<script setup>
import { onMounted, ref } from 'vue';
import { useI18n } from 'vue-i18n'

import AppSection from '../../components/AppSection.vue'
import { useGlobalState } from '../../store'
import { api } from '../../api'

const { loading } = useGlobalState()
const message = useMessage()

const settings = ref({})

const { t } = useI18n({
    messages: {
        en: {
            title: 'Worker Config',
            refresh: 'Refresh',
        },
        zh: {
            title: 'Worker 配置',
            refresh: '刷新',
        }
    }
});

const fetchData = async () => {
    try {
        loading.value = true
        const res = await api.fetch(`/admin/worker/configs`)
        Object.assign(settings.value, res)
    } catch (error) {
        message.error(error.message || "error");
    } finally {
        loading.value = false
    }
}

onMounted(async () => {
    await fetchData();
})
</script>

<template>
    <div class="app-center">
        <AppSection :title="t('title')" :glass="false" class="worker-config">
            <template #actions>
                <n-button tertiary @click="fetchData" :loading="loading">
                    {{ t('refresh') }}
                </n-button>
            </template>
            <pre class="worker-config__pre">{{ JSON.stringify(settings, null, 2) }}</pre>
        </AppSection>
    </div>
</template>

<style scoped>
.worker-config {
    width: min(900px, 100%);
    overflow: auto;
}

.worker-config__pre {
    margin: 0;
    font-family: var(--app-font-mono);
    font-size: 12px;
    line-height: 1.4;
}
</style>
