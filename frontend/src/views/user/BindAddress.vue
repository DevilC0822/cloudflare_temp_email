<script setup>
import AppSection from '../../components/AppSection.vue'
import { useGlobalState } from '../../store'
import Login from '../common/Login.vue'

const { userSettings } = useGlobalState()
</script>

<template>
    <div class="app-center" v-if="userSettings.user_email">
        <AppSection title="绑定地址" :glass="false" class="bind-address-section">
            <Login />
        </AppSection>
    </div>
</template>

<style scoped>
.bind-address-section {
    width: min(720px, 100%);
    text-align: left;
}
</style>
