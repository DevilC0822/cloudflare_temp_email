<script setup>
import { ref, h, onMounted, watch } from 'vue';
import { useI18n } from 'vue-i18n'
import { User, UserCheck, MailBulk } from '@vicons/fa'
import { SendOutlined } from '@vicons/material'

import AppSection from '../../components/AppSection.vue'
import { api } from '../../api'

const message = useMessage()

const { t } = useI18n({
    messages: {
        en: {
            title: 'Statistics',
            refresh: 'Refresh',
            userCount: 'User Count',
            addressCount: 'Address Count',
            activeAddressCount7days: '7 days Active Address Count',
            activeAddressCount30days: '30 days Active Address Count',
            mailCount: 'Mail Count',
            sendMailCount: 'Send Mail Count'
        },
        zh: {
            title: '统计',
            refresh: '刷新',
            userCount: '用户总数',
            addressCount: '邮箱地址总数',
            activeAddressCount7days: '7天活跃邮箱地址总数',
            activeAddressCount30days: '30天活跃邮箱地址总数',
            mailCount: '邮件总数',
            sendMailCount: '发送邮件总数'
        }
    }
});

const statistics = ref({
    addressCount: 0,
    userCount: 0,
    mailCount: 0,
    activeAddressCount7days: 0,
    activeAddressCount30days: 0,
    sendMailCount: 0,
})

const fetchStatistics = async () => {
    try {
        const {
            userCount, mailCount, sendMailCount,
            addressCount, activeAddressCount7days,
            activeAddressCount30days,
        } = await api.fetch(`/admin/statistics`);
        statistics.value.mailCount = mailCount || 0;
        statistics.value.sendMailCount = sendMailCount || 0;
        statistics.value.userCount = userCount || 0;
        statistics.value.addressCount = addressCount || 0;
        statistics.value.activeAddressCount7days = activeAddressCount7days || 0;
        statistics.value.activeAddressCount30days = activeAddressCount30days || 0;
    } catch (error) {
        console.log(error)
        message.error(error.message || "error");
    }
}

onMounted(async () => {
    await fetchStatistics()
})
</script>

<template>
    <AppSection :title="t('title')" :glass="false" class="admin-statistics">
        <template #actions>
            <n-button size="small" tertiary @click="fetchStatistics">
                {{ t('refresh') }}
            </n-button>
        </template>

        <div class="stats-grid">
            <div class="stats-item">
                <n-statistic :label="t('addressCount')" :value="statistics.addressCount">
                    <template #prefix>
                        <n-icon :component="User" />
                    </template>
                </n-statistic>
            </div>
            <div class="stats-item">
                <n-statistic :label="t('activeAddressCount7days')" :value="statistics.activeAddressCount7days">
                    <template #prefix>
                        <n-icon :component="UserCheck" />
                    </template>
                </n-statistic>
            </div>
            <div class="stats-item">
                <n-statistic :label="t('activeAddressCount30days')" :value="statistics.activeAddressCount30days">
                    <template #prefix>
                        <n-icon :component="UserCheck" />
                    </template>
                </n-statistic>
            </div>
            <div class="stats-item">
                <n-statistic :label="t('userCount')" :value="statistics.userCount">
                    <template #prefix>
                        <n-icon :component="User" />
                    </template>
                </n-statistic>
            </div>
            <div class="stats-item">
                <n-statistic :label="t('mailCount')" :value="statistics.mailCount">
                    <template #prefix>
                        <n-icon :component="MailBulk" />
                    </template>
                </n-statistic>
            </div>
            <div class="stats-item">
                <n-statistic :label="t('sendMailCount')" :value="statistics.sendMailCount">
                    <template #prefix>
                        <n-icon :component="SendOutlined" />
                    </template>
                </n-statistic>
            </div>
        </div>
    </AppSection>
</template>

<style scoped>
.stats-grid {
    display: grid;
    grid-template-columns: repeat(1, minmax(0, 1fr));
    gap: 12px;
}

@media (min-width: 720px) {
    .stats-grid {
        grid-template-columns: repeat(2, minmax(0, 1fr));
    }
}

@media (min-width: 980px) {
    .stats-grid {
        grid-template-columns: repeat(3, minmax(0, 1fr));
    }
}

.stats-item {
    padding: 12px;
    border: 1px solid var(--app-border);
    border-radius: var(--app-radius-sm);
    background: var(--app-surface-subtle);
}
</style>
