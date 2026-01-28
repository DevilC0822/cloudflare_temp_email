<script setup>
import { ref, h, onMounted } from 'vue';
import { useI18n } from 'vue-i18n'

import AppSection from '../../components/AppSection.vue'
import { api } from '../../api'
import { NButton, NPopconfirm } from 'naive-ui';

const message = useMessage()

const { t } = useI18n({
    messages: {
        en: {
            title: 'Attachments',
            refresh: 'Refresh',
            download: 'Download',
            action: 'Action',
            delete: 'Delete',
            deleteConfirm: 'Are you sure to delete this attachment?',
            deleteSuccess: 'Deleted successfully',
        },
        zh: {
            title: '附件',
            refresh: '刷新',
            download: '下载',
            action: '操作',
            delete: '删除',
            deleteConfirm: '确定要删除此附件吗？',
            deleteSuccess: '删除成功',
        }
    }
});
const data = ref([])
const showDownload = ref(false)
const curRow = ref({})
const curDownloadUrl = ref('')

const fetchData = async () => {
    try {
        const { results } = await api.fetch(
            `/api/attachment/list`
        );
        data.value = results;
    } catch (error) {
        console.log(error)
        message.error(error.message || "error");
    }
}

const columns = [
    {
        title: "key",
        key: "key"
    },
    {
        title: t('action'),
        key: 'actions',
        render(row) {
            return h('div', [
                h(NButton,
                    {
                        type: 'success',
                        tertiary: true,
                        onClick: async () => {
                            try {
                                const { url } = await api.fetch(`/api/attachment/get_url`, {
                                    method: 'POST',
                                    body: JSON.stringify({ key: row.key })
                                });
                                curDownloadUrl.value = url;
                                curRow.value = row;
                                showDownload.value = true;
                            }
                            catch (error) {
                                console.error(error);
                                message.error(error.message || "error");
                            }
                        }
                    },
                    { default: () => t('download') }
                ),
                h(NPopconfirm,
                    {
                        onPositiveClick: async () => {
                            try {
                                await api.fetch(`/api/attachment/delete`, {
                                    method: 'POST',
                                    body: JSON.stringify({ key: row.key })
                                });
                                message.success(t('deleteSuccess'));
                                await fetchData();
                            }
                            catch (error) {
                                console.error(error);
                                message.error(error.message || "error");
                            }
                        },
                    },
                    {
                        trigger: () => h(NButton,
                            {
                                tertiary: true,
                                type: "error",
                            },
                            { default: () => t('delete') }
                        ),
                        default: () => t('deleteConfirm')
                    }
                )
            ])
        }
    }
]

onMounted(async () => {
    await fetchData()
})
</script>

<template>
    <div>
        <n-modal v-model:show="showDownload" preset="dialog" :title="t('download')">
            <n-tag type="info">{{ curRow.key }}</n-tag>
            <n-button tag="a" target="_blank" tertiary type="info" size="small" :download="curRow.key.replace('/', '_')"
                :href="curDownloadUrl">
                {{ t('download') }}
            </n-button>
        </n-modal>

        <AppSection :title="t('title')" :glass="false" class="attachment-section">
            <template #actions>
                <n-button size="small" tertiary @click="fetchData">
                    {{ t('refresh') }}
                </n-button>
            </template>

            <div class="attachment-table">
                <n-data-table :columns="columns" :data="data" :bordered="false" embedded />
            </div>
        </AppSection>
    </div>
</template>

<style scoped>
.attachment-table {
    overflow: auto;
}

.attachment-table .n-data-table {
    min-width: 720px;
}
</style>
