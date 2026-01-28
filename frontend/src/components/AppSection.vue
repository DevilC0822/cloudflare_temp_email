<script setup>
import { computed, useSlots } from 'vue'

const props = defineProps({
    title: {
        type: String,
        default: ''
    },
    description: {
        type: String,
        default: ''
    },
    glass: {
        type: Boolean,
        default: true
    }
})

const slots = useSlots()

const showHeader = computed(() => {
    return !!props.title || !!props.description || !!slots.actions
})
</script>

<template>
    <section class="app-section" :class="{ 'app-glass': glass }">
        <header v-if="showHeader" class="app-section__header">
            <div class="app-section__titles">
                <n-text v-if="title" strong class="app-section__title">
                    {{ title }}
                </n-text>
                <n-text v-if="description" depth="3" class="app-section__desc">
                    {{ description }}
                </n-text>
            </div>
            <div v-if="$slots.actions" class="app-section__actions">
                <slot name="actions" />
            </div>
        </header>

        <div class="app-section__body">
            <slot />
        </div>
    </section>
</template>

<style scoped>
.app-section__title {
    font-size: 14px;
    line-height: 1.2;
}

.app-section__desc {
    display: block;
    margin-top: 4px;
    font-size: 12px;
    line-height: 1.3;
}
</style>
