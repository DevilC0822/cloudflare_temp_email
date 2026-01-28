<script setup>
import { useI18n } from 'vue-i18n'
import { useGlobalState } from '../store'
const { openSettings } = useGlobalState()


const { t } = useI18n({
    messages: {
        en: {
            copyright: "Copyright"
        },
        zh: {
            copyright: "版权所有"
        }
    }
});

</script>

<template>
    <footer class="footer">
        <div class="app-container">
            <n-divider class="footer-divider" />
            <div class="footer-content">
                <n-space justify="center" size="small" align="center" wrap>
                    <n-text depth="3">
                        {{ t('copyright') }} © 2023-{{ new Date().getFullYear() }}
                    </n-text>
                    <n-text depth="3">
                        <span v-html="openSettings.copyright"></span>
                    </n-text>
                </n-space>
            </div>
        </div>
    </footer>
</template>


<style scoped>
.footer {
    padding: 4px 0 22px;
}

.footer-divider {
    margin: 0;
    opacity: 0.6;
}

.footer-content {
    padding-top: 12px;
    text-align: center;
    color: var(--app-text-muted);
}
</style>
