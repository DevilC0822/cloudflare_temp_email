<script setup>
import { ref, onMounted } from 'vue';
import { useI18n } from 'vue-i18n'

import AppSection from '../../components/AppSection.vue'
import { useGlobalState } from '../../store'
import { api } from '../../api'

const message = useMessage()
const { loading } = useGlobalState()
const dbVersionData = ref({
    need_initialization: false,
    need_migration: false,
    current_db_version: '',
    code_db_version: ''
})

const { t } = useI18n({
    messages: {
        en: {
            title: 'Database',
            need_initialization_tip: 'Database initialization is required. Please initialize the database.',
            need_migration_tip: 'Database migration is required. Please migrate the database.',
            current_db_version: 'Current DB Version',
            code_db_version: 'Code Needed DB Version',
            init: 'Initialize Database',
            migration: 'Migrate Database',
            initializationSuccess: 'Database initialized successfully',
            migrationSuccess: 'Database migrated successfully',
            refresh: 'Refresh',
        },
        zh: {
            title: '数据库',
            need_initialization_tip: '需要初始化数据库，请初始化数据库',
            need_migration_tip: '需要迁移数据库，请迁移数据库',
            current_db_version: '当前数据库版本',
            code_db_version: '需要的数据库版本',
            init: '初始化数据库',
            migration: '升级数据库 Schema',
            initializationSuccess: '数据库初始化成功',
            migrationSuccess: '数据库升级成功',
            refresh: '刷新',
        }
    }
});

const fetchData = async () => {
    try {
        loading.value = true
        const res = await api.fetch('/admin/db_version');
        if (res) Object.assign(dbVersionData.value, res);
    } catch (error) {
        message.error(error.message || "error");
    } finally {
        loading.value = false
    }
}

const initialization = async () => {
    try {
        loading.value = true
        await api.fetch('/admin/db_initialize', {
            method: 'POST'
        });
        await fetchData();
        message.success(t('initializationSuccess'));
    } catch (error) {
        message.error(error.message || "error");
    } finally {
        loading.value = false
    }
}

const migration = async () => {
    try {
        loading.value = true
        await api.fetch('/admin/db_migration', {
            method: 'POST'
        });
        await fetchData();
        message.success(t('migrationSuccess'));
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
        <AppSection :title="t('title')" :glass="false" class="db-section">
            <template #actions>
                <n-button tertiary @click="fetchData" :loading="loading">
                    {{ t('refresh') }}
                </n-button>
                <n-button v-if="dbVersionData.need_initialization" type="primary" @click="initialization"
                    :loading="loading">
                    {{ t('init') }}
                </n-button>
                <n-button v-else-if="dbVersionData.need_migration" type="primary" @click="migration" :loading="loading">
                    {{ t('migration') }}
                </n-button>
            </template>

            <n-alert v-if="dbVersionData.need_initialization" type="warning" :show-icon="false" :bordered="false"
                class="db-alert">
                <span>{{ t('need_initialization_tip') }}</span>
            </n-alert>
            <n-alert v-else-if="dbVersionData.need_migration" type="warning" :show-icon="false" :bordered="false"
                class="db-alert">
                <span>{{ t('need_migration_tip') }}</span>
            </n-alert>

            <n-alert type="info" :show-icon="false" :bordered="false">
                <span>
                    {{ t('current_db_version') }}: {{ dbVersionData.current_db_version || "unknown" }},
                    {{ t('code_db_version') }}: {{ dbVersionData.code_db_version }}
                </span>
            </n-alert>
        </AppSection>
    </div>
</template>

<style scoped>
.db-section {
    width: min(900px, 100%);
}

.db-alert {
    margin-bottom: 10px;
}
</style>
