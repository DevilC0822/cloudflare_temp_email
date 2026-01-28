<script setup>
import { onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRouter } from 'vue-router'

import { useGlobalState } from '../../store'
import { api } from '../../api'
import UserLogin from './UserLogin.vue'

const message = useMessage()
const router = useRouter()

const {
    userSettings, userJwt, userOpenSettings
} = useGlobalState()

const { t } = useI18n({
    messages: {
        en: {
            currentUser: 'Current Login User',
            fetchUserSettingsError: 'Login password is invalid or account not exist, it may be network connection issue, please try again later.',
        },
        zh: {
            currentUser: '当前登录用户',
            fetchUserSettingsError: '登录信息已过期或账号不存在，也可能是网络连接异常，请稍后再尝试。',

        }
    }
});


onMounted(async () => {
    await api.getUserOpenSettings(message);
    // make sure user_id is fetched
    if (!userSettings.value.user_id) await api.getUserSettings(message);
});
</script>

<template>
    <section class="app-page">
        <div v-if="!userSettings.fetched" class="app-panel app-glass">
            <n-skeleton style="height: 36vh" />
        </div>

        <div v-else-if="userSettings.user_email" class="app-panel app-glass userbar-success">
            <n-space align="center" justify="space-between" :wrap="true">
                <div class="userbar-title">
                    {{ t('currentUser') }}
                </div>
                <n-tag type="success" size="small">
                    {{ userSettings.user_email }}
                </n-tag>
            </n-space>
        </div>

        <div v-else class="app-center">
            <div class="login-card app-glass">
                <n-alert v-if="userJwt" type="warning" :show-icon="false" :bordered="false" closable>
                    <span>{{ t('fetchUserSettingsError') }}</span>
                </n-alert>
                <UserLogin />
            </div>
        </div>
    </section>
</template>

<style scoped>
.login-card {
    width: min(680px, 100%);
    padding: 14px;
    text-align: left;
}

.userbar-title {
    font-weight: 600;
    color: var(--app-text);
}
</style>
