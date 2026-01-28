<script setup>
import { GithubAlt, Discord, Telegram } from '@vicons/fa'

import AppSection from '../../components/AppSection.vue'
import { useGlobalState } from '../../store'
const { announcement } = useGlobalState()
</script>

<template>
    <div class="app-center">
        <AppSection title="关于" :glass="false" class="about-section">
            <template #actions>
                <n-button tag="a" target="_blank" href="https://github.com/DevilC0822/cloudflare_temp_email" tertiary>
                    <template #icon>
                        <n-icon :component="GithubAlt" />
                    </template>
                    GitHub
                </n-button>
                <n-button tag="a" target="_blank" href="https://discord.gg/dQEwTWhA6Q" tertiary>
                    <template #icon>
                        <n-icon :component="Discord" />
                    </template>
                    Discord
                </n-button>
                <n-button tag="a" target="_blank" href="https://t.me/cloudflare_temp_email" tertiary>
                    <template #icon>
                        <n-icon :component="Telegram" />
                    </template>
                    Telegram
                </n-button>
            </template>

            <div v-if="announcement" v-html="announcement" class="about-announcement"></div>
        </AppSection>
    </div>
</template>

<style scoped>
.about-section {
    width: min(900px, 100%);
}

.about-announcement {
    margin-bottom: 12px;
}

</style>
