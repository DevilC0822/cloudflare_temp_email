<script setup>
import { computed, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute, useRouter } from 'vue-router'

import { useGlobalState } from '../store'

import AddressMangement from './user/AddressManagement.vue';
import UserSettingsPage from './user/UserSettings.vue';
import UserBar from './user/UserBar.vue';
import BindAddress from './user/BindAddress.vue';
import UserMailBox from './user/UserMailBox.vue';

const {
    userTab, globalTabplacement, userSettings
} = useGlobalState()

const route = useRoute()
const router = useRouter()

const userTabType = computed(() => {
    const placement = globalTabplacement.value;
    return placement === 'top' || placement === 'bottom' ? 'segment' : 'bar';
});

const syncUserTabFromRoute = () => {
    const tab = route.query.tab;
    if (typeof tab !== 'string') return;
    const available = new Set(['address_management', 'user_mail_box_tab', 'user_settings', 'bind_address']);
    if (available.has(tab)) userTab.value = tab;
};

watch(() => route.query.tab, () => {
    syncUserTabFromRoute();
}, { immediate: true });

watch(userTab, async (tab) => {
    if (!tab) return;
    if (route.query.tab === tab) return;
    await router.replace({ query: { ...route.query, tab } });
});

const { t } = useI18n({
    messages: {
        en: {
            address_management: 'Addresses',
            user_mail_box_tab: 'Inbox',
            user_settings: 'Security',
            bind_address: 'Bind',
        },
        zh: {
            address_management: '地址',
            user_mail_box_tab: '收件',
            user_settings: '安全',
            bind_address: '绑定',
        }
    }
});

</script>

<template>
    <section class="app-page">
        <UserBar />
        <div v-if="userSettings.user_email" class="app-panel app-glass">
            <n-tabs :type="userTabType" v-model:value="userTab" :placement="globalTabplacement" animated>
                <n-tab-pane name="address_management" :tab="t('address_management')">
                    <AddressMangement />
                </n-tab-pane>
                <n-tab-pane name="user_mail_box_tab" :tab="t('user_mail_box_tab')">
                    <UserMailBox />
                </n-tab-pane>
                <n-tab-pane name="user_settings" :tab="t('user_settings')">
                    <UserSettingsPage />
                </n-tab-pane>
                <n-tab-pane name="bind_address" :tab="t('bind_address')">
                    <BindAddress />
                </n-tab-pane>
            </n-tabs>
        </div>
    </section>
</template>

<style scoped>
</style>
